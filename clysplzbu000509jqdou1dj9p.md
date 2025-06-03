---
title: "Everything You Need to Know About Kubernetes Certificates with KubeCorp"
datePublished: Fri Jul 19 2024 13:01:54 GMT+0000 (Coordinated Universal Time)
cuid: clysplzbu000509jqdou1dj9p
slug: everything-you-need-to-know-about-kubernetes-certificates-with-kubecorp
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721393758843/cdf3a3dd-f4bb-4c7e-96fa-c9b35cf1b301.png
tags: kubernetes-security, kubernetes-certificates, kubernetes-api-server, how-to-manage-kubernetes-certificates, understanding-kubernetes-client-and-server-certificates, step-by-step-guide-to-kubernetes-certificate-management, role-of-certificates-in-kubernetes-security

---

Hey there, future Kubernetes wizards! 👋 Ready to unravel the mystery of Kubernetes certificates? Don't worry if it sounds scary - we're going to break it down using a fun analogy that'll make you feel right at home. let's dive into the world of Kubernetes security!

## The KubeCorp Office: Our Kubernetes Playground

Imagine Kubernetes as a big, bustling office called KubeCorp. This office has different departments, security checkpoints, and a whole lot of people trying to get work done. Sound familiar? Great! Let's take a walk through KubeCorp and see how it relates to our Kubernetes world.

## Meet the Players: Certificates in Kubernetes

Before we start our tour, let's quickly meet the main characters in our story:

* **ID Badges (Client Certificates)**: These are like your office ID. They prove you're allowed to be in certain areas of the building.
    
* **Security Guard Badges (Server Certificates)**: These are worn by security guards to prove they're real guards, not imposters.
    
* **The Big Boss's Signature (CA Certificates)**: This is like the company owner's signature, used to make all other badges official.
    

Now, let's start our day at KubeCorp!

## 9:00 AM: The Grand Entrance - Authenticating with the API Server

You walk up to the KubeCorp building and see Alice, the head of security, standing at the main entrance. In Kubernetes, Alice is like the API server - the main entry point for all communication.

Alice: "Good morning! Can I see your ID badge, please?"

You show Alice your shiny new employee badge. In Kubernetes, this is your client certificate. It's like your passport to the Kubernetes world, proving you're allowed to access certain things.

Alice then shows you her own special badge. This is like the API server's certificate in Kubernetes. It proves to you that you're talking to the real API server, not some fake one trying to trick you.

### How It Works in Kubernetes

When you use a tool like kubectl to talk to your cluster, it's like you're showing your ID badge. The API server checks if your "badge" (certificate) is valid and decides what you're allowed to do. At the same time, the API server shows its own certificate, so you know you're talking to the real deal.

### Creating and Managing Certificates

Let's look at how we can create and manage these certificates in Kubernetes, including options for both expiring and non-expiring certificates.

#### Imperative Approach

1. Generate a private key for a new user:
    
    ```bash
    openssl genrsa -out john.key 2048
    ```
    
2. Create a Certificate Signing Request (CSR):
    
    ```bash
    openssl req -new -key john.key -out john.csr -subj "/CN=john/O=developers"
    ```
    
3. Create a CertificateSigningRequest object in Kubernetes:
    
    For a certificate with expiration:
    
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: john-developer
    spec:
      request: $(cat john.csr | base64 | tr -d '\n')
      signerName: kubernetes.io/kube-apiserver-client
      expirationSeconds: 86400  # one day
      usages:
      - client auth
    EOF
    ```
    
    For a certificate without expiration (long-lived):
    
    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: certificates.k8s.io/v1
    kind: CertificateSigningRequest
    metadata:
      name: john-developer-long-lived
    spec:
      request: $(cat john.csr | base64 | tr -d '\n')
      signerName: kubernetes.io/kube-apiserver-client
      usages:
      - client auth
    EOF
    ```
    
    Note: Omitting the `expirationSeconds` field will result in a certificate with the default validity period (typically 1 year, but this can vary depending on your cluster configuration).
    
