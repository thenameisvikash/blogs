---
title: "Metrics Server on Kubernetes: A Practical Guide"
datePublished: Thu Aug 22 2024 05:04:02 GMT+0000 (Coordinated Universal Time)
cuid: cm04tieuo001909ie8ifbfmw0
slug: metrics-server-on-kubernetes-a-practical-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724302788124/5b9092d8-84a5-4059-ba99-be0f1569fe6f.png
tags: metrics-server-on-kubernetes, kubernetes-clusters, deploying-the-metrics-server, customizing-metrics-server-resources-for-production

---

Look, I get it. You're knee-deep in Kubernetes clusters, juggling pods like a circus performer, and now someone's asking about resource metrics. Cue the internal screaming, right? Been there, stressed about that. But here's the thing: setting up a metrics server doesn't have to be your next nightmare. In fact, it might just be the solution you didn't know you desperately needed.

Before we dive into the metric madness, if you're still getting your sea legs in the Kubernetes ocean, check out my previous article: [Building a Resilient Kubernetes Cluster: A Journey from Local to Production-Grade](https://devopswizard.hashnode.dev/building-a-resilient-kubernetes-cluster-a-journey-from-local-to-production-grade). It'll give you the foundation you need to make sense of what we're about to do.

## The Why: Metrics Matter (More Than You Think)

Picture this: It's 3 AM, and your phone's blowing up with alerts. Your app's slow, users are complaining, and you have no idea why. Without metrics, you're basically trying to fix a car engine in the dark. Been there, and let me tell you, it's not fun.

Metrics give you the superpower to see inside your cluster. They help you:

* Spot resource hogs before they become a problem
    
* Make smart decisions about scaling
    
* Justify that infrastructure upgrade to your boss (with actual data!)
    

## The What: Meet Your New Best Friend, the Metrics Server

The Metrics Server is like that quiet, efficient coworker who always has the answers. It collects CPU and memory usage data from all your containers and makes it available through the Kubernetes API. No fuss, no muss!

## The How: Let's Roll Up Our Sleeves

Alright, enough theory. Let's get our hands dirty. I've cooked up a script that'll make setting up the Metrics Server a breeze. Here's how we do it, step by step:

