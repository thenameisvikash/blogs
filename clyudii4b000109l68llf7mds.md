---
title: "Understanding Kubernetes Certificates: Full Guide to Certificates API, Solutions, and KubeConfig"
datePublished: Sat Jul 20 2024 16:58:49 GMT+0000 (Coordinated Universal Time)
cuid: clyudii4b000109l68llf7mds
slug: understanding-kubernetes-certificates-full-guide-to-certificates-api-solutions-and-kubeconfig
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721494520784/293cc488-0470-468f-a53b-a93494c1f41f.png
tags: service-account, kubeconfig, working-with-the-certificates-api, create-a-role-and-rolebinding, anatomy-of-a-kubeconfig-file

---

Welcome, Kubernetes enthusiasts, to our deep dive into the world of Kubernetes certificates! In this article, we will explore the Certificates API, walk through practical solutions, and master the intricacies of KubeConfig. This journey will equip you with the knowledge to manage Kubernetes certificates effectively, leverage advanced techniques, and apply real-world scenarios.

For continuity, you might want to read the previous article in this series: [Everything You Need to Know About Kubernetes Certificates](https://devopswizard.hashnode.dev/everything-you-need-to-know-about-kubernetes-certificates-with-kubecorp).

**Table of Contents**

1. [The Certificates API: Your Certificate Management Powerhouse](#the-certificates-api-your-certificate-management-powerhouse)
    
2. [Practical Solutions Using the Certificates API](#practical-solutions-using-the-certificates-api)
    
3. [KubeConfig: Your Gateway to Kubernetes Clusters](#kubeconfig-your-gateway-to-kubernetes-clusters)
    
4. [Advanced KubeConfig Techniques](#advanced-kubeconfig-techniques)
    
5. [Putting It All Together: A Real-World Scenario](#putting-it-all-together-a-real-world-scenario)
    
6. [Conclusion and Next Steps](#conclusion-and-next-steps)
    
7. [References and Further Reading](#references-and-further-reading)
    

## The Certificates API: Your Certificate Management Powerhouse

The Certificates API is a fundamental part of Kubernetes' security infrastructure. It enables programmatic handling of certificate signing requests (CSRs) within a Kubernetes cluster. Let’s delve into its key features and usage.

### Understanding the Certificates API

The Certificates API revolves around the `CertificateSigningRequest` (CSR) resource. This resource represents a request to have a certificate signed by the Kubernetes cluster's Certificate Authority (CA).

**Structure of a CSR**

Here’s a sample YAML configuration for a CSR:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # Optional
  usages:
  - client auth
```

**Explanation:**

* `name`: A unique name for the CSR.
    
* `request`: The CSR in base64 encoded format.
    
* `signerName`: Specifies the signer that will sign the certificate.
    
* `expirationSeconds`: Optional field specifying the certificate’s duration.
    
* `usages`: Defines how the certificate can be used (e.g., client authentication).
    

### Working with the Certificates API

**1\. Creating a CSR**

Generate a private key and CSR using OpenSSL:

```bash
openssl genrsa -out john.key 2048
openssl req -new -key john.key -out john.csr -subj "/CN=john/O=developers"
```

Create the CSR resource in Kubernetes:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: $(cat john.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages:
  - client auth
EOF
```

**2\. Viewing CSRs**

List all CSRs:

```bash
kubectl get csr
```

View details of a specific CSR:

```bash
kubectl get csr john-developer -o yaml
```

**3\. Approving or Denying CSRs**

Approve a CSR:

```bash
kubectl certificate approve john-developer
```

Deny a CSR:

```bash
kubectl certificate deny john-developer
```

**4\. Retrieving the Signed Certificate**

Retrieve the signed certificate after approval:

```bash
kubectl get csr john-developer -o jsonpath='{.status.certificate}' | base64 -d > john.crt
```

### Advanced Certificates API Features

**1\. Certificate Rotation**

To rotate a certificate, create a new CSR with the same name:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: $(openssl req -new -key john.key -out john.csr -subj "/CN=john/O=developers" 2>/dev/null | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages:
  - client auth
EOF
```

**2\. Custom Signers**

Use custom signers by specifying a different `signerName`:

```yaml
spec:
  signerName: example.com/my-custom-signer
```

This feature allows integration with external certificate management systems.

## Practical Solutions Using the Certificates API

Now that we have a solid understanding of the Certificates API, let's explore some practical solutions.

### Solution 1: Creating a New User with Specific Permissions

Create a new user named "jane" and grant her specific permissions.

**1\. Generate Private Key and CSR:**

```bash
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -out jane.csr -subj "/CN=jane/O=devops"
```

**2\. Create the CSR Resource:**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane-devops
spec:
  request: $(cat jane.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages:
  - client auth
EOF
```

**3\. Approve the CSR:**

```bash
kubectl certificate approve jane-devops
```

**4\. Retrieve the Signed Certificate:**

```bash
kubectl get csr jane-devops -o jsonpath='{.status.certificate}' | base64 -d > jane.crt
```

**5\. Create a Role and RoleBinding:**

```bash
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods
kubectl create rolebinding jane-pod-reader --role=pod-reader --user=jane
```

Jane now has a valid certificate and can read pods in the cluster.

### Solution 2: Certificate Rotation for a Service Account

Rotate the certificate for a service account named "web-service."

**1\. Generate a New Private Key and CSR:**

```bash
openssl genrsa -out web-service-new.key 2048
openssl req -new -key web-service-new.key -out web-service-new.csr -subj "/CN=system:serviceaccount:default:web-service"
```

**2\. Create a New CSR:**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: web-service-renewal
spec:
  request: $(cat web-service-new.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kubelet-serving
  usages:
  - server auth
EOF
```

**3\. Approve the CSR:**

```bash
kubectl certificate approve web-service-renewal
```

**4\. Retrieve the New Certificate:**

```bash
kubectl get csr web-service-renewal -o jsonpath='{.status.certificate}' | base64 -d > web-service-new.crt
```

**5\. Update the Service Account:**

Update the service account with the new certificate (adjust based on your configuration).

## KubeConfig: Your Gateway to Kubernetes Clusters

KubeConfig files are essential for accessing and managing Kubernetes clusters. They contain all the information needed to connect to one or more clusters, including authentication details.

### Anatomy of a KubeConfig File

A typical KubeConfig file includes three main sections:

1. **Clusters:** Defines the clusters you have access to.
    
2. **Users:** Specifies the user credentials for each cluster.
    
3. **Contexts:** Combines clusters and users, optionally with a namespace.
    

**Sample KubeConfig File:**

```yaml
apiVersion: v1
kind: Config
clusters:
- name: development
  cluster:
    server: https://1.2.3.4
    certificate-authority-data: <base64-encoded-ca-cert>
users:
- name: jane
  user:
    client-certificate-data: <base64-encoded-cert>
    client-key-data: <base64-encoded-key>
contexts:
- name: jane-dev
  context:
    cluster: development
    user: jane
    namespace: devops
current-context: jane-dev
```

Another sample KubeConfig file for different environments and clusters:

```yaml
apiVersion: v1
kind: Config
clusters:
  - name: dev-cluster
    cluster:
      server: https://dev-cluster.example.com
      certificate-authority-data: <base64-encoded-dev-ca-cert>
  - name: staging-cluster
    cluster:
      server: https://staging-cluster.example.com
      certificate-authority-data: <base64-encoded-staging-ca-cert>
  - name: prod-cluster
    cluster:
      server: https://prod-cluster.example.com
      certificate-authority-data: <base64-encoded-prod-ca-cert>

users:
  - name: dev-user
    user:
      client-certificate-data: <base64-encoded-dev-user-cert>
      client-key-data: <base64-encoded-dev-user-key>
  - name: staging-user
    user:
      client-certificate-data: <base64-encoded-staging-user-cert>
      client-key-data: <base64-encoded-staging-user-key>
  - name: prod-user
    user:
      client-certificate-data: <base64-encoded-prod-user-cert>
      client-key-data: <base64-encoded-prod-user-key>

contexts:
  - name: dev-context
    context:
      cluster: dev-cluster
      user: dev-user
      namespace: dev
  - name: staging-context
    context:
      cluster: staging-cluster
      user: staging-user
      namespace: staging
  - name: prod-context
    context:
      cluster: prod-cluster
      user: prod-user
      namespace: production

current-context: dev-context
```

### Working with KubeConfig

**1\. Viewing KubeConfig**

To view your current KubeConfig:

```bash
kubectl config view
```

To see the full configuration with all certificate data:

```bash
kubectl config view --raw
```

**2\. Adding a New Cluster**

```bash
kubectl config set-cluster production --server=https://5.6.7.8 --certificate-authority=/path/to/ca.crt
```

**3\. Adding User Credentials**

```bash
kubectl config set-credentials john --client-certificate=/path/to/john.crt --client-key=/path/to/john.key
```

\*\*4. Setting Contexts

\*\*

```bash
kubectl config set-context john-prod --cluster=production --user=john --namespace=production
```

**5\. Switching Contexts**

```bash
kubectl config use-context john-prod
```

## Advanced KubeConfig Techniques

### Merging Multiple KubeConfig Files

If you have multiple KubeConfig files, you can merge them using the `KUBECONFIG` environment variable.

```bash
export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/prod-config
kubectl config view --merge --flatten > $HOME/.kube/merged-config
export KUBECONFIG=$HOME/.kube/merged-config
```

### Using KubeConfig with External Tools

Many external tools, such as `kubectl`, `helm`, and `kustomize`, rely on KubeConfig for cluster access. Ensure your KubeConfig file is correctly set up and accessible by these tools.

## Putting It All Together: A Real-World Scenario

Let’s apply our knowledge in a real-world scenario. Imagine you are setting up a CI/CD pipeline for a new application. You need to create a new user with specific permissions, set up their access using KubeConfig, and ensure secure communication.

### Steps:

1. **Create a New User and Generate Certificates:**
    
    ```bash
    openssl genrsa -out ci-user.key 2048
    openssl req -new -key ci-user.key -out ci-user.csr -subj "/CN=ci-user/O=ci-cd"
    ```
    
2. **Create and Approve the CSR:**
    
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: ci-user-csr
    spec:
      request: $(cat ci-user.csr | base64 | tr -d '\n')
      signerName: kubernetes.io/kube-apiserver-client
      usages:
      - client auth
    EOF
    
    kubectl certificate approve ci-user-csr
    ```
    
3. **Retrieve the Signed Certificate:**
    
    ```bash
    kubectl get csr ci-user-csr -o jsonpath='{.status.certificate}' | base64 -d > ci-user.crt
    ```
    
4. **Set Up the KubeConfig File:**
    
    ```bash
    kubectl config set-cluster dev-cluster --server=https://1.2.3.4 --certificate-authority=ca.crt
    kubectl config set-credentials ci-user --client-certificate=ci-user.crt --client-key=ci-user.key
    kubectl config set-context ci-context --cluster=dev-cluster --user=ci-user --namespace=ci-cd
    kubectl config use-context ci-context
    ```
    
5. **Test Access:**
    
    ```bash
    kubectl get pods
    ```
    

This scenario ties together CSR creation, approval, certificate retrieval, and KubeConfig setup, demonstrating a practical application of the concepts discussed.

## Conclusion and Next Steps

In this article, we’ve explored the depths of Kubernetes certificates, the Certificates API, and KubeConfig. By mastering these tools, you can secure your Kubernetes clusters and manage access with confidence.

For more detailed insights, check out my previous article on this topic: [Everything You Need to Know About Kubernetes Certificates](https://devopswizard.hashnode.dev/everything-you-need-to-know-about-kubernetes-certificates-with-kubecorp).

## References and Further Reading

* [Kubernetes Official Documentation](https://kubernetes.io/docs/)
    
* [Kubernetes Certificates API](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
    
* [Managing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)
    
* [OpenSSL Documentation](https://www.openssl.org/docs/)