# Install Kubernetes Cluster using kubeadm

Follow this documentation to set up a Kubernetes cluster on **Ubuntu 22.04 LTS.**

## Software and OS used here:


| Software | Version |
| ------ | ----------- |
| OS | ubuntu 22.04 |
| kubeadm | 1.28.0-00 |
| kubelet | 1.28.0-00 |
| kubectl | 1.28.0-00 |

- This documentation guides you in setting up a cluster with one master node and three worker nodes.
-  If you desire fewer worker nodes, you can install software and join the worker nodes depending on that.

## Assumptions


1. You have 4 physical or virtual machines to create a Kubernetes cluster.
2. You have to assign a password to each node.
3. Each node has to assign the corresponding hostname.
4. If you are using any cloud provider virtual machine, kindly open the corresponding port number for the k8s cluster.

| Role   |    FQDN                   | IP          | OS            | RAM  | CPU | 
| ------ | ------------------------  | ------------| ------------  | -----| ----|
|Master  |k8s-master.kloudbyteshub.com  |172.16.0.100 | Ubuntu 22.04  |  2G  |   2 |
|Worker  |k8s-worker1.kloudbyteshub.com |172.16.0.101 | Ubuntu 22.04  |  1G  |   1 |
|Worker	 |k8s-worker2.kloudbyteshub.com |172.16.0.102 | Ubuntu 22.04  |  1G  |   1 |
|Worker	 |k8s-worker3.kloudbyteshub.com |172.16.0.103 | Ubuntu 22.04	 |  1G  |   1 |

## On both Kmaster and Kworker
#### Login as root user
```
sudo su -
```
Perform all the commands as root user unless otherwise specified
#### Stop and Disable firewall
```
ufw disable

```
#### Enable and Load Kernel modules
```
cat >>/etc/modules-load.d/containerd.conf<<EOF
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter

```
#### Disable and turn off SWAP
```
sed -i '/swap/d' /etc/fstab ; swapoff -a
```
#### Update sysctl settings for Kubernetes networking

```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF
sysctl --system
```
#### Install containerd runtime

```
apt update
apt install -qq -y ca-certificates curl gnupg lsb-release
```
```
mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list
```
```
apt update 
apt install -qq -y containerd.io
```
```
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
systemctl restart containerd
systemctl enable containerd >/dev/null
```
## k8s v1.29
#### Add apt repo for Kubernetes and Install Kubernetes components (kubeadm, kubelet and kubectl) v1.29

```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
sudo apt-get update
sudo apt-get install -y kubelet=1.29.0-1.1 kubeadm=1.29.0-1.1 kubectl=1.29.0-1.1
sudo apt-mark hold kubelet kubeadm kubectl
```

#### Install net-tools components (ifconfig )

```
apt install -qq -y net-tools >/dev/null 2>&1
```
#### Enable ssh password authentication

```
sed -i 's/^PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config
echo 'PermitRootLogin yes' >> /etc/ssh/sshd_config
systemctl reload sshd
```
#### Update /etc/hosts file

```
cat >>/etc/hosts<<EOF
172.16.0.100   k8s-master.kloudbytes.com     master 
172.16.0.101   k8s-worker1.kloudbytes.com    worker1 
172.16.0.102   k8s-worker2.kloudbytes.com    worker2
172.16.0.103   k8s-worker3.kloudbytes.com    worker3 
EOF
```
