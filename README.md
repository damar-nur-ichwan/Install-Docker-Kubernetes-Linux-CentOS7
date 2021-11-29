# Install-Docker-Kubernetes-Linux-CentOS7

Note: make sure you are on root
### *Open CMD*
## 1. Install Docker - All Nodes
```
yum remove docker* -y \
&& yum update -y \
&& yum install -y yum-utils device-mapper-persistent-data lvm2 \
&& yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo \
&& yum install docker -y \
&& systemctl start docker \
&& systemctl enable docker \
&& systemctl status docker
```

## 2. Install Kubernetes - All Nodes
```
yum remove kube* -y \
&& cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl net-tools\
&& systemctl enable kubelet \
&& systemctl start kubelet \
&& firewall-cmd --permanent --add-port=6443/tcp \
&& firewall-cmd --permanent --add-port=2379-2380/tcp \
&& firewall-cmd --permanent --add-port=10250/tcp \
&& firewall-cmd --permanent --add-port=10251/tcp \
&& firewall-cmd --permanent --add-port=10252/tcp \
&& firewall-cmd --permanent --add-port=10255/tcp \
&& firewall-cmd --reload \
&& cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system \
&& setenforce 0 \
&& sed -i ‘s/^SELINUX=enforcing$/SELINUX=permissive/’ /etc/selinux/config \
&& sed -i '/swap/d' /etc/fstab \
&& swapoff -a
```
## 3. Set Hostname && View IP enp0s8 - Master Nodes
```
  hostnamectl set-hostname master \
  && ip a
```
## 4. Set Hostname && View IP enp0s8 - Worker Nodes
```
  hostnamectl set-hostname worker1 \
  && ip a
```
## 5. Add DNS Name - ALL Nodes
```
vi /etc/hosts
```

add
```
<master public ip> master
<worker public ip> worker
```
example 
```
  192.168.1.10 master
  192.168.1.20 worker
```
than press ESC ```:wq```

### *Open VM DEKSTOP*
Note: make sure you are on root
## 6. Init Kubernetes Master - Master Nodes
```
  ifdown enp0s3 \
  && kubeadm init --pod-network-cidr=10.244.0.0/16 \
  && ifup enp0s3
```
### *Open CMD*
## 7. Install Flannel & Get Kubeadm Join Token - Master Nodes
```
  sudo cp /etc/kubernetes/admin.conf $HOME/ \
  && sudo chown $(id -u):$(id -g) $HOME/admin.conf \
  && export KUBECONFIG=$HOME/admin.conf \
  && kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml \
  && kubectl get nodes \
  && kubectl get pods -A
  && kubeadm token create --print-join-command
```
## 8. Copy and Paste Kubadm Join Token from Master Nodes - Worker Nodes

```
kubeadm reset -y
```
example:
```
kubeadm join 192.168.1.26:6443 --token a1atea.qf2itw3jxdo4jkzd --discovery-token-ca-cert-hash sha256:15cd536ceb9c4c3d4ea46d1a9bcd7816e45fbc3e58a6afec176d33a2ae9a865a
```
