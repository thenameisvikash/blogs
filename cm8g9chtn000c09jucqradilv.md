---
title: "Set up a Highly Available Kubernetes Cluster using kubeadm on Ubuntu 22.04"
seoTitle: "HA Kubernetes Cluster on Ubuntu"
seoDescription: "Learn to set up a highly available Kubernetes cluster using kubeadm on Ubuntu 22.04, with multiple master nodes and Nginx load balancing"
datePublished: Wed Mar 19 2025 18:30:27 GMT+0000 (Coordinated Universal Time)
cuid: cm8g9chtn000c09jucqradilv
slug: set-up-a-highly-available-kubernetes-cluster-using-kubeadm-on-ubuntu-2204
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1742285575700/88806d51-84a6-4f6b-8c7a-66c484e760e1.webp

---

Want to set up a Kubernetes cluster that’s ready to grow whenever you need it to? I’ve reworked this tech guide to make it super flexible, so you can tweak the number of master or worker nodes without breaking a sweat. Let’s dive in and get your cluster up and running on Ubuntu 22.04 LTS, with multiple master nodes, worker nodes, and an Nginx load balancer to keep things smooth.

## Environment

| Role | FQDN | IP | OS | RAM | CPU | API Version |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Master | master-1 | 10.0.1.24 | Ubuntu 22.04 | 4G | 2 | 1.30 |  |
| Master | master-2 | 10.0.1.100 | Ubuntu 22.04 | 4G | 2 | 1.30 |  |
| Master | master-3 | 10.0.1.101 | Ubuntu 22.04 | 4G | 2 | 1.30 |  |
| Worker | worker-1 |  | 10.0.1.162 | Ubuntu 22.04 | 2G | 2 | 1.30 |
| Worker | worker-2 | 10.0.1.5 | Ubuntu 22.04 | 2G | 2 | 1.30 |  |
| Worker | worker-3 | 10.0.1.6 | Ubuntu 22.04 | 2G | 2 | 1.30 |  |
| Nginx LB | nginx-lb | 10.0.1.84 | Ubuntu 22.04 | 1G | 1 | \- |  |

*Perform all commands as root user unless otherwise specified*

## Set up load balancer node

### Install Nginx

```bash
apt update && apt install -y nginx
```

### Configure Nginx

Create a new configuration file `/etc/nginx/conf.d/kubernetes.conf`:

```nginx
stream {
    upstream kubernetes {
        server 10.0.1.24:6443;
        server 10.0.1.100:6443;
        # Add additional master nodes here as needed
        # server 10.0.1.101:6443;
    }

    server {
        listen 6443;
        proxy_pass kubernetes;
    }
}
```

Add the following line to `/etc/nginx/nginx.conf` in the topmost scope (outside any existing blocks):

```nginx
include /etc/nginx/conf.d/*.conf;
```

### Restart Nginx service

```bash
systemctl restart nginx
```

### Verify Nginx Load Balancer

After configuring and restarting Nginx, verify that Nginx is running and listening on port 6443:

```bash
systemctl status nginx  # Ensure Nginx is active
ss -tuln | grep 6443  # Check if Nginx is listening on port 6443
```

## On all Kubernetes nodes (kmaster1, kmaster2, kmaster3, kworker1, kworker2, kworker3)

### Disable Firewall

```bash
ufw disable
```

### Disable swap

```bash
swapoff -a
sed -i '/swap/d' /etc/fstab
```

### Verify swap is disabled

```bash
free -h  # Ensure swap is disabled
```

### Update sysctl settings for Kubernetes networking

```bash
cat <<EOF | tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sysctl --system
```

### Install containerd

```bash
{
  apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update
  apt install -y containerd.io
  mkdir -p /etc/containerd
  containerd config default | tee /etc/containerd/config.toml
  sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
  systemctl restart containerd
  systemctl enable containerd
}
```

### Verify containerd installation

```bash
systemctl status containerd  # Ensure containerd is active
```

### Kubernetes Setup

Add Apt repository:

```bash
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg
  echo "deb [signed-by=/etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list
}
```

Install Kubernetes components:

```bash
apt update
apt install -y kubeadm=1.30.0-00 kubelet=1.30.0-00 kubectl=1.30.0-00
apt-mark hold kubelet kubeadm kubectl
```

## On any one of the Kubernetes master nodes (e.g., kmaster1)

### Initialize Kubernetes Cluster

```bash
kubeadm init --control-plane-endpoint="10.0.1.84:6443" --upload-certs --apiserver-advertise-address=10.0.1.24 --pod-network-cidr=192.168.0.0/16
```

### Save the join command for other master and worker nodes

The output of the `kubeadm init` command will include the `kubeadm join` commands for both master and worker nodes. Save these commands for later use.

### Deploy Calico network

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

## Join other nodes to the cluster

### Join other master nodes

Use the respective `kubeadm join` command you copied from the output of the `kubeadm init` command on the first master. Include the `--apiserver-advertise-address` parameter for each additional master node.

### Join worker nodes

Use the `kubeadm join` command for worker nodes that you copied from the output of the `kubeadm init` command.

## Downloading kube config to your local machine

On your host machine:

```bash
mkdir -p ~/.kube
scp root@10.0.1.24:/etc/kubernetes/admin.conf ~/.kube/config
```

## Verifying the cluster

```bash
kubectl cluster-info
kubectl get nodes
kubectl get cs
```

This completes the setup of your highly available Kubernetes cluster using containerd, Kubernetes 1.30, and Nginx as the load balancer on Ubuntu 22.04. This setup can be expanded in the future by adding more master or worker nodes as needed.

## Future Expansion

To add more master nodes:

1. Add the new master's IP address to the Nginx load balancer configuration.
    
2. Run the `kubeadm join` command with the `--control-plane` flag and the `--apiserver-advertise-address` parameter on the new master node.
    

To add more worker nodes:

1. Run the `kubeadm join` command on the new worker node.
    

By following this modular approach, you can scale your Kubernetes cluster to meet future demands.