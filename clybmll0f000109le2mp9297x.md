---
title: "Step-by-Step Kubernetes Rolling Updates and Rollbacks Tutorial"
seoTitle: "Kubernetes Rolling Updates & Rollbacks Guide"
seoDescription: "A comprehensive guide on Kubernetes Rolling Updates and Rollbacks with examples and strategies for aspiring CKA professionals"
datePublished: Sun Jul 07 2024 14:05:31 GMT+0000 (Coordinated Universal Time)
cuid: clybmll0f000109le2mp9297x
slug: step-by-step-kubernetes-rolling-updates-and-rollbacks-tutorial
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1720362052121/ed01102b-7301-4a68-a453-bad2af907445.png
tags: kubernetes-rolling-updates, kubernetes-rollbacks, kubernetes-deployment-strategies, rolling-update-kubernetes, kubernetes-recreate-strategy, kubernetes-update-pods, kubernetes-deployment-tutorial, kubernetes-imperative-commands, kubernetes-deployment-yaml, kubernetes-update-best-practices, kubernetes-deployment-rollback, kubernetes-zero-downtime-deployment, kubernetes-rolling-update-vs-recreate, kubernetes-cka-preparation, kubernetes-deployment-example

---

Hey there, fellow Kubernetes adventurers! 👋 Ready to level up your CKA (Certified Kubernetes Administrator) skills? Today, we're diving deep into the world of Rolling Updates and Recreate deployments in Kubernetes. Trust me, this stuff is cooler than a penguin's tuxedo—it's like giving your apps superpowers!

## Rolling Updates: The Smooth Operator of Kubernetes Deployment Strategies

Picture this: you're running a bustling café, and you've decided to replace all your tables with fancy new ones. But here's the catch—you can't just shut down for a day. Your regulars would riot! (I can almost hear the hangry mob now 😅)

So what do you do? You swap out a few tables at a time, keeping the café running smoothly. That, my friends, is essentially what a Rolling Update does in Kubernetes.

A Rolling Update is like that considerate café owner, gradually replacing old instances of your application with shiny new ones, ensuring zero downtime and continuous availability. No angry customers, just smooth sailing. Isn't that a relief?

### The Magic Behind the Curtain: How Kubernetes Performs Rolling Updates

Now, let's peek behind the scenes. When you trigger a Rolling Update, Kubernetes doesn't just wave a magic wand. Oh no, it's more like a well-choreographed dance routine:

1. **New ReplicaSet Creation**: Kubernetes creates a new ReplicaSet with the updated pod template.
    
2. **Gradual Scale-up**: It starts adding new pods to this ReplicaSet, carefully following the deployment's `maxSurge` parameter.
    
3. **Slow Fade-out**: Simultaneously, it begins removing pods from the old ReplicaSet, adhering to the `maxUnavailable` parameter.
    
4. **Health Checks Galore**: Throughout this process, Kubernetes is like an overprotective parent, constantly performing readiness and liveness probes to ensure the new pods are healthy before moving forward.
    
5. **Grand Finale**: The curtain falls when all new pods are up and running, and all old pods have taken their final bow.
    

Let's see this in action with some commands.

### Step-by-Step Example of Rolling Update

1. **Create a Deployment YAML File**
    
    First, create a deployment file `deployment.yaml` with the rolling update strategy specified:
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: cafe-app
    spec:
      replicas: 3
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxUnavailable: 1 # Specifies the maximum number of pods that can be unavailable during the update process
          maxSurge: 1       # Specifies the maximum number of extra pods that can be created during the update process
      selector:
        matchLabels:
          app: cafe-app
      template:
        metadata:
          labels:
            app: cafe-app
        spec:
          containers:
          - name: nginx
            image: nginx:1.14
    ```
    
    2. **Apply the Deployment**
        
    
    Apply the deployment using `kubectl apply`:
    
    ```bash
    kubectl apply -f deployment.yaml
    ```
    
2. **Check the Initial State**
    

Verify the initial deployment state:

```bash
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

4. **Update the Image to Trigger a Rolling Update**
    

Update the deployment to a new image version:

```bash
kubectl set image deployment/cafe-app nginx=nginx:1.15
```

5. **Watch the Rolling Update in Real-Time**
    

Monitor the update process:

```bash
kubectl rollout status deployment/cafe-app
```

### Imperative example Commands for Rolling Updates

Let's see this ballet in action with real commands:

```bash
# Create our initial deployment
kubectl create deployment cafe-app --image=nginx:1.14.2

# Check our initial setup
kubectl get deployments
kubectl get replicasets
kubectl get pods

# Time for an upgrade! Let's switch to a newer version
kubectl set image deployment/cafe-app nginx=nginx:1.16.1

# Watch the magic happen in real-time
kubectl rollout status deployment/cafe-app

# Take a bow and admire your handiwork
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

Run these commands, and you'll see two ReplicaSets doing their dance—one scaling down, one scaling up. It's like watching a perfectly synchronized tango!

### Use Cases for Rolling Updates

1. **Stateless Applications**: Perfect for web servers or API gateways that don't store session data.
    
2. **Continuous Delivery Pipelines**: Ideal for frequent, small updates to your application.
    
3. **Microservices Architecture**: Great for updating individual services without disrupting the entire system.
    
4. **Blue-Green Deployments**: Can be used as part of a blue-green deployment strategy for even more control.
    

Note:- <mark>By default, Kubernetes follows the RollingUpdate strategy for deployments. This strategy allows for updating applications with zero downtime by gradually replacing old pods with new ones.</mark>

## Recreate: The "Out with the Old, In with the New" Approach

Sometimes, you need a more... dramatic approach. Enter the Recreate strategy, the diva of Kubernetes deployment strategies.

The Recreate strategy is like closing down your café, throwing out all the old tables, and reopening with all new ones. Here's how it goes down:

1. "We're closed!" - All existing pods are terminated.
    
2. "Grand reopening!" - New pods are created with the latest version.
    

It's simple, it's bold, and, yes, it comes with downtime. But sometimes, that's exactly what you need.

### Example YAML for Recreate Strategy

Want to use Recreate? Here's how you'd write it in your deployment YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cafe-app
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: cafe
  template:
    metadata:
      labels:
        app: cafe
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
```