4. Approve the CSR:
    
    ```bash
    kubectl certificate approve john-developer
    # or for the long-lived certificate
    kubectl certificate approve john-developer-long-lived
    ```
    
5. Retrieve the signed certificate:
    
    ```bash
    kubectl get csr john-developer -o jsonpath='{.status.certificate}' | base64 -d > john.crt
    # or for the long-lived certificate
    kubectl get csr john-developer-long-lived -o jsonpath='{.status.certificate}' | base64 -d > john-long-lived.crt
    ```
    

#### Declarative Approach

You can also use YAML files to manage certificates. Here's an example of a CSR in YAML format for both expiring and non-expiring certificates:

For a certificate with expiration:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  request: <base64-encoded CSR>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
```

For a certificate without expiration (long-lived):

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer-long-lived
spec:
  request: <base64-encoded CSR>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
```

Save these as `john-csr.yaml` and `john-csr-long-lived.yaml` respectively, and apply them with:

```bash
kubectl apply -f john-csr.yaml
# or for the long-lived certificate
kubectl apply -f john-csr-long-lived.yaml
```

### Certificate Expiration Considerations

When deciding between certificates with expiration and those without, consider the following:

1. **Security**: Certificates with expiration provide better security as they limit the window of opportunity for a compromised certificate to be misused. Regular rotation of certificates is a security best practice.
    
2. **Maintenance**: Non-expiring certificates require less frequent maintenance but may pose a security risk if compromised and not revoked promptly.
    
3. **Automation**: For expiring certificates, consider setting up automated processes for renewal to prevent unexpected access issues due to expired certificates.
    
4. **Compliance**: Some regulatory requirements may mandate the use of certificates with specific expiration periods. Ensure your certificate management aligns with any applicable compliance standards.
    
5. **Use case**: Long-lived certificates might be suitable for certain system components or long-running processes, while short-lived certificates are often better for user authentication and temporary access grants.
    

Remember, even if you choose to use long-lived certificates, it's a good practice to periodically review and rotate them as part of your security maintenance routines.

## 10:00 AM: Resource Allocation - The Scheduler's Role

Next, you meet Bob from HR. In our Kubernetes world, Bob is like the Scheduler. The Scheduler's job is to decide where to place containers in the cluster.

Bob: "Hi there! I need to check something in the employee database. Mind if I use your computer?"

