---
title: "Building a Resilient Kubernetes Cluster: A Journey from Local to Production-Grade"
datePublished: Wed Aug 21 2024 06:07:17 GMT+0000 (Coordinated Universal Time)
cuid: cm03gbw1s000809mhge8zgcru
slug: building-a-resilient-kubernetes-cluster-a-journey-from-local-to-production-grade
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724219894523/200564e7-3ef3-4057-9fcc-6cc9ddec5e89.png
tags: high-availability, resilient-kubernetes-cluster, production-grade-kubernetes-cluster, 3-master-node-kubernetes-cluster, high-availaiblity-kubernetes-cluster, deploy-ha-kubernetes-cluster

---

Remember when you first heard about Kubernetes and went "Huh? Is that some kinda new hipster coffee shop?" Don't sweat it - we've all been there, staring at our screens like deer in headlights.

I still crack up thinking about my first dance with container orchestration. Picture this: me, running on fumes and energy drinks, trying to wrangle Kubernetes like a cat herder at a mouse convention. It was about as smooth as trying to eat soup with chopsticks while riding a mechanical bull. Yeah, that bad.

But hey, what doesn't kill us makes for a great war story, right? We're all in this mess together, and today we're gonna tame this beast and build something so awesome, your rubber duck will high-five you.

Now, if you caught [my last rant about setting up Kubernetes with Vagrant](https://devopswizard.hashnode.dev/my-wild-ride-setting-up-kubernetes-with-vagrant), you know we've already been through some stuff. We laughed, we cried, we may have thrown a keyboard or two (just kidding... maybe). But we survived, and like gluttons for punishment, we're back for more!

So grab your caffeine of choice, put on your "I break production for fun" t-shirt, and let's dive into the deep end of Kubernetes. By the time we're done, you'll be juggling containers like a pro and making the senior devs wonder if you've been secretly replaced by an AI (spoiler: you haven't, but they might).

Ready to make your computer do a happy dance? Let's rock and roll!

## What Are We Building?

We're not just setting up any old Kubernetes cluster. Oh no, we're going full-on production-grade here. Picture this:

* 3 Kubernetes master nodes (I'll explain why in a bit)
    
* 2 NGINX load balancers with a fancy thing called Keepalived
    
* A virtual IP that floats between our load balancers like a digital acrobat
    

Sounds complicated? Don't sweat it. We'll break it down step by step, and I promise you'll have a blast along the way.

### Cluster Blueprint

| **Role** | **FQDN** | **IP Address** | **OS** | **RAM** | **CPU** | **API Version** |
| --- | --- | --- | --- | --- | --- | --- |
| **Master** | master-1 | 192.168.56.11 | Ubuntu 22.04 | 4G | 2 | 1.30 |
| **Master** | master-2 | 192.168.56.12 | Ubuntu 22.04 | 4G | 2 | 1.30 |
| **Master** | master-3 | 192.168.56.13 | Ubuntu 22.04 | 4G | 2 | 1.30 |
| **Nginx LB** | nginx-lb-1 | 192.168.56.21 | Ubuntu 22.04 | 512M | 1 | \- |
| **Nginx LB** | nginx-lb-2 | 192.168.56.22 | Ubuntu 22.04 | 512M | 1 | \- |

This layout provides an overview of your cluster with essential details like role, FQDN, IP, operating system, and resource allocations.

## But First, Why This Setup?

Alright, let's address the elephant in the room. Why are we making things so complex? Well, my friend, this is where the magic of high availability comes in.

### 3 Master Nodes: The Three Musketeers

Remember the saying "two is one, and one is none"? In the world of Kubernetes, we take it a step further. Three master nodes give us a quorum. If one node decides to take an unscheduled vacation, the other two can still make decisions. It's like having a backup for your backup. Overkill for a laptop? Maybe. But hey, we're here to learn, right?

### 2 NGINX Load Balancers: The Dynamic Duo

Our load balancers are the bouncers at the club, making sure traffic gets distributed evenly. But why two? Because even bouncers need a break sometimes. If one goes down, the other steps up. It's like having an understudy in a Broadway show.

### Keepalived and Virtual IP: The Digital Acrobat

This is where things get really cool. Keepalived is like a health checker for our load balancers. It keeps an eye on them and, if one fails, it quickly moves our virtual IP to the healthy one. Imagine a tightrope walker who can instantly teleport to another rope if the current one snaps. That's our virtual IP!

Now that we know the "why," let's dive into the "how"!

## Step 1: Setting the Stage with Vagrant

First things first, we need to create our virtual playground. We'll use Vagrant for this because, let's face it, it's easier than herding cats. Here's our Vagrantfile:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.box_check_update = false

  $script = <<-SCRIPT
    sudo apt-get update
  SCRIPT

  (1..3).each do |i|
    config.vm.define "master-#{i}" do |master|
      master.vm.hostname = "master-#{i}"
      master.vm.network "private_network", ip: "192.168.56.#{10 + i}"
      master.vm.provider "virtualbox" do |vb|
        vb.memory = 4096
        vb.cpus = 2
      end
      master.vm.provision "shell", inline: $script
    end
  end

  (1..2).each do |i|
    config.vm.define "nginx-lb-#{i}" do |lb|
      lb.vm.hostname = "nginx-lb-#{i}"
      lb.vm.network "private_network", ip: "192.168.56.#{20 + i}"
      lb.vm.provider "virtualbox" do |vb|
        vb.memory = 512
        vb.cpus = 1
      end
      lb.vm.provision "shell", inline: $script
    end
  end
end
```

Save this as `Vagrantfile` and run `vagrant up`. It's like watching a digital city spring to life. Grab a coffee; this might take a few minutes.

## Step 2: NGINX Load Balancers with Keepalived - The High Availability Dance

Now, let's set up our NGINX load balancers with Keepalived. This part is like teaching two dancers to perform a perfect duet. Here's the script:

```bash
#!/bin/bash

# Set the variables for the virtual IP, network interface, and upstream server IPs
VIRTUAL_IP="192.168.56.100"
INTERFACE="enp0s8"  # Change this to the appropriate network interface
MASTER1_IP="192.168.56.11"
MASTER2_IP="192.168.56.12"
MASTER3_IP="192.168.56.13"

# Check if the script is run as root
if [ "$(id -u)" != "0" ]; then
    echo "This script must be run as root" 1>&2
    exit 1
fi

# Install necessary packages
apt-get update
apt-get install -y keepalived nginx

# Create the health check script
cat << EOF > /etc/keepalived/check_apiserver.sh
#!/bin/bash
errorExit() {
    echo "*** \$@" 1>&2
    exit 1
}
curl --silent --max-time 2 --insecure https://localhost:6443/ -o /dev/null || errorExit "Error GET https://localhost:6443/"
if ip addr | grep -q $VIRTUAL_IP; then
    curl --silent --max-time 2 --insecure https://$VIRTUAL_IP:6443/ -o /dev/null || errorExit "Error GET https://$VIRTUAL_IP:6443/"
fi
EOF
chmod +x /etc/keepalived/check_apiserver.sh

# Function to create configuration
create_config() {
    local role=$1
    if [ "$role" == "1" ]; then
        # Create the master configuration
        cat << EOF > /etc/keepalived/keepalived.conf
vrrp_script check_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -2
}

vrrp_instance VI_1 {
    interface $INTERFACE
    state MASTER
    virtual_router_id 51
    priority 101
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        $VIRTUAL_IP/24
    }

    track_script {
        check_apiserver
    }

    notify_master "/etc/keepalived/notify.sh"
    notify_backup "/etc/keepalived/notify.sh"
    notify_fault "/etc/keepalived/notify.sh"
}
EOF
    elif [ "$role" == "2" ]; then
        # Create the backup configuration
        cat << EOF > /etc/keepalived/keepalived.conf
vrrp_script check_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -2
}

vrrp_instance VI_1 {
    interface $INTERFACE
    state BACKUP
    virtual_router_id 51
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        $VIRTUAL_IP/24
    }

    track_script {
        check_apiserver
    }

    notify_master "/etc/keepalived/notify.sh"
    notify_backup "/etc/keepalived/notify.sh"
    notify_fault "/etc/keepalived/notify.sh"
}
EOF
    else
        echo "Invalid selection. Please run the script again and choose a valid option."
        exit 1
    fi
}

