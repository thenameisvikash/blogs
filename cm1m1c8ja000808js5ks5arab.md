---
title: "DevOps Diaries: Turbocharging Your ML Infrastructure with GPU Goodness"
seoTitle: "Turbocharge ML Infrastructure with GPUs"
seoDescription: "Accelerate ML training and satisfy data scientists with an optimal DevOps GPU setup"
datePublished: Sat Sep 28 2024 10:54:58 GMT+0000 (Coordinated Universal Time)
cuid: cm1m1c8ja000808js5ks5arab
slug: devops-diaries-turbocharging-your-ml-infrastructure-with-gpu-goodness
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1727520856564/eb5aec3b-648a-47f1-927d-37c498b01893.png

---

Hello fellow DevOps warriors and ML enthusiasts! 👋 It's your friendly neighborhood infrastructure guy here, ready to spill the beans on how to set up a GPU environment that'll make your data scientists weep tears of joy. We're talking about the kind of setup that turns ML models from slowpokes to speed demons. Buckle up, because we're about to take your infrastructure from 0 to hero!

## Why This DevOps Ninja is All About That GPU Life

Look, I get it. When I first heard about GPUs for ML, I thought, "Great, another fancy toy for the data science team to break." But let me tell you, these bad boys are the real deal. They're like giving your ML operations a nitrous boost. We're talking parallel processing that makes your CPU look like it's stuck in rush hour traffic.

Here's the deal: our job in DevOps is to make sure the infrastructure can handle whatever crazy ideas the ML team comes up with next. And trust me, they always have a bigger, more complex model up their sleeve. That's where GPUs come in – they're our secret weapon in keeping the data scientists happy and the models training faster than you can say "deployment pipeline."

## The Game Plan: DevOps Style

Alright, here's how we're gonna tackle this GPU setup like true DevOps pros:

1. Update our system (because security never sleeps)
    
2. Install NVIDIA drivers (the heart of our GPU beast)
    
3. Set up CUDA toolkit (the brain of our GPU operations)
    
4. Install NVIDIA Container Toolkit (because containers are life)
    