1. First, save this script as `setup_metrics_`[`server.sh`](http://server.sh):
    

```bash
#!/bin/bash

# Function to check if the command was successful
check_command() {
    if [ $? -ne 0 ]; then
        echo "Error: $1 failed. Exiting."
        exit 1
    fi
}

# Step 1: Apply the Metrics Server manifest
echo "Deploying Metrics Server..."
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
check_command "Deploying Metrics Server"

# Step 2: Wait for the Metrics Server deployment to be created
echo "Waiting for Metrics Server deployment to be created..."
sleep 10

# Step 3: Modify the Metrics Server deployment to add --kubelet-insecure-tls
echo "Modifying Metrics Server deployment to add --kubelet-insecure-tls..."
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
check_command "Patching Metrics Server deployment"

# Step 4: Wait for the Metrics Server to rollout the updated deployment
echo "Waiting for the Metrics Server to roll out..."
kubectl rollout status deployment metrics-server -n kube-system
check_command "Rolling out Metrics Server deployment"

# Step 5: Verify the Metrics Server is running
echo "Verifying Metrics Server status..."
kubectl get deployment metrics-server -n kube-system
check_command "Verifying Metrics Server"

echo "Metrics Server deployed and configured successfully!"
```

2. Make the script executable:
    
    ```bash
    chmod +x setup_metrics_server.sh
    ```
    
3. Run the script:
    
    ```bash
    ./setup_metrics_server.sh
    ```
    

Now, let's break down what's happening:

### Step 1: Deploying the Metrics Server

We're grabbing the latest version of the Metrics Server manifest from GitHub. It's like getting the newest iPhone on release day—always good to have the latest and greatest.

### Step 2: Waiting for Deployment

We give Kubernetes a moment to catch its breath. Ten seconds might seem arbitrary, but in my experience, it's usually enough time for the deployment to initialize.

### Step 3: Configuring for Insecure TLS

This is where we add the `--kubelet-insecure-tls` flag. It's like telling your Metrics Server, "Yeah, I know this connection isn't super secure, but we're all friends here." In production, you'll want proper certificates, but this gets you up and running quickly.

### Step 4: Rolling Out the Update

We wait for the changes to take effect. It's like waiting for your coffee to brew—patience is key.

### Step 5: Verification

Finally, we check if everything's running smoothly. If you see the deployment with available replicas, you're golden! You can double-check with:

```bash
kubectl get deployment metrics-server -n kube-system
```

## On-Prem vs. Cloud: Different Playgrounds, Same Game

Whether you're running Kubernetes in your basement or on a fancy cloud platform, this script's got you covered. But keep in mind:

### On-Premises:

* You might need to tweak firewall rules. It's like introducing your Metrics Server to the rest of your network—sometimes, formal introductions are necessary.
    

### Cloud:

* Check if your cloud provider already offers a metrics solution. No need to reinvent the wheel if it's already rolling.
    

## Tips and Tricks: Level Up Your Game

1. **Beginner Tip**: Always check the logs if something goes wrong. It's like being a detective in a cyber-crime novel.
    
    ```bash
    kubectl logs -n kube-system deployment/metrics-server
    ```
    
2. **Intermediate Trick**: For better security in production, set up proper TLS certificates. It's like upgrading from a padlock to a state-of-the-art security system.
    
3. **Advanced Move**: Customize resource requests and limits for the Metrics Server based on your cluster size. Let's dive deeper into this one.
    

### Customizing Metrics Server Resources for Production

In production, one size doesn't fit all. Here's how to tailor your Metrics Server:

1. First, let's see the current resource settings:
    
    ```bash
    kubectl get deployment metrics-server -n kube-system -o yaml | grep resources: -A 8
    ```
    
2. Now, let's customize it. Create a file named `metrics-server-resources.yaml`:
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: metrics-server
      namespace: kube-system
    spec:
      template:
        spec:
          containers:
          - name: metrics-server
            resources:
              requests:
                cpu: 100m
                memory: 200Mi
              limits:
                cpu: 300m
                memory: 500Mi
    ```
    
    Adjust these values based on your cluster size. For a large cluster (100+ nodes), you might go with:
    
    ```yaml
    requests:
      cpu: 200m
      memory: 300Mi
    limits:
      cpu: 500m
      memory: 1Gi
    ```
    
3. Apply your changes:
    
    ```bash
    kubectl apply -f metrics-server-resources.yaml
    ```
    
4. Verify the update:
    
    ```bash
    kubectl get deployment metrics-server -n kube-system -o yaml | grep resources: -A 8
    ```
    

Remember, finding the right values might take some trial and error. It's like tuning a guitar—you need to listen and adjust until it sounds just right.

4. **Pro Tip**: Set up monitoring for the Metrics Server itself. Because who watches the watchmen, right?
    

Let's dive deeper into this pro tip. Monitoring your Metrics Server is crucial for ensuring the reliability of your entire Kubernetes monitoring stack. After all, if your Metrics Server goes down, you're flying blind.

### Why Monitor the Metrics Server?

1. **Ensure Availability**: You want to know immediately if the Metrics Server stops functioning.
    
2. **Performance Tracking**: Monitor its resource usage to prevent it from becoming a bottleneck.
    
3. **Troubleshooting**: Historical data can be invaluable when diagnosing issues.
    

### How to Monitor the Metrics Server

There are several approaches you can take:

#### 1\. Use Prometheus and Grafana

This is a popular combo in the Kubernetes world. Here's how to set it up:

a. Install Prometheus Operator:

```bash
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/master/bundle.yaml
```

b. Create a ServiceMonitor for the Metrics Server:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: metrics-server-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      kubernetes.io/name: "Metrics-server"
  namespaceSelector:
    matchNames:
    - kube-system
  endpoints:
  - port: https
    scheme: https
    bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
    tlsConfig:
      insecureSkipVerify: true
```

Apply this with:

```bash
kubectl apply -f metrics-server-monitor.yaml
```

c. Set up Grafana and create a dashboard to visualize the metrics.

#### 2\. Use Built-in Kubernetes Tools

You can use `kubectl` to quickly check the status:

```bash
kubectl get deployment metrics-server -n kube-system
kubectl describe deployment metrics-server -n kube-system
```

For more detailed metrics:

```bash
kubectl top pod -n kube-system | grep metrics-server
```

#### 3\. Custom Scripts and Alerts

Write a simple script to check the Metrics Server's health and send alerts:

```bash
#!/bin/bash

if ! kubectl get deployment metrics-server -n kube-system &> /dev/null; then
    echo "ALERT: Metrics Server is not deployed!" | mail -s "Metrics Server Down" your@email.com
fi

READY=$(kubectl get deployment metrics-server -n kube-system -o jsonpath='{.status.readyReplicas}')
if [ "$READY" != "1" ]; then
    echo "ALERT: Metrics Server is not ready!" | mail -s "Metrics Server Not Ready" your@email.com
fi
```

Run this script regularly using a cron job.

### What to Monitor

1. **CPU and Memory Usage**: Ensure the Metrics Server isn't resource-starved.
    
2. **Request Latency**: High latency could indicate performance issues.
    
3. **Error Rates**: Sudden spikes in errors could signal problems.
    
4. **Scrape Duration**: How long it takes to collect metrics from all nodes.
    

### Best Practices

1. **Set Up Alerts**: Don't just collect data, act on it. Set up alerts for critical thresholds.
    
2. **Regular Health Checks**: Implement liveness and readiness probes for the Metrics Server.
    
3. **Version Tracking**: Keep track of the Metrics Server version and update regularly.
    
4. **Log Analysis**: Don't forget to monitor and analyze the Metrics Server logs.
    

By implementing these monitoring strategies, you'll ensure that your Metrics Server – and by extension, your entire Kubernetes monitoring setup – remains healthy and reliable. Remember, in the world of Kubernetes, even the watchers need watching!

**Expert Advice**: In large, multi-tenant clusters, use the `--kubelet-preferred-address-types` flag to optimize performance. It's like giving your Metrics Server a roadmap to find the quickest route to each kubelet.

## Wrapping Up

There you have it, folks! From zero to hero in Metrics Server deployment. Remember, understanding your cluster's resource usage is key to running a tight ship. With these metrics at your fingertips, you'll be the captain your Kubernetes fleet deserves.

Next time your boss asks about cluster performance, you'll be ready with charts, graphs, and insights that'll make their jaw drop. So go ahead, give this setup a whirl, and start your journey to becoming a Kubernetes metrics maestro!

Keep sailing those containerized seas, and may your metrics always be insightful and your deployments forever smooth!