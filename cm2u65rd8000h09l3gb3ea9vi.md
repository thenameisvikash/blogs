---
title: "Mastering Kubernetes Errors: A Complete Troubleshooting Guide"
seoTitle: "Kubernetes Error Troubleshooting Guide"
seoDescription: "Learn to troubleshoot Kubernetes errors with this comprehensive guide covering HTTP and container exit codes, pod statuses, node errors, and more"
datePublished: Tue Oct 29 2024 08:11:46 GMT+0000 (Coordinated Universal Time)
cuid: cm2u65rd8000h09l3gb3ea9vi
slug: mastering-kubernetes-errors-a-complete-troubleshooting-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1730189348601/83140692-80eb-4618-90c6-24cc2ae45003.png
tags: kubernetes-errors

---

Hello , Kubernetes warriors! 👋

After spending countless hours debugging Kubernetes issues (and drinking way too much coffee), I've put together this comprehensive guide to every error code you might encounter in your Kubernetes journey. Whether you're battling with container exit codes, HTTP errors, or mysterious pod states, this guide has got you covered.

## Table of Contents

1. [HTTP API Error Codes](#http-api-error-codes)
    
2. [Container Exit Codes](#container-exit-codes)
    
3. [Pod Status Errors](#pod-status-errors)
    
4. [Node-Related Errors](#node-related-errors)
    
5. [Networking Errors](#networking-errors)
    
6. [Storage and Volume Errors](#storage-and-volume-errors)
    
7. [RBAC and Authentication Errors](#rbac-and-authentication-errors)
    
8. [The Ultimate Debugging Arsenal](#the-ultimate-debugging-arsenal)
    

## HTTP API Error Codes {#http-api-error-codes}

When interacting with the Kubernetes API, you'll encounter these HTTP status codes:

| Error Code | Description | Common Cause | Solution |
| --- | --- | --- | --- |
| 400 | Bad Request | Malformed YAML/JSON | Validate your manifest syntax |
| 401 | Unauthorized | Invalid/expired credentials | Check/renew your kubeconfig |
| 403 | Forbidden | Insufficient RBAC permissions | Review role bindings |
| 404 | Not Found | Resource doesn't exist | Check resource name/namespace |
| 409 | Conflict | Resource state conflict | Retry or check resource version |
| 422 | Validation error | Invalid resource definition | Check API spec compliance |
| 429 | Too Many Requests | API rate limit exceeded | Implement request throttling |
| 500 | Internal Server Error | API server issues | Check control plane health |
| 503 | Service Unavailable | API server overload | Check cluster resources |
| 504 | Gateway Timeout | API server timeout | Check network connectivity |

### Debugging HTTP Errors

```bash
# Check API server health
kubectl get --raw /healthz

# Verify API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Test API connectivity
curl -k -v https://<kubernetes-api-server>:6443/api
```

## Container Exit Codes {#container-exit-codes}

These are the last words of your containers - learn to speak their language!

### Common Exit Codes

| Exit Code | Signal | Description | Typical Cause | Solution |
| --- | --- | --- | --- | --- |
| 0 | \- | Success | Normal termination | All good! |
| 1 | \- | General error | Application error | Check app logs |
| 2 | \- | Misuse of shell builtins | Shell command errors | Review Dockerfile |
| 125 | \- | Container failed to run | Command error | Check container config |
| 126 | \- | Command not executable | Permission issues | Check file permissions |
| 127 | \- | Command not found | Missing executable | Verify PATH/dependencies |
| 128 | \- | Invalid exit argument | Bad exit calls | Review exit handling |
| 137 | SIGKILL | Container killed | OOM/external kill | Check memory limits |
| 139 | SIGSEGV | Segmentation fault | Memory corruption | Debug app code |
| 143 | SIGTERM | Graceful termination | Normal shutdown | Implement shutdown hooks |
| 255 | \- | Exit status out of range | Invalid exit code | Fix exit handling |

### Signal-Based Exit Codes

When you see exit code 128+n, it means the container received signal n:

| Exit Code | Signal | Meaning | Common Cause |
| --- | --- | --- | --- |
| 130 (128+2) | SIGINT | Interrupt | User interrupted |
| 137 (128+9) | SIGKILL | Kill | OOM/force kill |
| 138 (128+10) | SIGBUS | Bus error | Hardware issues |
| 139 (128+11) | SIGSEGV | Segfault | Memory violation |
| 141 (128+13) | SIGPIPE | Broken pipe | IPC failure |
| 143 (128+15) | SIGTERM | Termination | Grace period expired |

### Exit Code Debugging Script

```bash
#!/bin/bash
# save as k8s-debug.sh

pod_name=$1
namespace=$2

echo "🔍 Investigating pod: $pod_name in namespace: $namespace"

# Get exit code and container name
container_status=$(kubectl get pod $pod_name -n $namespace -o json | jq '.status.containerStatuses[0]')
exit_code=$(echo $container_status | jq '.lastState.terminated.exitCode')
container_name=$(echo $container_status | jq -r '.name')

echo "📊 Container: $container_name, Exit Code: $exit_code"

case $exit_code in
    137)
        echo "💥 OOM Kill detected. Checking memory..."
        kubectl describe pod $pod_name -n $namespace | grep -A 5 "Memory:"
        echo "📈 Last known memory usage:"
        kubectl top pod $pod_name -n $namespace
        ;;
    143)
        echo "👋 Graceful termination. Checking termination logs..."
        kubectl logs $pod_name -n $namespace --previous
        echo "⏱️ Checking termination configuration:"
        kubectl get pod $pod_name -n $namespace -o jsonpath='{.spec.terminationGracePeriodSeconds}'
        ;;
    1)
        echo "❌ Application error. Checking logs..."
        kubectl logs $pod_name -n $namespace --previous
        echo "🔍 Checking environment variables:"
        kubectl exec $pod_name -n $namespace -- env
        ;;
    *)
        echo "⚠️ Unexpected exit code. Gathering all information..."
        kubectl describe pod $pod_name -n $namespace
        kubectl logs $pod_name -n $namespace --previous
        ;;
esac
```

## Pod Status Errors {#pod-status-errors}

### ImagePullBackOff

The container image can't be pulled. Common causes:

* Incorrect image name/tag
    
* Private registry authentication issues
    
* Network connectivity problems
    

```bash
# Check image pull secrets
kubectl get secrets

# Verify registry access
kubectl describe pod <pod-name> | grep "Events:"
```

### CrashLoopBackOff

The pod keeps crashing and restarting. Debug with:

```bash
# Check previous container logs
kubectl logs <pod-name> --previous

# Check resource usage
kubectl top pod <pod-name>

# Look for crash patterns
kubectl get events --field-selector involvedObject.name=<pod-name>
```

### Error Status Codes

```yaml
Events:
  Type     Reason     Age   From               Message
  ----     ------     ----  ----               -------
  Warning  Failed     2m    kubelet            Error: ImagePullBackOff
  Normal   Scheduled  3m    default-scheduler  Successfully assigned default/nginx-pod to node-1
  Warning  Failed     1m    kubelet            Error: CrashLoopBackOff
```

## Node-Related Errors {#node-related-errors}

### NodeNotReady

When your node isn't healthy:

```bash
# Check node status
kubectl describe node <node-name>

# Verify kubelet
systemctl status kubelet

# Check node pressure
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

### Common Node Error States

| State | Description | Check Command |
| --- | --- | --- |
| NotReady | Node unhealthy | `kubectl describe node` |
| MemoryPressure | Low memory | `kubectl top node` |
| DiskPressure | Low disk space | `df -h` on node |
| NetworkUnavailable | Network issues | `kubectl get pods -n kube-system` |

## Networking Errors {#networking-errors}

### DNS Resolution Issues

```bash
# Test DNS resolution
kubectl run test-dns --image=busybox:1.28 -- nslookup kubernetes.default

# Check CoreDNS pods
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Verify service endpoints
kubectl get endpoints kubernetes
```

### Service Connection Problems

```bash
# Test service connectivity
kubectl run test-net --image=busybox -it --rm -- wget -O- http://service-name

# Check service definition
kubectl describe service <service-name>

# Verify pod selectors
kubectl get pods --show-labels
```

## Storage and Volume Errors {#storage-and-volume-errors}

### PersistentVolume Issues

```bash
# Check PV status
kubectl get pv,pvc

# Verify storage class
kubectl get storageclass

# Debug volume mounts
kubectl describe pod <pod-name> | grep -A 10 Volumes
```

### Common Storage Errors

| Error | Description | Solution |
| --- | --- | --- |
| FailedMount | Volume mount failed | Check PV/PVC binding |
| FailedAttachVolume | Volume attachment failed | Verify storage provider |
| VolumeBindingFailed | PVC binding failed | Check storage class |

## RBAC and Authentication Errors {#rbac-and-authentication-errors}

### Permission Denied (403)

```bash
# Check current permissions
kubectl auth can-i <verb> <resource>

# View role bindings
kubectl get rolebindings,clusterrolebindings

# Debug service account
kubectl describe serviceaccount <name>
```

### Certificate Issues

```bash
# Check cert expiration
kubeadm certs check-expiration

# Verify cert paths
ls -la /etc/kubernetes/pki/

# View certificate details
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
```

## The Ultimate Debugging Arsenal {#the-ultimate-debugging-arsenal}

### Essential Commands

```bash
# Get all resources in a namespace
kubectl get all -n <namespace>

# Watch resource changes
kubectl get pods -w

# Get detailed events
kubectl get events --sort-by=.metadata.creationTimestamp

# Debug with ephemeral container
kubectl debug <pod-name> -it --image=ubuntu
```

### Advanced Debugging Techniques

```bash
# Port forward to service
kubectl port-forward svc/<service-name> 8080:80

# Copy files from pod
kubectl cp <pod-name>:/path/to/file ./local-file

# Execute commands in pod
kubectl exec -it <pod-name> -- sh

# View resource utilization
kubectl top pods --containers
```

### Monitoring and Alerts

Set up alerts for:

* Pod restarts &gt; threshold
    
* Memory usage &gt; 80%
    
* Persistent volume usage &gt; 85%
    
* Node NotReady state
    
* Service endpoint count &lt; expected
    

## Best Practices for Error Prevention

1. **Resource Management**
    
    * Set appropriate resource requests/limits
        
    * Implement horizontal pod autoscaling
        
    * Monitor resource usage trends
        
2. **High Availability**
    
    * Use pod disruption budgets
        
    * Implement readiness/liveness probes
        
    * Set up proper node anti-affinity
        
3. **Monitoring**
    
    * Deploy logging aggregation
        
    * Set up metrics collection
        
    * Implement alerting
        
    * Use distributed tracing
        
4. **Security**
    
    * Regular certificate rotation
        
    * Proper RBAC configuration
        
    * Network policy implementation
        
    * Image vulnerability scanning
        

## Final Thoughts

Remember, Kubernetes errors are like onions - they have layers. Don't just treat the symptoms; understand the root cause. Keep this guide handy, and you'll be troubleshooting like a pro! 🚀

### Quick Reference Card

Print this out and stick it on your monitor:

```plaintext
🔍 Quick Debug Commands:
kubectl get pods          # List pods
kubectl describe pod      # Detail view
kubectl logs             # Container logs
kubectl exec -it         # Shell access
kubectl get events       # Cluster events
```

Happy debugging! Remember: it's not a bug, it's an unexpected learning opportunity! 😉

---

*About the author: A battle-scarred DevOps engineer who has debugged more pods than had hot dinners.*

#Kubernetes #DevOps #Troubleshooting #CloudNative #TechBlog