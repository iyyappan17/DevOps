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
