---
title: "My CKA Journey: A Simple Guide to Understanding Kubernetes Scheduling"
seoTitle: "Mastering Kubernetes Scheduling: A Beginner's Guide to CKA Success"
seoDescription: "Kubernetes scheduling with this beginner-friendly guide. Learn about manual scheduling, labels, taints, tolerations, node affinity, and resource limits."
datePublished: Sat Jul 06 2024 12:04:50 GMT+0000 (Coordinated Universal Time)
cuid: clya2uizc000i09mmc4l82uu8
slug: my-cka-journey-a-simple-guide-to-understanding-kubernetes-scheduling
tags: kubernetes, cka, certified-kubernetes-administrator, kubernetes-scheduling, manual-scheduling-kubernetes, labels-and-selectors-kubernetes, taints-and-tolerations-kubernetes, node-affinity-kubernetes, kubernetes-resource-limits, kubernetes-resource-requests, daemonsets-kubernetes, static-pods-kubernetes, kubernetes-cluster-management, kubernetes-node-management, kubernetes-best-practices

---

Hey there, fellow Kubernetes enthusiasts and curious minds! 👋

It's me again, your friendly neighborhood Kubernetes newbie, back with another update on my CKA (Certified Kubernetes Administrator) adventure. Grab a cup of coffee (or tea, if that's your jam), get comfy, and let me tell you about the wild ride I've been on this week. We're talking Kubernetes scheduling, and let me tell you, it's been a real brain-bender!

## 1\. Manual Scheduling: Taking Control of Pod Placement

Imagine you're organizing a party, and you have specific seating arrangements for your guests. Manual scheduling in Kubernetes is like assigning guests to specific seats at your party.

When I first heard about manual scheduling, I thought, "Why would I want to do that? Isn't Kubernetes supposed to handle this stuff?" But then I realized – sometimes, you just know better than the machine. Maybe you've got a super-powerful node that you want to reserve for specific tasks, or perhaps you're trying to keep certain pods close to each other for performance reasons.

### How to Use Manual Scheduling

Here's how you do it:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-special-pod
spec:
  containers:
  - name: my-app
    image: my-app:v1
  nodeName: my-powerful-node
```

See that `nodeName` field? That's where the magic happens. You're basically telling Kubernetes, "Hey, I don't care what you think, put this pod on this specific node!" The `nodeName` field specifies the exact node where you want your pod to run. It's like giving your pod a VIP pass to a particular node.

### Practical Insights:

But here's the thing – use this power wisely. It's like assigning VIP seats at a party. If everyone gets a VIP seat, it won't be special anymore, and you'll have a chaotic seating arrangement.

* **Use Case:** Manual scheduling is useful for debugging, testing, or specific scenarios where precise control over pod placement is required.
    
* **Drawbacks:** Overusing manual scheduling can lead to inefficient resource utilization and uneven load distribution across your cluster.
    

## 2\. Labels and Selectors: The Ultimate Organizational Hack

Next up, we've got labels and selectors. When I first encountered these, I had a lightbulb moment. It was like discovering color-coded folders after years of throwing everything into a single, messy drawer!

### Understanding Labels and Selectors

Labels are just key-value pairs you stick on your Kubernetes objects. It's like putting stickers on your stuff. For example:

```yaml
metadata:
  labels:
    app: web
    tier: frontend
    environment: production
```

Now, selectors are how you find stuff with specific labels. It's like having a magic wand that can summon all your "production" "web" apps with a flick of the wrist!

```yaml
spec:
  selector:
    app: web
    tier: frontend
