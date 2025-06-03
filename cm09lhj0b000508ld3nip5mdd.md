---
title: "Kubernetes Probes: A Deep Dive into Application Health Checks"
seoTitle: "Understanding Kubernetes Probes for App Health"
seoDescription: "A deep dive into Kubernetes probes for better application health checks, ensuring reliability and restful nights for developers"
datePublished: Sun Aug 25 2024 13:18:15 GMT+0000 (Coordinated Universal Time)
cuid: cm09lhj0b000508ld3nip5mdd
slug: kubernetes-probes-a-deep-dive-into-application-health-checks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1724591783218/2f303418-44c9-4aca-976c-768d4ee924ce.png
tags: kubernetes-readiness-probe, kubernetes-probes, liveness-probes

---

It was 2 AM, and my phone was buzzing incessantly. Bleary-eyed, I fumbled for it, already knowing what I'd see. Yep, production was down. Again.

As I dragged myself to my laptop, a familiar sense of dread washed over me. Why hadn't our monitoring caught this? Why did our users have to be the ones to alert us? There had to be a better way.

Little did I know, that night would be the start of my journey into the world of Kubernetes probes – a journey that would transform how I approach application health and reliability.

If you've ever found yourself in a similar late-night debugging session, or if you're just starting out and want to avoid this scenario altogether, stick with me. I'm about to share everything I've learned about Kubernetes probes, from the basics to some advanced tricks that have saved my bacon more times than I can count.

## The Probe Trio: Your Application's Health Dream Team

Kubernetes offers three types of probes, each with its own special role in keeping your applications healthy and user-friendly. Let's break them down:

```bash
graph TD
    A[Kubernetes Pod] --> B[Liveness Probe]
    A --> C[Readiness Probe]
    A --> D[Startup Probe]
    B --> E[Is the container alive?]
    C --> F[Is the container ready for traffic?]
    D --> G[Is the container still starting up?]
    E --> |Yes| H[Continue running]
    E --> |No| I[Restart container]
    F --> |Yes| J[Allow traffic]
    F --> |No| K[Stop traffic]
    G --> |Yes| L[Wait and check again]
    G --> |No| M[Allow liveness and readiness probes]
```

### 1\. Liveness Probe: The Heartbeat Monitor

Remember that night I mentioned? A properly configured liveness probe could have prevented it. This probe is like a doctor constantly checking your application's pulse. If the app flatlines, Kubernetes steps in as the defibrillator, restarting the container.

Here's a basic example of a liveness probe in action:

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
  failureThreshold: 3
```

But here's where it gets interesting. In one of our microservices, we had an issue where the app would occasionally hang but still respond to basic HTTP requests. Our solution? We implemented a custom liveness probe that checked not just if the service was responding, but if it was actually processing requests:

```yaml
livenessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - 'curl -s http://localhost:8080/process-check | grep "OK"'
  initialDelaySeconds: 10
  periodSeconds: 5
```

This probe actually executes a command in the container, making a curl request to a special endpoint that checks if the service is processing requests correctly. If it doesn't get an "OK" response, Kubernetes knows it's time for a restart.

### 2\. Readiness Probe: The Traffic Controller

While the liveness probe is all about keeping your app alive, the readiness probe is about keeping it functional for users. It's like a bouncer at a club, deciding whether your app is ready to handle requests or needs a bit more time to get its act together.

A basic readiness probe might look like this:

```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

But let's take it up a notch. In our payment processing service, we needed to ensure that not only was the app up, but it had a valid connection to our payment gateway. Here's how we handled that:

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
  failureThreshold: 3
```

Our `/ready` endpoint doesn't just return a 200 OK. It actually attempts to make a test connection to our payment gateway. If that fails, the probe fails, and Kubernetes stops sending traffic to that pod until it's back in a good state.

### 3\. Startup Probe: The Patience Teacher

The startup probe is the newest member of the probe family, and boy, has it been a game-changer for some of our more complex applications. It's like a patient parent, giving your app the time it needs to get ready before the other probes start their work.

Here's a basic startup probe configuration:

```yaml
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

We used this to great effect in our data processing application that needed to load a large amount of data into memory on startup. Before the startup probe, our liveness probe would sometimes kill the container before it had finished initializing. Here's how we solved it:

```yaml
startupProbe:
  httpGet:
    path: /startup-complete
    port: 8080
  failureThreshold: 60
  periodSeconds: 10
```

Our `/startup-complete` endpoint checks if all the necessary data has been loaded. This configuration gives the app up to 10 minutes (60 \* 10 seconds) to start up before Kubernetes begins to worry.

## Advanced Probe Techniques: Lessons from the Trenches

