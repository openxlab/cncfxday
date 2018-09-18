# 98 Prepare Ubuntu KVM Host

## Getting Super Powers

**配置网络采用bridge方式**

```
cat << EOF | sudo tee /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens2f0:
      dhcp4: false
  bridges:
    br-ex:
      interfaces: [ens2f0]
      addresses: [ 192.168.100.100/24 ]
      gateway4: 192.168.100.1
      nameservers:
          addresses:
              - "192.168.100.1"
      parameters:
        forward-delay: 0
EOF
sudo netplan apply
```

**sudo免密配置**

```
echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/ubuntu
sudo chmod 0440 /etc/sudoers.d/ubuntu
```

**配置apt proxy**

```text
echo 'Acquire::http::Proxy "http://proxy.com.cn:80";' | sudo tee /etc/apt/apt.conf
```

**更新系统**

```text
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo reboot
```

**配置嵌套虚拟化**

```text
sudo cat /proc/cpuinfo | egrep "vmx|svm"
cat /sys/module/kvm_intel/parameters/nested
echo 'options kvm_intel nested=1' | sudo tee -a /etc/modprobe.d/qemu-system-x86.conf
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
```

**安装KVM及相关管理工具**

```text
sudo apt-get install -y qemu-kvm libvirt-bin libvirt-clients libvirt-daemon-system ubuntu-vm-builder virtinst bridge-utils virt-manager libguestfs-tools
sudo adduser $USER libvirt
sudo adduser $USER libvirt-qemu
sudo adduser $USER libvirtd
sudo kvm-ok
```

**配置libvirt相关网络**

```text
brctl show
virsh net-list
virsh net-destroy default
virsh net-undefine default

virsh net-define /dev/stdin <<EOF
<network>
  <name>br-ex</name>
  <bridge name="br-ex" />
  <forward mode="bridge"/>
</network>
EOF
virsh net-autostart br-ex
virsh net-start br-ex

virsh net-define /dev/stdin <<EOF
<network>
   <name>br-nat</name>
   <bridge name="br-nat" />
   <forward mode="nat"/>
   <ip address='192.168.80.1' netmask='255.255.255.0'>
      <dhcp> <range start='192.168.80.100' end='192.168.80.200'/>
      </dhcp>
   </ip>
</network>
EOF
virsh net-autostart br-nat
virsh net-start br-nat

virsh net-define /dev/stdin <<EOF
<network>
   <name>br-pxe</name>
   <bridge name="br-pxe" />
   <ip address='192.168.24.254' netmask='255.255.255.0'>
   </ip>
</network>
EOF
virsh net-autostart br-pxe
virsh net-start br-pxe
```

**安装CentOS虚拟机**

```text
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/centos.qcow2 100G
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/centos.qcow2
sudo virt-install --connect qemu:///system --name centos \
  --ram 16384 --vcpus=8,maxvcpus=8,sockets=2,cores=4 \
  --network network=br-nat,model=virtio \
  --network network=br-ex,model=virtio \
  --disk path=/var/lib/libvirt/images/centos.qcow2,format=qcow2,device=disk,bus=virtio \
  --cdrom CentOS-7-x86_64-Minimal-1804.iso \
  --graphics=vnc,listen=0.0.0.0 --hvm --cpu=IvyBridge,+vmx \
  --os-type linux --os-variant=rhel7
sudo mv /var/lib/libvirt/images/centos.qcow2 centos.qcow2.ip252
virsh destroy centos
virsh undefine centos

VM_NAME=centos-harbor
sudo cp centos.qcow2.ip252 /var/lib/libvirt/images/$VM_NAME.qcow2
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/$VM_NAME.qcow2
sudo virt-install --connect qemu:///system --import --name $VM_NAME \
  --ram 16384 --vcpus=8,maxvcpus=8,sockets=2,cores=4 \
  --network=bridge:br-nat,model=virtio --network=bridge:br-ex,model=virtio \
  --disk path=/var/lib/libvirt/images/$VM_NAME.qcow2,format=qcow2 \
  --os-type linux --os-variant=rhel7
```

**安装Ubuntu虚拟机**

```text
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/ubuntu.qcow2 100G
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/ubuntu.qcow2
sudo virt-install --connect qemu:///system --name ubuntu \
  --ram 16384 --vcpus=8,maxvcpus=8,sockets=2,cores=4 \
  --network network=br-nat,model=virtio \
  --network network=br-ex,model=virtio \
  --disk path=/var/lib/libvirt/images/ubuntu.qcow2,format=qcow2,device=disk,bus=virtio \
  --cdrom ubuntu-18.04.1-server-amd64.iso \
  --graphics=vnc,listen=0.0.0.0 --hvm --cpu=IvyBridge,+vmx \
  --os-type linux
sudo mv /var/lib/libvirt/images/ubuntu.qcow2 ubuntu.qcow2.ip253
virsh destroy ubuntu
virsh undefine ubuntu

VM_NAME=ubuntu-hal
sudo cp ubuntu.qcow2.ip253 /var/lib/libvirt/images/$VM_NAME.qcow2
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/$VM_NAME.qcow2
sudo virt-install --connect qemu:///system --import --name $VM_NAME \
  --ram 16384 --vcpus=8,maxvcpus=8,sockets=2,cores=4 \
  --network=bridge:br-nat,model=virtio --network=bridge:br-ex,model=virtio \
  --disk path=/var/lib/libvirt/images/$VM_NAME.qcow2,format=qcow2 \
  --os-type linux --os-variant=ubuntu16.04
```

**安装Windows虚拟机**

```text
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/win2019.qcow2 80G
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/win2019.qcow2
sudo virt-install --connect qemu:///system --name win2019 \
  --ram 16384 --vcpus=8,maxvcpus=8,sockets=2,cores=4 \
  --network network=br-ex,model=virtio \
  --network network=br-nat,model=virtio \
  --disk Windows_InsiderPreview_Server_vNext_zh-cn_17744.iso,device=cdrom \
  --disk virtio-win-0.1.141.iso,device=cdrom \
  --disk path=/var/lib/libvirt/images/win2019.qcow2,format=qcow2,device=disk,bus=virtio \
  --graphics vnc,listen=0.0.0.0,port=5910 --hvm --cpu=IvyBridge,+vmx \
  --virt-type kvm --os-type windows

virsh vncdisplay win2019 
```

**系统其他设置**

```text
ssh-keygen -b 2048 -t rsa -f ~/.ssh/id_rsa -q -N ""
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.80.111
ssh 'root@192.168.80.111'

sudo apt-get install -y firefox firefox-locale-zh-hans
sudo mv 'WenQuanYi Micro Hei.ttf' /usr/share/fonts
sudo fc-cache -f -v

export http_proxy="http://proxy.com.cn:80"
export https_proxy="https://proxy.com.cn:80"
export no_proxy="localhost,127.0.0.1,192.168.0.0/16"
alias curl='curl -x http://proxy.com.cn:80'
```