# Create the notification script
cat << EOF > /etc/keepalived/notify.sh
#!/bin/bash
case "\$1" in
"MASTER") systemctl restart nginx ;;
"BACKUP") systemctl stop nginx ;;
"FAULT") systemctl stop nginx ;;
esac
EOF
chmod +x /etc/keepalived/notify.sh

# Create the streams-enabled directory if it doesn't exist
mkdir -p /etc/nginx/streams-enabled

# Configure NGINX stream
cat << EOF > /etc/nginx/streams-enabled/kubernetes-masters.conf
stream {
    upstream kubernetes {
        server $MASTER1_IP:6443 max_fails=3 fail_timeout=30s;
        server $MASTER2_IP:6443 max_fails=3 fail_timeout=30s;
        server $MASTER3_IP:6443 max_fails=3 fail_timeout=30s;
    }
    server {
        listen 6443;
        proxy_pass kubernetes;
        proxy_connect_timeout 1s;
    }
}
EOF

# Update NGINX main configuration
echo "include /etc/nginx/streams-enabled/*.conf;" | tee -a /etc/nginx/nginx.conf
nginx -t
systemctl reload nginx

# Interactive menu
echo "Select the role for this server:"
echo "1) Master"
echo "2) Backup"
read -rp "Enter your choice (1 or 2): " choice