After years of working with these probes, I've picked up a few tricks that have made our systems more robust:

1. **Use context timeouts**: When your probe makes HTTP requests or DB queries, always use a context with a timeout. I once spent hours debugging an issue where a hanging DB query was causing our readiness probe to never complete, effectively taking down our entire service.
    
    Here's an example of how to implement this in Go:
    
    ```go
    func readinessHandler(w http.ResponseWriter, r *http.Request) {
        ctx, cancel := context.WithTimeout(r.Context(), 5*time.Second)
        defer cancel()
    
        err := checkDatabaseConnection(ctx)
        if err != nil {
            http.Error(w, "Database not ready", http.StatusServiceUnavailable)
            return
        }
    
        w.WriteHeader(http.StatusOK)
    }
    
    func checkDatabaseConnection(ctx context.Context) error {
        return db.PingContext(ctx)
    }
    ```
    
2. **Probe your dependencies**: Don't just check if your app is up. Check if it can reach its critical dependencies. We include checks for database connections, cache services, and even external APIs in our readiness probes.
    
    Here's a more comprehensive readiness probe implementation:
    
    ```go
    func readinessHandler(w http.ResponseWriter, r *http.Request) {
        checks := []struct {
            name string
            check func(context.Context) error
        }{
            {"database", checkDatabaseConnection},
            {"cache", checkCacheConnection},
            {"payment_gateway", checkPaymentGateway},
        }
    
        ctx, cancel := context.WithTimeout(r.Context(), 10*time.Second)
        defer cancel()
    
        for _, c := range checks {
            if err := c.check(ctx); err != nil {
                http.Error(w, fmt.Sprintf("%s not ready: %v", c.name, err), http.StatusServiceUnavailable)
                return
            }
        }
    
        w.WriteHeader(http.StatusOK)
    }
    ```
    
3. **Implement probe endpoints strategically**: Don't just return `200 OK` from your `/healthz` endpoint. Use it as an opportunity to do a self-check of your application's health. But be careful not to make it too heavy – you don't want your health check to be the thing that brings down your app!
    
4. **Use the** `terminationGracePeriodSeconds`: This Kubernetes setting works hand-in-hand with your probes. It determines how long Kubernetes will wait for your app to shut down gracefully before forcefully killing it. Align this with your probe settings to ensure smooth rollouts and rollbacks.
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app
    spec:
      template:
        spec:
          terminationGracePeriodSeconds: 60
          containers:
          - name: my-app
            image: my-app:v1
            ports:
            - containerPort: 8080
            livenessProbe:
              httpGet:
                path: /healthz
                port: 8080
              failureThreshold: 3
              periodSeconds: 10
            readinessProbe:
              httpGet:
                path: /ready
                port: 8080
              failureThreshold: 3
              periodSeconds: 10
    ```
    
5. **Monitor your probe metrics**: Kubernetes exposes metrics about probe successes and failures. Set up alerts on these. A sudden increase in probe failures can be an early warning sign of trouble brewing.
    
    You can use Prometheus to scrape these metrics and set up alerts. Here's an example Prometheus alert rule:
    
    ```yaml
    groups:
    - name: probe_alerts
      rules:
      - alert: HighProbeFailureRate
        expr: rate(kubelet_probe_total{type="readiness",result="failure"}[5m]) > 0.1
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: High probe failure rate
          description: More than 10% of readiness probes are failing for {{ $labels.pod }}
    ```
    

## The Human Side of Probes

Implementing effective probes isn't just a technical challenge – it's a cultural one. It requires a shift in how we think about application health and reliability.

I remember the resistance I faced when I first proposed implementing comprehensive probes across our microservices. "It's just extra overhead," some said. "Our monitoring tools are enough," others argued.

But after that one incident where our probes caught and mitigated an issue before it became user-facing, minds started to change. Now, defining probe strategies is a key part of our application design process.

## Wrapping Up: The Probe Journey Continues

As I write this, it's been years since that 2 AM wake-up call that started my probe journey. Our systems are more reliable, our nights are more restful, and our users are happier.

But the journey isn't over. With each new application, each new Kubernetes feature, we find new ways to leverage probes to make our systems more robust.

So, whether you're just starting out with Kubernetes or you're a seasoned pro, I encourage you to dive deep into probes. Experiment, push their limits, and share what you learn. Because at the end of the day, we're all in this together, working towards more reliable, more resilient systems.

And who knows? Maybe the next great probe innovation will come from you. Happy probing!

## Further Reading

For more information on Kubernetes probes, check out these official Kubernetes documentation pages:

1. [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
    
2. [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
    
3. [Container Probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
    

These resources will provide you with the most up-to-date and detailed information straight from the source.