# Install Kubernetes Cluster using kubeadm

Follow this documentation to set up a Kubernetes cluster on **Ubuntu 22.04 LTS.**

## Software and OS used here:


| Software | Version |
| ------ | ----------- |
| OS | ubuntu 22.04 |
| kubeadm | 1.29.0-00 |
| kubelet | 1.29.0-00 |
| kubectl | 1.29.0-00 |

- This documentation guides you in setting up a cluster with one master node and three worker nodes.
-  If you desire fewer worker nodes, you can install software and join the worker nodes depending on that.

## Assumptions


1. You have 4 physical or virtual machines to create a Kubernetes cluster.
2. You have to assign a password to each node.
3. Each node has to assign the corresponding hostname.
4. If you are using any cloud provider virtual machine, kindly open the corresponding port number for the k8s cluster.

| Role   |    FQDN                   | IP          | OS                | RAM  | CPU | 
| ------ | ------------------------  | ------------| ----------------  | -----| ----|
|Master  |k8s-master.kloudbyteshub.com  |192.168.0.121 | Ubuntu 22.04  |  2G  |   2 |
|Worker  |k8s-worker1.kloudbyteshub.com |192.168.0.122 | Ubuntu 22.04  |  1G  |   1 |
|Worker	 |k8s-worker2.kloudbyteshub.com |192.168.0.123 | Ubuntu 22.04  |  1G  |   1 |
|Worker	 |k8s-worker3.kloudbyteshub.com |192.168.0.124 | Ubuntu 22.04	 |  1G  |   1 |

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
apt-get update
apt-get install -y apt-transport-https ca-certificates curl gpg
```

```
mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```
apt-get update
apt-get install -y kubelet=1.29.1-1.1 kubeadm=1.29.1-1.1 kubectl=1.29.1-1.1
apt-mark hold kubelet kubeadm kubectl
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
192.168.0.121   k8s-master.kloudbyteshub.com     master 
192.168.0.122   k8s-worker1.kloudbyteshub.com    worker1 
192.168.0.123   k8s-worker2.kloudbyteshub.com    worker2
192.168.0.124   k8s-worker3.kloudbyteshub.com    worker3 
EOF
```
## Kubernetes Setup On k8s-master

#### Pull required containers

```
kubeadm config images pull
```
#### Initialize Kubernetes Cluster
Note: apiserver-advertise-address is your k8s-master ip address

```
kubeadm init --apiserver-advertise-address=192.168.0.121 --pod-network-cidr=172.16.0.0/16 
```
#### Deploy Calico network -- If k8s v1.29
#### add Calico 3.27.3 CNI

```
kubectl apply -f  https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml
```
#### Cluster join command

```
kubeadm token create --print-join-command > /print-join-cluster.txt
```
Note: use the "print-join-cluster.txt file for future use to join the worker node into k8s-master.

#### To be able to run kubectl commands as root user
If you want to be able to run kubectl commands as root user, then as a root user perform these
```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

#### To be able to run kubectl commands as non-root user
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
#### Kubectl auto-complete and k alias

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
alias k=kubectl
complete -o default -F __start_kubectl k
```
## On k8s-worker<*>
#### Join the worker node into k8s-worker<*>
Use the output from kubeadm token create command in the previous step from the master server and run here.
```
kubeadm join 192.168.0.121:6443 --token t8j95u.93frfpwydl3z3mbb   --discovery-token-ca-cert-hash sha256:67fac942809be283243bc4ef969f4b6a6135cb737299b00c3409d54b48b2b0f8
```
## Verifying the cluster (On k8s-master)
#### Get Kubernetes Cluster Nodes status
```
kubectl get nodes
```
or
```
kubectl get no
kubectl get node
kubectl get nodes
```
The above commands will give the same output.

```
kubectl get node -o wide
```
Note: with node wide option

#### Get component status
```
kubectl get cs
```