5. Slap Docker on there (because, duh, it's 2024)
    
6. Verify our setup (trust, but verify – the DevOps mantra)
    

And because we're DevOps and we automate everything, we're going to wrap this all up in a nice, tidy bash script. It's like Infrastructure as Code, but for your GPU setup. Let's dive in!

## The Setup: Time to Flex Those DevOps Muscles

First things first, let's get our system up to date. It's like brushing your teeth before a dentist appointment – basic hygiene, folks:

```bash
sudo apt update && sudo apt upgrade -y
```

Now, let's lay down the foundation. Think of these as the load-bearing walls of our GPU mansion:

```bash
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

Next, we're adding the NVIDIA package repository. It's like adding a high-performance parts store to your neighborhood:

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa -y
sudo apt update
```

Time for the main event – installing the NVIDIA driver. We're going with version 545, which is like the latest iPhone but for your GPU:

```bash
sudo apt install -y nvidia-driver-545
```

Now for the CUDA toolkit. This is what lets us run our ML models on the GPU. It's like the secret sauce in your grandma's recipe:

```bash
CUDA_VERSION=12.3
sudo apt install -y nvidia-cuda-toolkit
```

Alright, container time! We're setting up the NVIDIA Container Toolkit because in this house, we containerize everything:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

Last but not least, Docker. Because what's a DevOps setup without Docker, right?

```bash
# Install Docker
sudo apt-get install -y docker.io

# Start Docker and enable it to run on boot
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the docker group (no more sudo for docker commands, woohoo!)
sudo usermod -aG docker $USER
```

## The Moment of Truth: Did Our DevOps Magic Work?

Alright, time to see if our GPU sorcery paid off. We're going to check our NVIDIA driver, CUDA, and Docker setup. It's like a health check for your GPU infrastructure:

```bash
if command -v nvidia-smi &> /dev/null
then
    echo "NVIDIA driver installed successfully. GPU, meet DevOps. DevOps, meet GPU."
    nvidia-smi
else
    echo "NVIDIA driver installation failed. Time to debug!"
    exit 1
fi

if command -v nvcc &> /dev/null
then
    echo "CUDA toolkit installed successfully. Let the parallel processing begin!"
    nvcc --version
else
    echo "CUDA toolkit installation failed. Back to the drawing board..."
    exit 1
fi

if command -v docker &> /dev/null
then
    echo "Docker installed successfully. Container all the things!"
    docker --version
else
    echo "Docker installation failed. A DevOps engineer's worst nightmare..."
    exit 1
fi
```

If everything went according to plan, you should see your GPU info, CUDA version, and Docker version. It's like getting all green lights on your CI/CD pipeline!

## Running Your First GPU-Powered Container: A DevOps Dream

Now that we've got our GPU infrastructure humming, let's flex those muscles. Here's how you spin up a GPU-powered container:

```bash
sudo docker run -d --name ml-powerhouse --gpus all -p 8080:8080 your_awesome_ml_image
```

This command is creating a new container that's more powerful than my first three laptops combined. It's got access to all GPUs, it's mapping ports, and it's using your custom ML image. It's DevOps and Data Science, holding hands and singing kumbaya.

## Keeping Your GPU Drivers Fresh: The DevOps Update Tango

Alright, let's talk about everyone's favorite topic: updates! Keeping your GPU drivers updated is like patching your servers – it's not glamorous, but it'll save your bacon one day. Here's the DevOps way to keep those drivers fresh:

1. First, let's see what we're working with:
    
    ```bash
    nvidia-smi
    ```
    
    This shows your current driver version. It's like checking your infrastructure's vital signs.
    
2. Now, let's see what upgrades are available:
    
    ```bash
    ubuntu-drivers devices
    ```
    
    This lists all available drivers. It's like a buffet of GPU goodness.
    
3. For the "let the machines do the work" approach:
    
    ```bash
    sudo ubuntu-drivers autoinstall
    ```
    
    This automatically installs the recommended driver. It's like auto-scaling, but for your GPU.
    
4. If you want to handpick your driver (you control freak, you):
    
    ```bash
    sudo apt install nvidia-driver-XXX
    ```
    
    Replace XXX with the version number. It's like manually adjusting your Kubernetes pods.
    
5. After installation, it's reboot time:
    
    ```bash
    sudo reboot
    ```
    
    The classic "turn it off and on again" – works for GPUs too!
    
6. Once you're back, verify the new driver:
    
    ```bash
    nvidia-smi
    ```
    
    If you see the new version, congratulations! You've just updated your GPU infrastructure. Time to update that runbook!
    

## Wrapping Up: The DevOps Mic Drop

And there you have it, folks! You've just set up a GPU environment that would make any ML engineer drool. You've containerized it, you've automated it, and you know how to keep it updated. You're not just a DevOps engineer anymore – you're a GPU-Ops ninja!

Remember:

* Keep those drivers updated – it's like continuous integration for your GPU
    
* Monitor your GPU usage with `nvidia-smi` – because what gets measured, gets managed
    
* Always use `--gpus all` with Docker – don't let those GPUs sit idle!
    
* Collaborate with your data science team – their models are only as good as the infrastructure they run on
    

Now go forth and conquer the ML world with your DevOps superpowers! May your pipelines be ever green and your GPUs ever powerful! 💻🚀🔧

## Bonus: The Ultimate GPU Setup Script

Because we're DevOps and we automate all the things, here's a bash script that does everything we just talked about:

```bash
#!/bin/bash

# Update the package list (because a clean system is a happy system)
sudo apt update && sudo apt upgrade -y

# Install required dependencies (laying the groundwork)
sudo apt install -y build-essential dkms linux-headers-$(uname -r)

# Add the NVIDIA package repository (gotta catch 'em all)
sudo add-apt-repository ppa:graphics-drivers/ppa -y
sudo apt update

# Install the NVIDIA driver (may the GPU force be with you)
sudo apt install -y nvidia-driver-545

# Install the CUDA toolkit (unleash the parallel processing kraken)
CUDA_VERSION=12.3
sudo apt install -y nvidia-cuda-toolkit

# Add the NVIDIA Container Toolkit repository (containers, assemble!)
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

# Update the package list again (double the updates, double the fun)
sudo apt-get update

# Install the NVIDIA Container Toolkit (GPU-powered containers, here we come)
sudo apt-get install -y nvidia-container-toolkit

# Install Docker (because containers are life)
sudo apt-get install -y docker.io

# Start Docker and enable it to run on boot (set it and forget it)
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the docker group (sudo no more)
sudo usermod -aG docker $USER

# Verify NVIDIA driver installation (trust, but verify)
if command -v nvidia-smi &> /dev/null
then
    echo "NVIDIA driver installed successfully. GPUs, meet your new best friend."
    nvidia-smi
else
    echo "NVIDIA driver installation failed. Time to hit the logs!"
    exit 1
fi

# Verify CUDA installation (because CUDA is cuda-awesome)
if command -v nvcc &> /dev/null
then
    echo "CUDA toolkit installed successfully. Parallel processing party time!"
    nvcc --version
else
    echo "CUDA toolkit installation failed. Back to the drawing board..."
    exit 1
fi

# Verify Docker installation (containers or bust)
if command -v docker &> /dev/null
then
    echo "Docker installed successfully. Container all the things!"
    docker --version
else
    echo "Docker installation failed. A DevOps engineer's worst nightmare..."
    exit 1
fi

# Clean up (leaving no trace, like a true DevOps ninja)
sudo apt autoremove -y

echo "All installations completed successfully. You may need to log out and back in for Docker permissions to take effect. Now go build something awesome!"
```

Save this script, make it executable with `chmod +x gpu_`[`setup.sh`](http://setup.sh), and run it with `sudo ./gpu_`[`setup.sh`](http://setup.sh). It's like an infrastructure makeover in a single script!

Now go forth, automate all the things, and may your GPUs be ever in your favor! 🚀🔧🎉