In Kubernetes, when the Scheduler needs to talk to the API server, it also needs to prove who it is. It uses its own special certificate (like Bob's HR badge) to do this.

### How It Works in Kubernetes

The Scheduler has its own certificate that it uses to authenticate with the API server. When it needs to make decisions about where to place pods, it presents this certificate to prove it has the authority to do so.

### Scheduler Certificate Management

The Scheduler's certificate is typically managed by the cluster itself. However, you can rotate these certificates using kubeadm:

```bash
sudo kubeadm alpha certs renew scheduler
```

## 11:00 AM: Securing the Vault - etcd's Role in Kubernetes

Charlie guards the company's most important documents. In Kubernetes, Charlie represents etcd, which stores all the important cluster data.

Charlie: "Hello! I need to update some files. Can you confirm you're authorized to request this?"

In Kubernetes, etcd needs to prove its identity to others (like a server certificate) and also check who's talking to it (like checking client certificates).

### How It Works in Kubernetes

When components need to read or write data to etcd, they present their certificates. Etcd checks these certificates to make sure only authorized components can access the sensitive cluster data.

### Managing etcd Certificates

For etcd, you'll typically use the certificates generated during cluster setup. However, you can rotate these certificates as well:

```bash
sudo kubeadm alpha certs renew etcd-server
sudo kubeadm alpha certs renew etcd-peer
sudo kubeadm alpha certs renew etcd-healthcheck-client
```

## 2:00 PM: Networking Magic - kube-proxy's Role

Dave manages how everyone in the office talks to each other. In Kubernetes, Dave is like kube-proxy, which manages network rules on nodes.

Dave: "Hey! I need to set up a new connection between departments. Got your network access card?"

In Kubernetes, kube-proxy needs to prove it's allowed to modify network rules. It does this with its own special certificate.

### How It Works in Kubernetes

Kube-proxy uses its certificate to authenticate with the API server when it needs to update network rules. This ensures that only the real kube-proxy can make changes to how pods communicate.

### Kube-proxy Certificate Management

Like the scheduler, kube-proxy certificates are managed by the cluster. You can rotate them using:

```bash
sudo kubeadm alpha certs renew admin.conf
```

## 4:00 PM: The Node Guardian - kubelet's Dual Role

Finally, you meet the CEO, who in our Kubernetes world represents the kubelet. The kubelet is super important - it's responsible for making sure containers are running on each node.

CEO: "Great to meet you! I'll need to check on all departments and also respond to requests from them."

The kubelet is special because it needs to both prove who it is (like a client certificate) and also accept incoming secure connections (like a server certificate).

### How It Works in Kubernetes

The kubelet uses its client certificate to authenticate with the API server when reporting node status or pod status. It also uses a server certificate to accept secure connections from the API server, like when the API server needs to initiate port forwarding to a pod.

### Kubelet Certificate Management

You can manually generate kubelet client and server certificates, but it's usually easier to let kubeadm handle this. To rotate kubelet client certificates:

```bash
sudo kubeadm alpha certs renew kubelet.conf
```

For the kubelet server certificate, you can enable automatic rotation by setting the `--rotate-server-certificates` flag in the kubelet configuration.

## 5:00 PM: The Trust Anchor - Certificate Authority's Role

At the end of the day, you meet the company founder - the person who started it all. In Kubernetes, this is like the Certificate Authority (CA).

Founder: "Welcome to the company! Here's a special stamp that makes your badge official."

The CA is super important because it signs all other certificates, making them trustworthy.

### How It Works in Kubernetes

The CA certificate is used to sign all other certificates in the cluster. When components verify each other's certificates, they're essentially checking if the certificate was signed by this trusted CA.

### Managing the CA

In most cases, you won't need to directly manage the CA certificate. However, if you need to create a new CA or rotate the existing one, you can use tools like cfssl or openssl. Here's a basic example using openssl:

1. Generate a new CA private key:
    
    ```bash
    openssl genrsa -out ca.key 4096
    ```
    
2. Create a new CA certificate:
    
    ```bash
    openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt
    ```
    

Remember, rotating the CA certificate is a major operation that requires updating all certificates in the cluster!

## Wrapping Up: How Everyone Checks Everyone Else

So, how does all this certificate checking work in practice? Here's a simple breakdown:

1. When any two components in Kubernetes talk to each other, they exchange certificates.
    
2. Each component checks if the other's certificate is signed by the trusted CA.
    
3. If the certificate is valid, they then check what permissions that certificate grants.
    
4. Based on these permissions, they decide whether to allow the action or not.
    

It's like everyone in our KubeCorp office checking each other's badges before sharing information or allowing actions. This way, even if someone sneaks into the office, they can't do anything without a proper badge!

## Conclusion

Whew! We've taken quite a journey through KubeCorp, haven't we? From the front desk to the CEO's office, we've seen how Kubernetes uses certificates to keep everything secure and running smoothly.

Remember, Kubernetes security might seem complicated, but it's really just about making sure everyone is who they say they are, and only allowed to do what they're supposed to do. Just like in a real office!

As you continue learning about Kubernetes, keep this office analogy in mind. It might make those technical concepts a bit easier to grasp. And remember, every expert was once a beginner, so don't get discouraged if it takes some time to sink in.

Keep exploring, keep asking questions, and most importantly, have fun with it! Kubernetes is a powerful tool, and understanding its security is a big step towards becoming a true Kubernetes wizard.

Until next time, happy clustering! 🚀