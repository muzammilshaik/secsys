---
author: muju
title: "Kubernetes Cluster Setup"
description: "Learn how to set up and manage Kubernetes clusters for containerized applications. This guide covers installation, configuration, and best practices."
categories: ["Linux"]
tags: ["Automation", "Docker", "Linux", "DevOps"]
image:
  path: /assets/dev/kube/kubernetes.webp
  lqip: data:image/webp;base64,UklGRtQBAABXRUJQVlA4WAoAAAAQAAAAEwAAEwAAQUxQSIsAAAABgFNt2/LmycSJBGYwwCACNSQT6uBkZsaik54K6OnI48/4/hIiYgKgGZzOQiBG5owgMMuonsSGFVTZbVItfuAEndwpAcD8JRC/LUBKIGeAFq0DHGln4IP2CbdgoLdqRL1vxPDBiKciQ2NK6NB6gOlIOZsAON/0vbugjP3p+U9Avcxr8VVoN141m1ACAFZQOCAiAQAA8AYAnQEqFAAUAD6RQJpKJaOiIagIALASCWwAnTKEdXem8IZsH33hQHPt6YBvDZKh/GuLZkgF/OkWZ+bNTygMAAD6MfA3/NCO7/ftr7xz71nWzdAYviZtYg+gfdrtg/Tp4hzzy1+pO2kCX7u4nkR6Fk3ZkqICWtz+MTCn9H1ZhMdsmn1btqfkXfNRiyYkHFGKBFBmKYzn8f/9iLH91lHvYiPWd+mLRGD1UhH/jRNx/9jB9qalU9hqU8cN4tj/hMl0TGLIrjwy3tJfq1IYi7OpURv7z9/Qi5lO+41B2kD5EDUE/+zEwCoUer+P3T1kjyZ0JR/1JGnx/zHb+qJb/2Lx9B8P+9UlMJF1/nyvJDtSIRA+78jkMzmAjIZWiCxI2KAAAAA=j
published: true
hidden: false
toc: true
# https://medium.com/@venkataramarao.n/kubernetes-setup-using-ansible-script-8dd6607745f6
# https://medium.com/@sirtcp/deploying-a-three-node-kubernetes-cluster-on-debian-12-with-kubeadm-and-rook-ceph-for-persistent-eb080f31d3fc
# https://medium.com/@subhampradhan966/how-to-install-kubernetes-cluster-kubeadm-setup-on-ubuntu-22-04-step-by-step-guide-dfcf33253f5f ==> kube on ubuntu 20.04 vps
# https://github.com/CloudHub-Social/CloudHub-Cluster?tab=readme-ov-file ==> good modules are present used as source
# https://github.com/manupanand-freelance-developer/kubernetes-cluster-infra-aws/blob/main/k8s-infra-selfmanaged/ansible/roles/common/tasks/common.yml ==> ansible code exist for refference
# https://github.com/HouseOfLogicGH/KubernetesOnProxmoxWithTerraformAndAnsible
# lxc  is not recomended as it cause issues https://forum.proxmox.com/threads/help-me-install-k8s-into-lxc-container.149140/ so need wto switch the setup to vm

# https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/
# br_netfilter and overlay

# https://manjit28.medium.com/automating-vm-creation-in-proxmox-a-complete-guide-18652f346db5 ==> auto vm creation with bsh scripts
# https://pswalia2u.medium.com/deploying-kubernetes-cluster-2ef2fbdd233a
---

Kubernetes has become the go-to container orchestration platform for managing scalable, containerized applications. Whether you're running a homelab, setting up a microservices app, or learning DevOps, Kubernetes is worth mastering.

This document provides a step-by-step guide to setting up a Kubernetes cluster using kubeadm on multiple nodes. Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications

**Kubernetes Nodes**

In a Kubernetes cluster, you will encounter two distinct categories of nodes:

**Master Nodes:** These nodes play a crucial role in managing the control API calls for various components within the Kubernetes cluster. This includes overseeing pods, replication controllers, services, nodes, and more.

**Worker Nodes:** Worker nodes are responsible for providing runtime environments for containers. It’s worth noting that a group of container pods can extend across multiple worker nodes, ensuring optimal resource allocation and management.

## 🛠️ Prerequisites:

Before diving into the installation, ensure that your environment meets the following prerequisites:

- A fresh ubuntu (24.04) machine (1 master and 2 slave)
- Minimum 2 CPU cores and 2GB RAM
- Root or sudo access
- Swap disabled (swapoff -a)
- Internet access

## 📦 Step 1: Prepare the System

First, update your system and install some essential packages:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl apt-transport-https ca-certificates gnupg lsb-release curl
```

Disable swap (Kubernetes doesn’t support it):
```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

## 🌐 Step 2: Install Container Runtime (containerd)

As of Kubernetes v1.24+, Docker is no longer directly supported. Instead, we’ll use `containerd`, which is a lightweight container runtime.

Install containerd on all nodes:

```bash
sudo apt install -y containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

## 🔧 Step 3: Install kubeadm, kubelet, and kubectl

Add the [Kubernetes apt repository](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management){:target="_blank"}:

```bash
# If the folder `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
```

> In releases older than Debian 12 and Ubuntu 22.04, folder /etc/apt/keyrings does not exist by default, and it should be created before the curl command.
{: .prompt-info }

```bash
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
```

Install Kubernetes components:

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## ⚖️ Step 4: Initialize the Master Node

Now, on the master node only, initialize the cluster:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

After a successful init, follow the output instructions to set up your `kubectl` config:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
## 🤝 Step 5: Join Worker Nodes to the Cluster

Run the kubeadm join ... command shown earlier on each worker node. It will look something like this:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```
This connects the worker nodes to your master and makes them part of the cluster.

To verify, return to the master node and run:

```bash
kubectl get nodes
```

You should see all 3 nodes (1 master + 2 workers) listed.

## 🛡️ Step 6: Install a Pod Network (Calico)

Still on the master, install Calico networking so your pods can communicate across nodes:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

Verify everything is running:

```bash
kubectl get nodes
kubectl get pods -A
```

## 🚧 Optional: Allow Workloads on Master (Homelab Only)

If you're in a non-production or homelab environment and want the master to also run pods:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

> Production clusters should leave master nodes tainted to dedicate them for control plane operations only.
{: .prompt-info }

## 🧪 Testing the Setup

Let’s deploy a test nginx pod to verify that scheduling and networking are working fine:

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc
```

Visit your node IP with the NodePort to see if nginx is running.

## Conclusion

By setting up a Kubernetes cluster with one master and two worker nodes using kubeadm, you’ve laid the foundation for mastering container orchestration. This setup empowers you to test deployments, experiment with microservices, and grow your DevOps skills in a real-world environment.

Keep exploring Kubernetes features like Ingress controllers, persistent volumes, Helm charts, and observability tools. And if you're ever lost in the sea of commands, bookmark this handy [Kubernetes Cheat Sheet]({% post_url 2025-03-29-kcheat %}) to keep your workflow smooth and efficient.

