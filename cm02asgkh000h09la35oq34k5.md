---
title: "My Wild Ride Setting Up Kubernetes with Vagrant"
datePublished: Tue Aug 20 2024 10:44:26 GMT+0000 (Coordinated Universal Time)
cuid: cm02asgkh000h09la35oq34k5
slug: my-wild-ride-setting-up-kubernetes-with-vagrant
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724150310980/d9a600a8-f266-4667-bc4a-9a079a3528bf.png
tags: kubernetes-setup, kubernetes-kubeadm-clustersetup-containerorchestration-docker-cloudnative-devops-containerization-microservices-cloudcomputing-cloudinfrastructure, k8s-kubeadm, setup-kubernetes-cluster-using-kubeadm, kubernetes-cluster-using-vagrant

---

I finally bit the bullet and decided to set up a Kubernetes cluster using Vagrant. Let me tell you, it wasn't exactly smooth sailing. If you're thinking about trying this yourself, you might want to learn from my mistakes - trust me, I made plenty.

## Prerequisites: VirtualBox and Vagrant

Before we dive into Kubernetes, we need to set up our virtualization environment. Here's the step-by-step process I followed:

### 1\. Install VirtualBox

First, we need to install VirtualBox. Here's how I did it on Ubuntu:

```bash
sudo apt update
sudo apt install virtualbox
```

For other operating systems, you can download VirtualBox from the official website: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

### 2\. Configure VirtualBox Network

After installing VirtualBox, we need to configure its network settings to allow our custom IP range. Here's what I did:

```bash
sudo nano /etc/vbox/networks.conf
```

In the `networks.conf` file, I added these lines:

```bash
* 192.168.56.0/21
* 172.16.16.0/24
```

This configuration allows VirtualBox to use our custom IP range (172.16.16.0/24) without any issues.

## [The Vagrant Voyage Begins](https://www.virtualbox.org/wiki/Downloads)

[First things](https://www.virtualbox.org/wiki/Downloads) first, let's talk Vagrant. If you haven't used it before, trust me, it's a game-changer for creating and managing VMs. Here's how I got started:

1. Headed over to the Vagrant website and grabbed the version for my OS.
    
2. Installed it (following the instructions like a good developer).
    
3. Fired up my terminal and ran `vagrant --version` to make sure everything was kosher.
    

Easy peasy, right? Well, hold onto your hats, because things were about to get interesting.

### 3\. Install Vagrant

Now that VirtualBox is set up, let's install Vagrant:

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install vagrant
```

For macOS or Windows, you can download Vagrant from the official website: [https://www.vagrantup.com/downloads](https://www.vagrantup.com/downloads)

[After installation, I verified it](https://www.vagrantup.com/downloads) was working:

```bash
vagrant --version
Vagrant 2.4.1
```

## Crafting the Cluster

I whipped up a Vagrantfile to define tw[o VMs: one for the master node and](https://www.vagrantup.com/downloads) one for the worker. Here's what it looked like:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_check_update = false
  
  # Common provisioning script for updating packages
  $script = <<-SCRIPT
    sudo apt-get update
  SCRIPT
  
  # Master Node
  config.vm.define "master-1" do |master|
    master.vm.hostname = "master-1"
    master.vm.network "private_network", ip: "172.16.16.11"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = 4096
      vb.cpus = 3
    end
    master.vm.provision "shell", inline: $script
  end
  
  # Worker Node
  config.vm.define "worker-1" do |worker|
    worker.vm.hostname = "worker-1"
    worker.vm.network "private_network", ip: "172.16.16.21"
    worker.vm.provider "virtualbox" do |vb|
      vb.memory = 4096
      vb.cpus = 4
    end
    worker.vm.provision "shell", inline: $script
  end
end
```

I saved this as `Vagrantfile` in a new directory called test, then ran this inside the test directory:

```bash
vagrant up
```

I fired up the VMs with a quick `vagrant up`, and boom! We were in business. Or so I thought...

## The IP Address Nightmare

Here's where things got weird. I SSH'd into the nodes and ran `hostname -I`, expecting to see one neat little IP address. Instead, I got this:

```bash
10.0.2.15 172.16.16.11
```

What the...? Both nodes had two IP addresses! Turns out, Vagrant was playing tricks on me. It sets up two network interfaces: one for its internal networking (10.0.2.15) and another for the private network we defined (172.16.16.11 for master, 172.16.16.21 for worker).

At first, I thought, "No biggie, right?" Wrong. When I tried to deploy pods and check their logs with `kubectl logs`, all I got were errors saying the resource couldn't be found. The culprit? Both nodes were using the same internal IP (10.0.2.15), which was messing with Kubernetes networking. Talk about a facepalm moment!

## The Fix: A Script to Rule Them All

After banging my head against the wall for a while, I had an epiphany. What if I could create a script smart enough to handle this IP madness, and flexible enough to work in other environments too? Challenge accepted!

Here's the beast I came up with (brace yourselves, it's a long one):