### Use Cases for Recreate Strategy

1. **Database Schema Changes**: When you need to update your database schema and ensure all instances are using the new schema simultaneously.
    
2. **Incompatible Versions**: When your new version is not backward-compatible with the old one.
    
3. **Resource Constraints**: In environments with limited resources where running multiple versions simultaneously isn't feasible.
    
4. **Stateful Applications**: For applications that maintain local state and can't easily coexist with different versions.
    

## Rollbacks: Your "Oops" Button in Kubernetes

Let's face it, we've all been there. You deploy a new version, and suddenly your perfectly running app turns into a dumpster fire. 🔥 Panic sets in. But wait! Before you start updating your resume, remember—Kubernetes has your back with rollbacks.

Think of a rollback as your "Undo" button. It's like being able to time-travel back to when everything was working fine. How cool is that?

### Example Commands for Rollbacks

Here's how to hit that "Undo" button:

```bash
# Go back to the last known good version
kubectl rollout undo deployment/cafe-app

# Or if you're feeling adventurous, pick a specific version
kubectl rollout undo deployment/cafe-app --to-revision=2

# Check the status of your rollback
kubectl rollout status deployment/cafe-app

# View the rollout history
kubectl rollout history deployment/cafe-app
```

### Use Cases for Rollbacks

1. **Critical Bugs**: When a new deployment introduces unexpected, severe issues in production.
    
2. **Performance Degradation**: If the new version causes significant performance problems.
    
3. **Compliance Issues**: When a deployment accidentally violates regulatory requirements.
    
4. **Integration Failures**: If the new version breaks integration with other services in your ecosystem.
    

## Rolling Update vs Recreate: The Kubernetes Deployment Strategy Showdown

Alright, it's time for the main event! In the red corner, we have Rolling Update. In the blue corner, Recreate. Let's break down this epic battle:

1. **Availability**:
    
    * Rolling Update: Keeps your app up and running, like a 24/7 convenience store.
        
    * Recreate: Closes shop for a bit, like that fancy restaurant that's only open for dinner.
        
2. **Speed**:
    
    * Rolling Update: Slow and steady, like a careful painter adding layers to a masterpiece.
        
    * Recreate: Fast and furious, like ripping off a band-aid.
        
3. **Resource Usage**:
    
    * Rolling Update: A bit of a resource hog, running both old and new versions for a while.
        
    * Recreate: Efficient with resources, like a minimalist's dream apartment.
        
4. **Use Cases**:
    
    * Rolling Update: Perfect for stateless apps or when downtime is scarier than a horror movie.
        
    * Recreate: Great for apps that need a clean slate or when a quick switcheroo is more important than constant availability.
        

## Your Kubernetes Command Spellbook for Deployments and Rollbacks

Here's a quick reference guide for all the magical incantations we've covered:

```bash
# Summon a new deployment
kubectl create deployment cafe-app --image=nginx:1.14.2

# Upgrade your deployment (Rolling Update style)
kubectl set image deployment/cafe-app nginx=nginx:1.16.1
# Or for the hands-on approach
kubectl edit deployment/cafe-app

# Upgrade with a Recreate strategy
# First, edit your deployment to set strategy.type to Recreate, then:
kubectl set image deployment/cafe-app nginx=nginx:1.16.1

# Check if your spell worked
kubectl rollout status deployment/cafe-app

# Pause mid-spell (careful, it's tricky!)
kubectl rollout pause deployment/cafe-app

# Resume your spell
kubectl rollout resume deployment/cafe-app

# Oops! Go back in time
kubectl rollout undo deployment/cafe-app

# Pick a specific moment in time to return to
kubectl rollout undo deployment/cafe-app --to-revision=2

# Check your spell history
kubectl rollout history deployment/cafe-app

# Make your deployment grow or shrink
kubectl scale deployment/cafe-app --replicas=5

# Get the full story on your deployment
kubectl describe deployment/cafe-app
```

## Wrapping Up: Your Journey to Kubernetes Deployment Mastery

And there you have it, folks! Rolling updates and recreate deployments aren't just boring technical concepts—they're the secret sauce that keeps our applications running smoothly in the wild world of Kubernetes.

As you continue your CKA journey, remember that these are practical tools you'll use daily in the Kubernetes environment. Mastering them is essential for any Kubernetes explorer!

Keep practicing, keep breaking things (in your test environment, of course!), and before you know it, you'll be updating and rolling back deployments like a Kubernetes wizard. 🧙‍♂️

Until next time, happy Kuberneting! May your pods always be healthy and your deployments forever smooth. 🚀