```

This selector would match any Kubernetes object with both the `app: web` and `tier: frontend` labels. It's incredibly powerful for grouping related resources together.

### Practical Insights:

I've found these incredibly useful for organizing my deployments, services, and pretty much everything else in my cluster. Trust me, future you will thank present you for using labels generously!

* **Use Case:** Labels and selectors are essential for grouping and managing resources in large clusters, facilitating operations like scaling and updating.
    
* **Best Practices:** Use meaningful labels consistently across your resources to make your cluster easier to manage and navigate.
    

## 3\. Taints and Tolerations: The Bouncer at the Kubernetes Club

Alright, this one's a bit trickier, but stick with me. Imagine you're running an exclusive club, and you've got a bouncer at the door (that's your taint). The bouncer keeps everyone out unless they have a special VIP pass (that's your toleration).

### How Taints and Tolerations Work:

Taints are applied to nodes and prevent pods from being scheduled on them unless the pods have a matching toleration.

Here's how you taint a node:

```bash
kubectl taint nodes my-node special=true:NoSchedule
```

This is like telling your bouncer, "Don't let anyone in unless they've got a 'special' VIP pass."

And here's how you make a pod tolerate that taint:

```yaml
spec:
  tolerations:
  - key: "special"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
```

This is like giving your pod that VIP pass, saying, "I can handle that special node!"

### Types of Taints:

1. **NoSchedule:** The node will not accept any pods that do not tolerate the taint.
    
2. **PreferNoSchedule:** Kubernetes will try to avoid placing a pod on the node unless no other nodes are available.
    
3. **NoExecute:** Existing pods that do not tolerate the taint will be evicted, and new pods that do not tolerate the taint will not be scheduled.
    

### Practical Insights:

* **Use Case:** Taints and tolerations are perfect for isolating specific workloads, ensuring that only certain pods run on particular nodes, like dedicating nodes to high-priority tasks.
    
* **Drawbacks:** Taints and tolerations can be restrictive and may require careful management to avoid unintentional pod exclusion.
    

### Control Plane Nodes:

By default, Kubernetes taints control plane nodes to prevent workloads from being scheduled on them. This can be overridden if needed:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

This command removes the taint from all control plane nodes, allowing pods to be scheduled on them.

## 4\. Node Affinity: Finding the Perfect Match for Your Pods

If taints and tolerations are about keeping pods away from nodes, node affinity is all about attracting pods to nodes. It's like a dating app for pods and nodes!

### Types of Node Affinity:

1. **RequiredDuringSchedulingIgnoredDuringExecution:** Pods must be scheduled on nodes that match the specified criteria. If no such nodes are available, the pod will not be scheduled.
    
2. **PreferredDuringSchedulingIgnoredDuringExecution:** Kubernetes will try to schedule the pod on nodes that match the criteria but will still schedule it on other nodes if necessary.
    

### How Node Affinity Works:

Node affinity allows you to specify rules that attract pods to nodes with certain labels. It's more flexible and expressive than node selectors.

Let's say you've got some nodes with SSDs, and you want your database pods to run on those nodes. You could use node affinity to make that happen:

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: In
            values:
            - ssd
```

I know, I know, that looks like a mouthful. But let's break it down:

* `requiredDuringSchedulingIgnoredDuringExecution`: This is like saying, "I absolutely must have this, or I won't run at all!"
    
* `matchExpressions`: This is where you define what you're looking for.
    
* `key: disk`, `operator: In`, `values: [ssd]`: This is saying, "I want nodes that have SSDs."
    

### Practical Insights:

It's like being able to say, "I want an apartment with a dishwasher, within walking distance of a coffee shop, and it must allow pets!" But for pods.

* **Use Case:** Node affinity is ideal for ensuring pods are placed on nodes with specific characteristics, like hardware capabilities or geographic location.
    
* **Flexibility:** Unlike taints and tolerations, node affinity allows for both required and preferred rules, offering more nuanced control over pod placement.
    

### Combining Taints and Tolerations with Node Affinity: Crafting Specific Scheduling Rules

Using taints, tolerations, and node affinity together allows for creating precise scheduling rules that can be crucial for managing resources and workloads in a production environment. Here are some examples to help you understand how these concepts can be applied in real-world scenarios.

### Example 1: High-Performance Workloads

**Scenario:** You have a set of high-performance nodes with GPUs that are dedicated to machine learning (ML) workloads. You want to ensure that only ML pods are scheduled on these nodes, and other pods are kept off.

#### Step-by-Step Implementation

