---
title: "Kubernetes Ingress Essentials: Mastering NGINX Ingress Controller"
seoTitle: "Kubernetes Ingress Essentials: Mastering NGINX Ingress Controller"
datePublished: Thu Aug 08 2024 06:52:47 GMT+0000 (Coordinated Universal Time)
cuid: clzkx8bon000109ji20r5hqi0
slug: kubernetes-ingress-essentials-mastering-nginx-ingress-controller
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1723099927611/be51682a-9a7f-40e0-b922-ecc20ef8b69e.png

---

Hello, Kubernetes enthusiasts! 👋 Ready to take your cluster management skills to the next level? Today, we're diving deep into the world of Ingress and the NGINX Ingress Controller. Buckle up – we're about to start an exciting journey through the intricacies of Kubernetes traffic management!

Understanding the Need for Ingress

Before we dive into the technical details, let's understand why Ingress is such a crucial component in the Kubernetes ecosystem.

The Limitations of Services Alone

Imagine you're running a microservices-based application with multiple services:

* Product Catalog (/products)
    
* User Accounts (/accounts)
    
* Order Management (/orders)
    

Using only Kubernetes Services, you'd need to expose each one separately:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: product-catalog
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: product-catalog
---
apiVersion: v1
kind: Service
metadata:
  name: user-accounts
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: user-accounts
---
apiVersion: v1
kind: Service
metadata:
  name: order-management
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
  selector:
    app: order-management
```

This approach has several drawbacks:

1. Each service requires its own LoadBalancer, which can be costly in cloud environments.
    
2. Managing multiple endpoints becomes complex for clients.
    
3. Implementing SSL/TLS termination for each service separately is cumbersome.
    
4. Path-based or host-based routing isn't possible out of the box.
    

Enter Ingress: The Traffic Director Your Cluster Deserves

Ingress solves these problems by providing a unified entry point for your cluster. It's like having a smart receptionist for your services!

Here's a simple Ingress resource that routes traffic to our three services:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ecommerce-ingress
spec:
  rules:
  - host: myecommerce.com
    http:
      paths:
      - path: /products
        pathType: Prefix
        backend:
          service:
            name: product-catalog
            port: 
              number: 80
      - path: /accounts
        pathType: Prefix
        backend:
          service:
            name: user-accounts
            port: 
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: order-management
            port: 
              number: 80
```

But remember, Kubernetes doesn't come with a built-in Ingress controller. It's like having traffic rules without a traffic cop to enforce them. That's where NGINX Ingress Controller comes in!

NGINX Ingress Controller: Your Cluster's Traffic Cop

NGINX is like the seasoned traffic officer of the web server world. It's fast, efficient, and comes with a ton of features. Let's see how to deploy it in our cluster!

Prerequisites: Setting Up the Stage

Before we deploy NGINX Ingress Controller, we need to set up some permissions. It's like giving our traffic cop the authority to do their job.

Create a namespace:

```bash
kubectl create namespace ingress-nginx
```

Create a ServiceAccount:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-nginx
```

Create ClusterRole and ClusterRoleBinding:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
      - "networking.k8s.io"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-nginx
```

Deploying NGINX Ingress Controller

Now that we've set the stage, let's bring in our star performer – the NGINX Ingress Controller!

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: k8s.gcr.io/ingress-nginx/controller:v1.2.0
          args:
            - /nginx-ingress-controller
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --election-id=ingress-controller-leader
            - --ingress-class=nginx
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

Deploying NGINX Ingress Controller through GitHub

To deploy the official NGINX Ingress Controller, we can use the resources provided in the official GitHub repository. This method ensures we're using the latest stable version and following best practices.

1. First, let's create a namespace for our Ingress resources:
    

```bash
kubectl create namespace ingress-nginx
```

2. Now, we'll apply the mandatory resources:
    

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

This command creates all necessary resources including the Deployment, ServiceAccount, ClusterRole, and ClusterRoleBinding.

3. Verify the installation:
    

```bash
kubectl get pods -n ingress-nginx
```

You should see the Ingress controller pod running.

Creating a ConfigMap for NGINX Ingress Controller

Now, let's create a separate ConfigMap for our NGINX Ingress Controller. Here's an example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
data:
  use-forwarded-headers: "true"
  compute-full-forwarded-for: "true"
  client-header-buffer-size: "16k"
  large-client-header-buffers: "4 16k"
  http2-max-field-size: "16k"
  http2-max-header-size: "128k"
  enable-brotli: "true"
  enable-access-log: "true"
  access-log-path: "/var/log/nginx/access.log"
  error-log-path: "/var/log/nginx/error.log"
