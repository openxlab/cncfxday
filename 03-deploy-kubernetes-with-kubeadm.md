# 03 Deploy Kubernetes With Kubeadm

## Kubeadm On Ubuntu

### master

01 Install kubelet kubeadm kubectl

```bash
curl -x http://proxy.com.cn:80 -s https://packages.cloud.google.com/apt/doc/apt-key.gpg |sudo apt-key add -
sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```

02 Initializing the master cluster with kubeadm

```bash
free -m
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf

export http_proxy="http://proxy.com.cn:80"
export https_proxy="http://proxy.com.cn:80"
export no_proxy="localhost,127.0.0.1,192.168.0.0/16"

kubeadm config images pull
ubuntu@kube-master-ubuntu:~$ docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy-amd64                v1.11.3             be5a6e1ecfa6        9 days ago          97.8MB
k8s.gcr.io/kube-apiserver-amd64            v1.11.3             3de571b6587b        9 days ago          187MB
k8s.gcr.io/kube-controller-manager-amd64   v1.11.3             a710d6a92519        9 days ago          155MB
k8s.gcr.io/kube-scheduler-amd64            v1.11.3             ca1f38854f74        9 days ago          56.8MB
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        3 months ago        45.6MB
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        5 months ago        219MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        9 months ago        742kB

sudo kubeadm init --kubernetes-version=v1.11.3 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.100.110

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl cluster-info
kubectl get nodes -o wide
kubectl get pods --all-namespaces
kubectl get services --all-namespaces
kubectl taint nodes --all node-role.kubernetes.io/master-

curl -x http://proxy.com.cn:80 -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml

curl -x http://proxy.com.cn:80 -O https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
echo "  type: NodePort" >> kubernetes-dashboard.yaml
sed -i 's/- port: 443/- port: 443\n      nodePort: 30443/' kubernetes-dashboard.yaml
kubectl apply -f kubernetes-dashboard.yaml
kubectl -n kube-system get service kubernetes-dashboard
https://192.168.100.111:30443

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EOF
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep tiller | awk '{print $1}')

```

### worker

01 Install kubelet kubeadm

```bash
curl -x http://proxy.com.cn:80 -s https://packages.cloud.google.com/apt/doc/apt-key.gpg |sudo apt-key add -
sudo add-apt-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

sudo apt-get update
sudo apt-get install -y kubelet kubeadm
sudo apt-mark hold kubelet kubeadm

```

02 Join a worker with kubeadm

```bash
free -m
sudo rm /etc/resolv.conf
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf

sudo kubeadm join 192.168.100.110:6443 --token 19gvc6.gq2wbmt6a7wep7bg --discovery-token-ca-cert-hash sha256:6d3478271720d131411d1c04d5d72ffd4e754c05b5e23928783c88fee3f83365

```

## Kubeadm On CentOS

### master

some

```bash

```

### worker

some

```bash

```