```bash
#!/bin/bash

set -e

# Variables
MASTER_IPS=("172.16.16.11")  # List of master IPs
WORKER_IPS=("172.16.16.21")  # List of worker IPs
K8S_VERSION="1.30.3-1.1"

# Function to prompt for environment type
get_environment_type() {
    echo "Select the environment type:"
    echo "1) Virtual Machine (e.g., Vagrant)"
    echo "2) Cloud or Bare Metal"
    read -p "Enter your choice (1 or 2): " ENV_TYPE
    case $ENV_TYPE in
        1) ENVIRONMENT="vm" ;;
        2) ENVIRONMENT="non-vm" ;;
        *) echo "Invalid choice. Exiting."; exit 1 ;;
    esac
}

# Function to get the correct IP address
get_ip_address() {
    if [ "$ENVIRONMENT" == "vm" ]; then
        # For VM, prompt for the network prefix
        read -p "Enter the network prefix for your VMs (e.g., 172.16.16): " NETWORK_PREFIX
        LOCAL_IP=$(hostname -I | tr ' ' '\n' | grep "^$NETWORK_PREFIX" | head -n 1)
        if [[ -z "$LOCAL_IP" ]]; then
            echo "Error: No IP address found with the given prefix."
            exit 1
        fi
    else
        # For non-VM, use the primary IP
        LOCAL_IP=$(hostname -I | awk '{print $1}')
    fi
    echo $LOCAL_IP
}

# Function to update /etc/hosts
update_hosts() {
    echo "Updating /etc/hosts..."
    {
        for i in "${!MASTER_IPS[@]}"; do
            echo "${MASTER_IPS[$i]} master-$((i+1))"
        done
        for i in "${!WORKER_IPS[@]}"; do
            echo "${WORKER_IPS[$i]} worker-$((i+1))"
        done
    } | sudo tee -a /etc/hosts
}

# Function to set the hostname
set_hostname() {
    LOCAL_IP=$(get_ip_address)

    # Determine the hostname based on the IP
    if [[ " ${MASTER_IPS[@]} " =~ " ${LOCAL_IP} " ]]; then
        for i in "${!MASTER_IPS[@]}"; do
            if [ "${MASTER_IPS[$i]}" == "$LOCAL_IP" ]; then
                HOSTNAME="master-$((i+1))"
                break
            fi
        done
    elif [[ " ${WORKER_IPS[@]} " =~ " ${LOCAL_IP} " ]]; then
        for i in "${!WORKER_IPS[@]}"; do
            if [ "${WORKER_IPS[$i]}" == "$LOCAL_IP" ]; then
                HOSTNAME="worker-$((i+1))"
                break
            fi
        done
    else
        HOSTNAME="node-$(echo $LOCAL_IP | sed 's/[^0-9]//g')"
    fi

    echo "Setting hostname to $HOSTNAME..."
    sudo hostnamectl set-hostname $HOSTNAME
}

# Get the environment type
get_environment_type

# Set hostname
set_hostname

# Update /etc/hosts
update_hosts

# Common setup for all Kubernetes nodes
echo "Setting up node..."

# Disable Firewall
sudo ufw disable

# Disable swap
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

# Verify swap is disabled
free -h

# Update sysctl settings for Kubernetes networking
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system

# Install containerd
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

# Wait for containerd to fully start
echo "Waiting for containerd to start..."
while ! systemctl is-active --quiet containerd; do
    sleep 1
done

# Install Kubernetes components
echo "Adding Kubernetes repository and installing components..."

# Add Kubernetes signing key
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add Kubernetes repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list

# Install Kubernetes components
sudo apt update
sudo apt install -y kubelet=$K8S_VERSION kubeadm=$K8S_VERSION kubectl=$K8S_VERSION
sudo apt-mark hold kubelet kubeadm kubectl

echo "Setup completed!"
```