```

Apply this ConfigMap:

```bash
kubectl apply -f nginx-configmap.yaml
```

Benefits of Creating a Separate ConfigMap for Ingress Controller:

1. Centralized configuration: All NGINX-specific settings are in one place.
    
2. Easy updates: You can modify the ConfigMap without touching the controller deployment.
    
3. Version control: You can track changes to your Ingress configuration in your version control system.
    
4. Flexibility: Different teams can manage their own ConfigMaps for different environments or use cases.
    
5. Separation of concerns: It keeps the Ingress controller deployment yaml clean and focused on pod specifications.
    

Breaking Down Technical Terms for Beginners

Let's break down some of the technical terms we've used:

* Ingress: Think of it as a smart router for your cluster. It decides where to send incoming web traffic.
    
* ConfigMap: A Kubernetes resource for storing configuration data. It's like a config file, but more flexible and easier to manage.
    
* Namespace: A way to divide cluster resources between multiple users or projects. It's like having separate folders for different projects on your computer.
    
* Deployment: A Kubernetes resource that manages a set of identical pods. It ensures that a specified number of pod replicas are running at any given time.
    
* ServiceAccount: An account used by pod processes to interact with the Kubernetes API server. It's like a user account, but for pods instead of humans.
    
* ClusterRole and ClusterRoleBinding: These define permissions in your cluster. Think of them as setting up who's allowed to do what in your Kubernetes environment.
    

Ingress in Action: Cloud vs On-Premises

Now, let's talk about how Ingress works in different environments. The beauty of Kubernetes is its flexibility, but this means we need to approach Ingress slightly differently depending on our infrastructure.

Ingress in Cloud Environments

In cloud environments like AWS, GCP, or Azure, the Ingress controller typically creates a cloud-specific load balancer automatically. Here's how it works:

1. You deploy the NGINX Ingress Controller.
    
2. The controller creates a Kubernetes Service of type LoadBalancer.
    
3. The cloud provider automatically provisions a load balancer (e.g., ELB in AWS, Cloud Load Balancer in GCP).
    
4. The load balancer directs traffic to the Ingress controller, which then routes it based on the Ingress rules.
    

Here's an example of how to expose the Ingress controller in a cloud environment:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-ssl-cert: "arn:aws:acm:region:account-id:certificate/cert-id"
    service.beta.kubernetes.io/aws-load-balancer-backend-protocol: "http"
    service.beta.kubernetes.io/aws-load-balancer-ssl-ports: "https"
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 80
      protocol: TCP
      name: https
  selector:
    app: nginx-ingress
```

In this example, we're using annotations specific to AWS to configure SSL termination at the load balancer level.

Ingress in On-Premises Environments

In on-premises environments, things work a bit differently. Since there's no cloud provider to automatically provision a load balancer, you need to handle external access yourself. Here are a few common approaches:

1. NodePort Service: Expose the Ingress controller using a NodePort service, then configure your own load balancer (e.g., HAProxy, F5) to direct traffic to the NodePort on your Kubernetes nodes.
    

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
    - port: 443
      targetPort: 443
      protocol: TCP
      name: https
  selector:
    app: nginx-ingress
```

2. MetalLB: Use MetalLB to simulate a LoadBalancer service in your on-premises environment. This allows you to use the same configuration as in cloud environments.
    
3. Ingress Controller as DaemonSet: Deploy the Ingress controller as a DaemonSet and use the host network. This ensures an instance of the controller runs on every node, and you can configure your external load balancer to direct traffic to any node in the cluster.
    

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
spec:
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      hostNetwork: true
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: k8s.gcr.io/ingress-nginx/controller:v1.2.0
          args:
            - /nginx-ingress-controller
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --election-id=ingress-controller-leader
            - --ingress-class=nginx
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
```

Advanced Routing Strategies

Now that we understand how to deploy Ingress in different environments, let's explore some advanced routing strategies.

Host-Based Routing

Host-based routing allows you to direct traffic to different services based on the hostname. This is particularly useful when you're hosting multiple domains on the same cluster.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-routing
spec:
  rules:
  - host: app1.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port: 
              number: 80
  - host: app2.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port: 
              number: 80
```

In this example, traffic to [app1.mycompany.com](http://app1.mycompany.com) will be routed to the app1-service, while traffic to [app2.mycompany.com](http://app2.mycompany.com) will go to app2-service.

Path-Based Routing

We've already seen an example of path-based routing earlier, but let's dive a bit deeper. Path-based routing allows you to direct traffic to different services based on the URL path.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-routing
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port: 
              number: 80
      - path: /admin
        pathType: Prefix
        backend:
          service:
            name: admin-service
            port: 
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port: 
              number: 80
```

In this setup:

