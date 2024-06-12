# Install Kubernetes Cluster using kubeadm

Follow this documentation to set up a Kubernetes cluster on **Ubuntu 22.04 LTS.**

## Software and OS used here:
___

| Software | Version |
| ------ | ----------- |
| OS | ubuntu 22.04 |
| kubeadm | 1.28.0-00 |
| kubelet | 1.28.0-00 |
| kubectl | 1.28.0-00 |

- This documentation guides you in setting up a cluster with one master node and three worker nodes.
-  If you desire fewer worker nodes, you can install software and join the worker nodes depending on that.

## Assumptions
___

1. You have 4 physical or virtual machines to create a Kubernetes cluster.
2. You have to assign a password to each node.
3. Each node has to assign the corresponding hostname.
4. If you are using any cloud provider virtual machine, kindly open the corresponding port number for the k8s cluster.