1. **Label the High-Performance Nodes:** First, label your GPU nodes so they can be identified by the node affinity rules.
    
    ```bash
    kubectl label nodes gpu-node-1 hardware=gpu
    kubectl label nodes gpu-node-2 hardware=gpu
    ```
    
2. **Taint the High-Performance Nodes:** Apply a taint to the GPU nodes to repel non-ML workloads.
    
    ```bash
    kubectl taint nodes gpu-node-1 dedicated=gpu:NoSchedule
    kubectl taint nodes gpu-node-2 dedicated=gpu:NoSchedule
    ```
    
3. **Create a Pod with Node Affinity and Tolerations:** Define a pod spec that uses node affinity to prefer GPU nodes and tolerations to allow scheduling on the tainted nodes.
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: ml-workload
    spec:
      containers:
      - name: ml-container
        image: ml-image:v1
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: hardware
                operator: In
                values:
                - gpu
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "gpu"
        effect: "NoSchedule"
    ```
    

In this example, the `ml-workload` pod is attracted to nodes with the `hardware=gpu` label due to the node affinity rule. Additionally, the pod can tolerate the `dedicated=gpu:NoSchedule` taint, ensuring it is allowed to run on those GPU nodes.

### Example 2: Isolation of Sensitive Workloads

**Scenario:** You have a set of nodes dedicated to running sensitive workloads that require extra security. You want to ensure that only pods with a specific security profile are scheduled on these nodes.

#### Step-by-Step Implementation

1. **Label the Sensitive Nodes:** Label your nodes to identify them for the sensitive workloads.
    
    ```bash
    kubectl label nodes sensitive-node-1 environment=secure
    kubectl label nodes sensitive-node-2 environment=secure
    ```
    
2. **Taint the Sensitive Nodes:** Apply a taint to these nodes to keep general workloads away.
    
    ```bash
    kubectl taint nodes sensitive-node-1 security=sensitive:NoSchedule
    kubectl taint nodes sensitive-node-2 security=sensitive:NoSchedule
    ```
    
3. **Create a Pod with Node Affinity and Tolerations:** Define a pod spec that uses node affinity to target the secure environment and tolerations to allow scheduling on the tainted nodes.
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: sensitive-app
    spec:
      containers:
      - name: sensitive-container
        image: secure-app-image:v1
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - secure
      tolerations:
      - key: "security"
        operator: "Equal"
        value: "sensitive"
        effect: "NoSchedule"
    ```
    

In this scenario, the `sensitive-app` pod is directed to nodes with the `environment=secure` label due to the node affinity rule. It also tolerates the `security=sensitive:NoSchedule` taint, ensuring it can run on these nodes despite the taint.

### Example 3: Staging vs. Production Environments

**Scenario:** You have nodes dedicated to staging and production environments. You want to ensure that staging pods do not run on production nodes and vice versa.

#### Step-by-Step Implementation

1. **Label the Staging and Production Nodes:** Label your nodes to distinguish between staging and production environments.
    
    ```bash
    kubectl label nodes staging-node-1 environment=staging
    kubectl label nodes production-node-1 environment=production
    ```
    
2. **Taint the Production Nodes:** Apply a taint to production nodes to repel staging workloads.
    
    ```bash
    kubectl taint nodes production-node-1 environment=production:NoSchedule
    ```
    
3. **Create Staging and Production Pods with Node Affinity and Tolerations:** Define pod specs that use node affinity and tolerations appropriately.
    
    **Staging Pod:**
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: staging-app
    spec:
      containers:
      - name: staging-container
        image: staging-app-image:v1
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - staging
      tolerations: []
    ```
    
    **Production Pod:**
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: production-app
    spec:
      containers:
      - name: production-container
        image: production-app-image:v1
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: environment
                operator: In
                values:
                - production
      tolerations:
      - key: "environment"
        operator: "Equal"
        value: "production"
        effect: "NoSchedule"
    ```
    