* Requests to [myapp.com/api/\*](http://myapp.com/api/*) will go to the api-service
    
* Requests to [myapp.com/admin/\*](http://myapp.com/admin/*) will go to the admin-service
    
* All other requests will be handled by the web-service
    

Organizing Ingress Resources Across Namespaces

In larger organizations or more complex setups, you might want to deploy Ingress resources in different namespaces. This allows for better organization and separation of concerns, especially when different teams are responsible for different services.

Multi-Namespace Ingress

Here's how you can set up Ingress resources in different namespaces:

Create namespaces for different teams or applications:

```bash
kubectl create namespace team-a
kubectl create namespace team-b
```

Deploy services in their respective namespaces:

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: service-a
  namespace: team-a
spec:
  ports:
    - port: 80
  selector:
    app: service-a
---
apiVersion: v1
kind: Service
metadata:
  name: service-b
  namespace: team-b
spec:
  ports:
    - port: 80
  selector:
    app: service-b
```

Create Ingress resources in each namespace:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-a
  namespace: team-a
spec:
  rules:
  - host: service-a.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-a
            port: 
              number: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-b
  namespace: team-b
spec:
  rules:
  - host: service-b.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: service-b
            port: 
              number: 80
```

With this setup, each team can manage their own Ingress resources without interfering with others. The NGINX Ingress Controller will automatically pick up Ingress resources from all namespaces.

Advanced Features of NGINX Ingress Controller

Now that we've covered the basics and some advanced routing strategies, let's explore some powerful features of the NGINX Ingress Controller.

SSL/TLS Termination

Securing your traffic is crucial. Here's how you can enable SSL/TLS termination:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port: 
              number: 80
```

Don't forget to create a Secret with your SSL certificate:

```bash
kubectl create secret tls myapp-tls --key path/to/tls.key --cert path/to/tls.crt
```

URL Rewriting

NGINX Ingress Controller allows you to rewrite the URL before passing it to your backend service. This is particularly useful when your backend service expects a different URL structure than what's exposed externally.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite-example-ingress
  namespace: default
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api/v2/(.*)
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port: 
              number: 80
```

In this example, a request to /api/v2/products will be rewritten to /products before being sent to the backend service.

Rate Limiting

To protect your services from abuse or overload, you can implement rate limiting:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rate-limit-example
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "10"
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port: 
              number: 80
```

This configuration limits requests to 10 per second per client IP address.

Session Affinity

If your application requires session stickiness, you can enable it like this:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: session-affinity-example
  annotations:
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
    nginx.ingress.kubernetes.io/session-cookie-expires: "172800"
    nginx.ingress.kubernetes.io/session-cookie-max-age: "172800"
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port: 
              number: 80
```

This configuration ensures that subsequent requests from the same client are routed to the same backend pod.

Monitoring and Troubleshooting

No matter how well you set up your Ingress, issues can still arise. Let's look at some ways to monitor and troubleshoot your NGINX Ingress Controller.

Enabling Access Logs

Access logs are crucial for understanding traffic patterns and diagnosing issues. Enable them like this:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-configuration
  namespace: ingress-nginx
data:
  enable-access-log: "true"
  access-log-path: "/var/log/nginx/access.log"
```

Prometheus Metrics

NGINX Ingress Controller exposes Prometheus metrics by default. You can scrape these metrics to monitor the health and performance of your Ingress:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-ingress-controller-metrics
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: nginx-ingress
  namespaceSelector:
    matchNames:
      - ingress-nginx
  endpoints:
  - port: metrics
```

Common Troubleshooting Commands

Check if the Ingress Controller is running:

```bash
kubectl get pods -n ingress-nginx
```

View Ingress Controller logs:

```bash
kubectl logs -n ingress-nginx deployment/nginx-ingress-controller
```

Verify your Ingress resource:

```bash
kubectl describe ingress my-ingress
```

Check if the backend services are running:

```bash
kubectl get svc,endpoints
```

Test connectivity to your services:

```bash
kubectl run -it --rm debug --image=busybox --restart=Never -- wget -O- http://myapp-service.default.svc.cluster.local
```

Real-Life Example and Use Cases

Imagine you're running an e-commerce platform. You have different services for products, user accounts, and orders. With Ingress, you can:

1. Route all product-related requests (like /products/\*) to your product service.
    
2. Send user account requests (/accounts/\*) to the user account service.
    
3. Direct order-related traffic (/orders/\*) to the order management service.
    

All of this happens through a single entry point, making it easier to manage and secure your application.

Tips and Tricks

1. Use annotations wisely: NGINX Ingress Controller offers a wide range of annotations. Use them to fine-tune your Ingress behavior without cluttering your Ingress resources.
    
2. Implement proper security measures: Always use HTTPS, implement network policies, and consider using OAuth2 or JWT authentication for sensitive applications.
    
3. Set resource limits: Ensure your Ingress Controller has enough resources, but also set limits to prevent it from consuming all cluster resources.
    
4. Keep your Ingress Controller updated: Regularly update to the latest stable version to benefit from bug fixes and new features.
    
5. Use canary deployments: Implement canary releases using Ingress to gradually roll out changes to your services.
    
6. Monitor and alert: Set up proper monitoring and alerting for your Ingress Controller to quickly catch and respond to issues.
    
7. Use namespaces: Organize your Ingress resources and related services into namespaces for better management.
    

Conclusion

Whew! We've covered a lot of ground in this comprehensive guide to NGINX Ingress Controller. From understanding the basics of Ingress to deploying through GitHub, creating ConfigMaps, exploring use cases, and diving into powerful features – you're now equipped to handle Kubernetes traffic like a pro!

Remember, mastering Ingress is a journey. Don't be afraid to experiment, and always refer to the official documentation for the most up-to-date information.

Got any cool Ingress tricks up your sleeve? Or maybe you're puzzling over a particularly tricky routing scenario? Drop a comment below – let's learn from each other and keep the conversation going!

Until next time, happy clustering! 🚀