create_config "$choice"

# Start and enable services
systemctl enable keepalived
systemctl start keepalived
systemctl enable nginx
systemctl start nginx
sudo systemctl restart keepalived
sudo systemctl restart nginx
```

Save this as `hanginx_`[`keepalive.sh`](http://keepalive.sh). This script is doing a lot, so let's break it down:

1. It installs NGINX and Keepalived.
    
2. Sets up health checks for our Kubernetes API server.
    
3. Configures Keepalived to manage our virtual IP.
    
4. Sets up NGINX to load balance traffic to our Kubernetes masters.
    

Run this script on both of your NGINX VMs. Remember to choose "Master" for one and "Backup" for the other when prompted.

## Step 3: Kubernetes Node Setup - Where the Magic Happens

Now for the main event - setting up our Kubernetes nodes. This script is the heart of our operation:

```bash
#!/bin/bash

set -e

# Variables
MASTER_IPS=("192.168.56.11","192.168.56.12","192.168.56.13")  # List of master IPs
WORKER_IPS=("")  # List of worker IPs (you can add it or leave blank its for future use )
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

Save this as `k8s_`[`setup.sh`](http://setup.sh). This script is like a Swiss Army knife for Kubernetes setup. It:

1. Prepares the system for Kubernetes (disables swap, sets up networking).
    
2. Installs containerd as our container runtime.
    
3. Installs Kubernetes components (kubelet, kubeadm, kubectl).
    

Run this script on all your master nodes. It's like watching a master chef prepare a gourmet meal - methodical, precise, and when it's done, you've got something beautiful.

## Step 4: Initializing the Cluster - The Big Bang Moment

Now, the moment we've all been waiting for. On our first master node, we'll run:

```bash
kubeadm init --control-plane-endpoint="192.168.56.100:6443" --upload-certs --apiserver-advertise-address=192.168.56.11 --pod-network-cidr=192.168.0.0/16
```

This command is like the Big Bang for our Kubernetes universe. It kicks off the cluster creation and sets the stage for the other nodes to join in.

Notice how we're using our virtual IP (192.168.56.100) as the control-plane endpoint? That's our high availability magic at work!

### Save the join command for other master and worker nodes

The output of the `kubeadm init` command will include the `kubeadm join` commands for both master and worker nodes. Save these commands for later use.

### Deploy Calico network

```bash
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

## Wrapping Up: You've Built a Beast!

And there you have it, folks! We've just set up a production-grade, high-availability Kubernetes cluster right on our local machines. It's like having a miniature data center at your fingertips!

Now, I know what you're thinking. "Great, I've got a cluster. Now what?" Well, my friend, the fun is just beginning. This setup isn't just for show - it's a robust foundation that can handle real-world workloads. Those multiple master nodes? They're your safety net when things go sideways. And that virtual IP with Keepalived? It's your insurance policy against failures.

As you start playing with this setup, you'll hit some roadblocks. That's normal. Heck, I once spent an entire day debugging a cluster only to realize I had a typo in my config file. But that's the beauty of it - every problem you solve is a badge of honor in the Kubernetes world.

So go ahead, deploy some apps, break things (intentionally, of course), and learn from the process. Before you know it, you'll be the Kubernetes guru that everyone turns to for advice. And when that happens, remember us little people, will ya?

Happy clustering, and may your pods always be running and your services always responsive!