In this example, the `staging-app` pod is attracted to nodes labeled `environment=staging` and does not need any tolerations. The `production-app` pod is attracted to nodes labeled `environment=production` and tolerates the `environment=production:NoSchedule` taint to ensure it can be scheduled on production nodes.

### Practical Insights and Use Cases

* **High-Performance Workloads:** Combining taints, tolerations, and node affinity ensures that only appropriate workloads (e.g., ML tasks requiring GPUs) run on specialized hardware nodes.
    
* **Sensitive Workloads:** Use these techniques to isolate sensitive or critical workloads, ensuring they run only on designated secure nodes.
    
* **Environment Separation:** Maintain clear boundaries between different environments (e.g., staging and production) by using taints and tolerations alongside node affinity.
    

### Best Practices

* **Monitor Resource Usage:** Regularly monitor resource usage and adjust taints, tolerations, and node affinity rules as needed to optimize performance and resource utilization.
    
* **Document Rules:** Maintain clear documentation of your taint, toleration, and node affinity rules to ensure team members understand the scheduling logic.
    
* **Test Thoroughly:** Test your configurations in a staging environment before applying them to production to avoid disruptions.
    

By combining taints, tolerations, and node affinity, you can create highly specific and effective scheduling rules that cater to various production needs, enhancing the overall efficiency and reliability of your Kubernetes clusters.

## 5\. Resource Requirements and Limits: Being a Good Neighbor in the Kubernetes Community

Think of your Kubernetes cluster as a shared office with a limited number of desks (resources like CPU and memory). Resource requests and limits help ensure everyone gets a fair share of the resources they need to do their work, without any one person hogging everything or crashing the system.

#### What Are Resource Requests and Limits?

* **Resource Requests:** These are the minimum resources a pod needs to function. Think of it as reserving a desk in the office – you’re ensuring you have enough space to work comfortably.
    
* **Resource Limits:** These are the maximum resources a pod is allowed to use. It’s like agreeing not to take up more than a certain amount of space, even if you need to spread out a bit more sometimes.
    

Here’s what it looks like in a pod spec:

```yaml
spec:
  containers:
  - name: my-app
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### Practical Considerations and Examples

1. **Avoiding Resource Hogging:** Imagine you have an application that occasionally needs more CPU power. Setting a CPU limit ensures it doesn’t take over the entire cluster, leaving other applications starved for resources. However, setting the limit too low might cause performance issues for your app.
    
2. **Handling Memory Limits:** Unlike CPU, which can be throttled (slowed down) if a pod exceeds its limit, memory is a bit stricter. If a pod exceeds its memory limit, it will be killed and restarted with an "Out of Memory" (OOM) error. This is like getting kicked out of your desk because you brought too many things and cluttered the office.
    
3. **Balancing Requests and Limits:** It’s important to strike a balance. For instance, if you set memory limits too high, you might waste resources because other pods can’t use them even if they’re available. If you set them too low, critical applications might get killed frequently, disrupting your services.
    

#### Real-World Example: Web Application with Variable Load

Imagine you’re running a web application with multiple components:

* **Frontend:** Needs consistent but not heavy resources.
    
* **Backend API:** Can have spikes in CPU usage.
    
* **Database:** Requires a lot of memory but stable CPU usage.
    

Here’s how you might configure the resources:

**Frontend Pod:**

```yaml
spec:
  containers:
  - name: frontend
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

**Backend API Pod:**

```yaml
spec:
  containers:
  - name: backend
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
```

**Database Pod:**

```yaml
spec:
  containers:
  - name: database
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
```

### Key Points to Consider

1. **Testing and Monitoring:** Regularly monitor your applications to understand their resource usage patterns. Use tools like Prometheus and Grafana to visualize and analyze resource consumption. Adjust your requests and limits based on real data.
    
2. **Avoid Overcommitting:** Overcommitting resources can lead to contention and performance issues. If multiple pods exceed their requests simultaneously, it can cause resource starvation and affect the stability of your cluster.
    
3. **Graceful Degradation:** Design your applications to handle resource limits gracefully. For instance, if your backend API pod hits its CPU limit, it should throttle requests or degrade performance in a controlled manner, rather than crashing.
    
