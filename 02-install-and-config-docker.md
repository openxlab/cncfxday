# 02 Install&Config Docker

## Install Docker On Ubuntu
### kube-master
01 Upgrade Ubuntu System

```bash
ssh 'ubuntu@192.168.80.110'

echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/ubuntu
sudo chmod 0440 /etc/sudoers.d/ubuntu

sudo sed -i 's/\/dev\/mapper\/ubuntuvm--vg-swap_1/\#\/dev\/mapper\/ubuntuvm--vg-swap_1/' /etc/fstab

cat <<EOF | sudo tee /etc/hosts
127.0.0.1         kube-master-ubuntu
192.168.80.110    kube-master-ubuntu
192.168.80.111    kube-worker-ubuntu
EOF

cat <<EOF | sudo tee /etc/hostname
kube-master-ubuntu
EOF

echo 'Acquire::http::Proxy "http://proxy.com.cn:80";' | sudo tee /etc/apt/apt.conf

ps -e | grep -e apt -e adept | grep -v grep
sudo apt-get remove -y --purge lxd-client lxcfs
sudo reboot

sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo reboot
```

02 Install Docker

```bash
sudo apt-get install -y curl gnupg software-properties-common apt-transport-https ca-certificates

curl -x http://proxy.com.cn:80 -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-cache madison docker-ce
sudo apt-get install -y docker-ce
sudo usermod -a -G docker $USER
sudo systemctl enable docker && sudo systemctl start docker

```

### kube-worker
01 Upgrade Ubuntu System

```bash
ssh 'ubuntu@192.168.80.111'

echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/ubuntu
sudo chmod 0440 /etc/sudoers.d/ubuntu

sudo sed -i 's/\/dev\/mapper\/ubuntuvm--vg-swap_1/\#\/dev\/mapper\/ubuntuvm--vg-swap_1/' /etc/fstab

cat <<EOF | sudo tee /etc/hosts
127.0.0.1         kube-worker-ubuntu
192.168.80.110    kube-master-ubuntu
192.168.80.111    kube-worker-ubuntu
EOF

cat <<EOF | sudo tee /etc/hostname
kube-worker-ubuntu
EOF

echo 'Acquire::http::Proxy "http://proxy.com.cn:80";' | sudo tee /etc/apt/apt.conf

ps -e | grep -e apt -e adept | grep -v grep
sudo apt-get remove -y --purge lxd-client lxcfs
sudo reboot

sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo reboot
```

02 Install Docker

```bash
sudo apt-get install -y curl gnupg software-properties-common apt-transport-https ca-certificates

curl -x http://proxy.com.cn:80 -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-cache madison docker-ce
sudo apt-get install -y docker-ce
sudo usermod -a -G docker $USER
sudo systemctl enable docker && sudo systemctl start docker

```

## Install Docker On CentOS

## Docker Proxy Config
docker proxy

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
cat <<EOF | sudo tee /etc/systemd/system/docker.service.d/docker-proxy.conf
[Service]
Environment="HTTP_PROXY=http://proxy.com.cn:80/"
Environment="HTTPS_PROXY=http://proxy.com.cn:80/"
Environment="NO_PROXY=localhost,127.0.0.1,172.17.0.0/16,192.168.0.0/16"
EOF

sudo systemctl daemon-reload
sudo systemctl show --property=Environment docker
sudo systemctl status docker
sudo systemctl restart docker
```