I know, I know, it's a lot to take in. But trust me, this script is smarter than it looks. It can figure out whether it's running on a VM or a real server, and it knows just what to do in each case. The real magic happens in the `get_ip_address` function:

```bash
get_ip_address() {
    if [ "$ENVIRONMENT" == "vm" ]; then
        read -p "Enter the network prefix for your VMs (e.g., 172.16.16): " NETWORK_PREFIX
        LOCAL_IP=$(hostname -I | tr ' ' '\n' | grep "^$NETWORK_PREFIX" | head -n 1)
        if [[ -z "$LOCAL_IP" ]]; then
            echo "Error: No IP address found with the given prefix."
            exit 1
        fi
    else
        LOCAL_IP=$(hostname -I | awk '{print $1}')
    fi
    echo $LOCAL_IP
}
```

This little gem makes sure we're using the right IP address for Kubernetes networking, avoiding all that confusion with Vagrant's extra interface.

## The Home Stretch: Cluster Init and Networking

After running our super-script on both nodes, we were almost there. Just two more steps to go:

1. Initialize the Kubernetes cluster
    
2. Set up the Container Network Interface (CNI)
    

Here's how I did it:

### Initializing the Cluster

On the master node, I ran:

```bash
sudo kubeadm init --apiserver-advertise-address=172.16.16.11 --pod-network-cidr=192.168.0.0/16
```

This command is like telling Kubernetes, "Hey, use this IP address to talk to everyone, and here's the range of IPs you can use for pods." Make sure to save the `kubeadm join` command it spits out - you'll need it to add your worker node to the party.

### Setting Up the CNI

For our cluster to actually work, we need a CNI plugin. I went with Calico because, well, why not? Here's how I set it up:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.1/manifests/calico.yaml
```

This command basically tells Kubernetes, "Here's how you should handle networking for all those pods you're going to run."

After running this, I waited a few minutes for all the Calico pods to start up. You can watch the progress by running:

```bash
kubectl get pods -n kube-system -w
```

Once everything was showing as 'Running', I knew we were in business!

## The Sweet Taste of Success

After all that blood, sweat, and tears (okay, maybe just sweat and tears), I finally had a working Kubernetes cluster. Running `kubectl get nodes -o wide` now showed:

```bash
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
master-1   Ready    control-plane   43m   v1.30.3   172.16.16.11   <none>        Ubuntu 22.04.4 LTS   5.15.0-117-generic   containerd://1.7.20
worker-1   Ready    <none>          32m   v1.30.3   172.16.16.21   <none>        Ubuntu 22.04.4 LTS   5.15.0-117-generic   containerd://1.7.20
```

I could finally deploy pods and check their logs without pulling my hair out!

## Lessons Learned

This whole adventure taught me a few things:

1. When using Vagrant or any VM setup, always be ready for network shenanigans.
    
2. Double-check (and triple-check) your node IP configurations before diving into Kubernetes setup.
    
3. A flexible setup script can save you from a world of pain across different environments.
    

Have you guys run into similar nightmares while setting up Kubernetes clusters? I'd love to hear your war stories in the comments below!

## What's Next?

Hold onto your hats, because next time, we're taking things up a notch. I'm planning to set up a high-availability Kubernetes cluster with 3 master nodes and two NGINX load balancers using a virtual IP. We'll use Vagrant to simulate a production-ready environment, and I'll break down every step of the process.

Stay tuned - it's gonna be epic!

Happy clustering, everyone!