4. **Use Horizontal Pod Autoscaling:** Combine resource requests and limits with Horizontal Pod Autoscaling (HPA) to automatically scale the number of pod replicas based on resource usage. This ensures that your application can handle increased load without manual intervention.
    

## 6\. DaemonSets: The Omnipresent Pods

DaemonSets are pretty cool. They ensure that all (or some) nodes run a copy of a pod. When I first learned about these, I thought, "Why would I want that?" Then I realized they're perfect for things like log collectors or monitoring agents.

### How DaemonSets Work:

Here's a simple DaemonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      name: monitoring-agent
  template:
    metadata:
      labels:
        name: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent:v1
```

This will make sure every node in your cluster is running one copy of the monitoring-agent pod. It's like having a mini-me on every node, keeping an eye on things for you!

### Practical Insights:

* **Use Case:** DaemonSets are perfect for running system-level services like log collectors, monitoring agents, and network proxies.
    
* **Management:** DaemonSets are managed like any other Kubernetes workload but ensure that pods are distributed across all nodes, providing comprehensive coverage.
    

## 7\. Static Pods: The Independent Operators

Static pods are... different. They're created directly by the kubelet on a node, without the API server even knowing about them. It's like having a secret club that the principal doesn't know about!

### How Static Pods Work:

To create a static pod, you just drop a pod definition file in a special directory on a node (usually `/etc/kubernetes/manifests`, but it can vary). The kubelet sees it and says, "Oh, I should create this pod!"

Here's an example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-pod
spec:
  containers:
  - name: static-container
    image: static-image:v1
```

The kubelet will continuously monitor this file, and if it changes or is deleted, the pod will be updated or removed accordingly.

### Practical Insights:

* **Use Case:** Static pods are ideal for critical components like control plane pods that need to be always up and running.
    
* **Management:** The kubelet on the node manages static pods. The location of the directory where static pod definitions are placed is usually `/etc/kubernetes/manifests`. To remove a static pod, you simply delete its definition file from this directory.
    
* **Drawbacks:** Static pods can be tricky to manage and should be used judiciously. Since they are not managed by the API server, you don't have the usual Kubernetes management features like kubectl commands.
    

## 8\. Multiple Schedulers: Customizing the Traffic Control

Kubernetes allows you to run multiple schedulers simultaneously. This is advanced stuff, but it's like having multiple traffic controllers, each with their own rules for directing pods.

### How It Works:

To use a custom scheduler, you specify it in the pod definition:

```yaml
spec:
  schedulerName: my-custom-scheduler
```

This tells Kubernetes to use `my-custom-scheduler` for scheduling this pod.

### Practical Insights:

Multiple schedulers are powerful for environments with diverse scheduling needs. You can create custom schedulers tailored to specific workloads or operational requirements.

### Setting Up Multiple Schedulers:

1. **Deploy the Custom Scheduler:** Deploy your custom scheduler as a Kubernetes deployment.
    
2. **Specify the Scheduler Name:** Update the pod spec to use the custom scheduler by setting the `schedulerName` field.
    

```yaml
spec:
  schedulerName: my-custom-scheduler
```

3. **Use Case:** Multiple schedulers are useful in large clusters with varying scheduling requirements. For example, you might have a custom scheduler optimized for batch processing jobs while another handles latency-sensitive applications.
    

## Wrapping Up

That was a lot, wasn't it? But here's the thing – understanding these concepts has already made me feel so much more confident in managing Kubernetes clusters. Each topic we covered is like a new tool in my Kubernetes toolbox.

Remember, it's okay if all of this doesn't click immediately. Kubernetes is complex, and learning it is a journey. I'm still on that journey myself! The key is to keep practicing, keep asking questions, and most importantly, keep playing around with these concepts in a real cluster.

Next week, who knows what Kubernetes mysteries we'll unravel? But I'm excited to find out, and I hope you'll join me on this adventure. Until then, happy clustering! 🚀

#Kubernetes #CKA #LearningInPublic #TechJourney