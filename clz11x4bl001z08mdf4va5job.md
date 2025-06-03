---
title: "How to Use Kubernetes Storage: Volumes and Persistent Volume Claims"
datePublished: Thu Jul 25 2024 09:08:38 GMT+0000 (Coordinated Universal Time)
cuid: clz11x4bl001z08mdf4va5job
slug: how-to-use-kubernetes-storage-volumes-and-persistent-volume-claims
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721898261015/e3636b34-11cf-4539-a0f5-1cbd33fdf8ae.png
tags: kubernetes-persistent-volumes, kubernetes-storage, storage-container, persistentvolumes, how-to-use-kubernetes-storage, volumes-and-persistent-volume-claims, persistentvolumeclaims, provision-a-new-pv

---

Hello, fellow Kubernetes enthusiasts! 👋

Imagine you're moving into a new apartment. You've got all your stuff packed in boxes, but when you arrive, you realize there's no furniture! That's kind of like running containers in Kubernetes without proper storage management. Your applications (the stuff in boxes) need a place to store their data (the furniture).

In this article, we're going to unpack the world of Kubernetes storage. We'll explore Volumes, Persistent Volumes, Persistent Volume Claims, and how they all fit together in the grand scheme of things. So, grab your favorite caffeinated beverage, and let's start this moving day!

## The Storage Dilemma in Kubernetes

In our new Kubernetes apartment complex, we need to solve the storage problem with a few key concepts:

1. Volumes (temporary storage)
    
2. Persistent Volumes (PVs) (long-term storage options)
    
3. Persistent Volume Claims (PVCs) (requests for storage)
    

Let's unpack these boxes one by one, shall we?

## Volumes: The Temporary Storage Solution

Volumes in Kubernetes are like those foldable tables you might use when you first move in. They're not permanent, but they get the job done for a while.

### How Volumes Work

When you create a Pod (think of it as a tiny apartment for your containers), you can specify one or more volumes to be used by the containers in that Pod. These volumes are tied to the Pod's lifecycle, meaning they exist as long as the Pod exists.

Here's a simple example of a Pod with a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: my-volume
      mountPath: /data
  volumes:
  - name: my-volume
    emptyDir: {}
```

In this example, we're using an `emptyDir` volume. It's like a temporary drawer that's created when the Pod moves in and cleared out when the Pod moves out. Perfect for those things you need to store temporarily but don't want to keep forever.

### Types of Volumes

Just like how you might have different storage solutions in your apartment, Kubernetes supports many types of volumes. Here are a few common ones:

* `emptyDir`: Our temporary drawer from the example above.
    
* `hostPath`: It's like storing stuff in the building's basement. It mounts a file or directory from the host node's filesystem.
    
* `configMap` and `secret`: Think of these as secure lockboxes for storing configuration data and secrets.
    
* Cloud provider-specific volumes: These are like renting extra storage units from AWS, GCP, or Azure.
    

Let's look at a `hostPath` volume example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      path: /data
      type: Directory
```

This Pod uses a `hostPath` volume, mounting the `/data` directory from the host node to `/test-pd` in the container. It's like having a storage closet in your apartment building that you can access.

### When to Use Volumes

You might want to use regular volumes when:

* You need a scratch pad for temporary calculations (faster than network storage)
    
* Your containers in the same Pod need to share data (like roommates sharing a whiteboard)
    
* You need to access the apartment building's storage room (node-level filesystems)
    

Now that we've set up our temporary storage, let's move on to more permanent solutions.

## Persistent Volumes: The Furniture Store of Kubernetes

If volumes are like foldable tables, Persistent Volumes (PVs) are like going to IKEA. They represent a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.

### How Persistent Volumes Work

PVs are cluster resources. They exist independently of any Pod and have their own lifecycle. It's like having a storage unit that stays even if you move out of your apartment. Here's an example of a PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  nfs:
    server: nfs-server.default.svc.cluster.local
    path: "/path/to/data"
```

This PV is backed by an NFS server. It's like renting a 5GB storage unit that can be accessed by one tenant at a time. Let's break down some key aspects:

* `capacity`: How much storage space you're renting (5GB in this case)
    
* `accessModes`: Rules for using the storage (we'll cover these next)
    
* `persistentVolumeReclaimPolicy`: What happens when you're done with the storage
    
* `storageClassName`: A way to categorize different types of storage
    

### Access Modes

PVs support different access modes. Think of these as the rules for using your storage unit:

* `ReadWriteOnce` (RWO): Only one tenant can use the storage unit at a time, but they can both read and write. It's like a private storage room.
    
* `ReadOnlyMany` (ROX): Multiple tenants can read from the storage unit, but nobody can write to it. Think of it as a public library.
    
* `ReadWriteMany` (RWX): Multiple tenants can read and write to the storage unit at the same time. This is like a shared workspace.
    

Choose the access mode based on your application's needs and the capabilities of your storage system. Not all storage types support all access modes, so check your storage provider's documentation.

### Reclaim Policies

When a tenant moves out, what happens to their storage unit? That's where reclaim policies come in:

* `Retain`: The storage unit is kept as-is, waiting for manual cleanup. It's like keeping your stuff in storage even after you've moved out.
    
* `Delete`: The storage unit is automatically cleared out and made available for the next tenant. This is like a full clean-out service when you move.
    
* `Recycle`: The contents are scrubbed before the unit is made available again (this policy is being phased out, so I wouldn't recommend using it).
    

Now that we've set up our storage units, let's see how we can request to use them.

## Persistent Volume Claims: Your Storage Shopping List

If PVs are like items in IKEA, Persistent Volume Claims (PVCs) are like your shopping list. They represent a request for storage by a user.

### How PVCs Work

PVCs describe the type of storage a Pod needs. Kubernetes then tries to find a PV that matches these requirements. Here's an example PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
```

This PVC is essentially saying, "I need a storage unit that's at least 8GB and can be written to by one tenant at a time." It's like telling the storage facility, "I need a unit that's at least 8 feet by 10 feet and I'll be the only one using it."

### Using PVCs in Pods and Deployments

Once you have a PVC, you can use it in a Pod or Deployment. It's like telling your movers which storage unit to use:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app-image
        volumeMounts:
        - name: storage
          mountPath: /data
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: my-pvc
```

This Deployment uses the PVC we created earlier, mounting it at `/data` in the containers. It's like giving each of your three tenants (replicas) access to the same storage unit.

## Real-Life Scenarios

Now that we've unpacked the basics, let's look at some real-world scenarios where you might use these storage options. It's like seeing how different people set up their apartments!

1. **Database Storage**: Imagine you're running a PostgreSQL database in Kubernetes. You'd use a PV and PVC to ensure your data persists even if the database Pod gets rescheduled to a different node. Here's how you might set it up:
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: postgres-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: postgres
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: postgres
      template:
        metadata:
          labels:
            app: postgres
        spec:
          containers:
          - name: postgres
            image: postgres:13
            volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/postgresql/data
          volumes:
          - name: postgres-storage
            persistentVolumeClaim:
              claimName: postgres-pvc
    ```
    
    This setup is like having a secure room for your important documents (database files) that stays intact even if you need to move to a different apartment in the building.
    
2. **Shared File Storage**: Your web application needs to store user uploads that should be accessible by multiple frontend Pods. A PV with ReadWriteMany access mode would be perfect for this. Here's an example using NFS:
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: nfs-pv
    spec:
      capacity:
        storage: 50Gi
      accessModes:
        - ReadWriteMany
      nfs:
        server: nfs-server.default.svc.cluster.local
        path: "/uploads"
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nfs-pvc
    spec:
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 50Gi
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: web-app
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: web-app
      template:
        metadata:
          labels:
            app: web-app
        spec:
          containers:
          - name: web-app
            image: my-web-app:latest
            volumeMounts:
            - name: uploads
              mountPath: /app/uploads
          volumes:
          - name: uploads
            persistentVolumeClaim:
              claimName: nfs-pvc
    ```
    
    This setup is like having a shared closet in your apartment building where all residents can store and access their belongings.
    
3. **Log Aggregation**: Use emptyDir volumes for temporary log storage on nodes before they're shipped off to your logging system. Here's an example:
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: counter
    spec:
      containers:
      - name: count
        image: busybox
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        emptyDir: {}
    ```
    
    This is like having a notepad in your apartment where you jot down notes temporarily before transferring them to your main notebook.
    

Now that we've seen how these storage solutions work in real-life scenarios, let's explore some advanced features. It's like upgrading your living space with smart home technology!

## Advanced Topics: Dynamic Provisioning and CSI Drivers

### Dynamic Provisioning

Dynamic provisioning allows you to automatically create storage resources when they are needed. It's like having a just-in-time furniture delivery service. Here's how you can set it up:

1. First, you need to define a StorageClass:
    
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: fast
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-ssd
    ```
    
2. Then, in your PVC, you can reference this StorageClass:
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: fast-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      storageClassName: fast
      resources:
        requests:
          storage: 10Gi
    ```
    

When this PVC is created, Kubernetes will automatically provision a new PV that matches the requirements. It's like calling a furniture store, and they deliver and set up exactly what you need, right when you need it.

### Container Storage Interface (CSI) Drivers

CSI is a standard for exposing arbitrary block and file storage systems to containerized workloads in Kubernetes. It's like having a universal remote that works with any TV brand. Here's a basic example of how you might use a CSI driver:

1. First, you need to deploy the CSI driver. This varies depending on the storage provider, but here's a simplified example:
    
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: CSIDriver
    metadata:
      name: mycsidriver.example.com
    spec:
      attachRequired: true
      podInfoOnMount: true
    ```
    
2. Then, you can create a StorageClass that uses this CSI driver:
    
    ```yaml
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: csi-storageclass
    provisioner: mycsidriver.example.com
    parameters:
      fstype: ext4
    ```
    
3. Finally, you can use this StorageClass in your PVCs:
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: csi-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
      storageClassName: csi-storageclass
    ```
    

Using CSI drivers allows you to leverage a wide range of storage solutions in a standardized way, making your Kubernetes clusters more flexible and powerful. It's like being able to easily switch between different types of storage units without having to change how you pack your boxes.

## Tips and Tricks

Now that we've toured our Kubernetes storage apartment, here are some tips to make your stay more comfortable:

For the newcomers (just moved in):

* Start simple! Play around with emptyDir volumes before diving into the more complex stuff. It's like using cardboard boxes before investing in fancy storage solutions.
    
* Use labels and selectors to keep your PVs and PVCs organized. It's like labeling your moving boxes - future you will thank present you for this.
    
* Always set resource requests and limits for your storage. It's like setting a budget before you go furniture shopping - it keeps things under control.
    

## Tips and Tricks

Now that we've toured our Kubernetes storage apartment, here are some tips to make your stay more comfortable:

For the newcomers (just moved in):

* Start simple! Play around with emptyDir volumes before diving into the more complex stuff. It's like using cardboard boxes before investing in fancy storage solutions.
    
* Use labels and selectors to keep your PVs and PVCs organized. It's like labeling your moving boxes - future you will thank present you for this.
    
* Always set resource requests and limits for your storage. It's like setting a budget before you go furniture shopping - it keeps things under control.
    

For the Kubernetes veterans (long-time residents):

* Implement dynamic provisioning with StorageClasses to automate PV creation. It's like having a smart home system that anticipates your storage needs.
    
* Use volume snapshots for backup and restore operations. Think of it as taking photos of your perfectly organized closet before a big move.
    
* Explore StatefulSets for applications that require stable, unique network identifiers and persistent storage. It's perfect for databases, like giving each database its own dedicated room with a custom nameplate.
    
* Consider using local persistent volumes for high-performance workloads. It's like having a built-in safe in your apartment for your most valuable possessions.
    

## Best Practices

Let's wrap up with some best practices to keep your Kubernetes storage neat and tidy:

1. **Right-size your storage**: Don't over-provision. Start small and use storage that can be expanded later if needed. It's like buying modular furniture that you can add to over time.
    
2. **Use StorageClasses**: They allow you to define different "tiers" of storage. It's like having different types of storage units - economy, standard, and premium.
    
3. **Implement monitoring**: Keep an eye on your storage usage and performance. It's like having a smart meter for your utilities.
    
4. **Plan for disaster recovery**: Use tools like Velero to backup your PVs and PVCs. It's like having insurance for your belongings.
    
5. **Consider multi-zone deployments**: For critical data, replicate across multiple zones. It's like having a backup apartment in case of emergencies.
    
6. **Use Pod Disruption Budgets**: When using PVs with access mode ReadWriteOnce, ensure you have Pod Disruption Budgets to prevent data unavailability during node maintenance.
    
7. **Regularly audit your storage**: Clean up unused PVs and PVCs. It's like decluttering your apartment regularly to keep it tidy.
    

Here's an example of a Pod Disruption Budget:

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

This ensures that at least 2 pods of `my-app` are always available, even during node maintenance.

## Conclusion

Congratulations! You've now unpacked the essentials of Kubernetes storage. From temporary Volumes to long-term Persistent Volumes, and from manual provisioning to dynamic StorageClasses, you're well-equipped to set up and manage storage in your Kubernetes clusters.

Remember, just like organizing an apartment, managing Kubernetes storage is an ongoing process. As your applications grow and evolve, so will your storage needs. Don't be afraid to reorganize, optimize, and try new solutions.

Here's a quick checklist to keep in mind as you embark on your Kubernetes storage journey:

1. Understand your application's storage needs
    
2. Choose the right type of volume for each use case
    
3. Use PersistentVolumes and PersistentVolumeClaims for data that needs to persist
    
4. Implement StorageClasses for dynamic provisioning
    
5. Regularly monitor and maintain your storage resources
    
6. Plan for scalability and disaster recovery
    

And most importantly, always keep your data safe and backed up. After all, in the world of Kubernetes, your data is your most valuable possession!

Happy Kuberneting, and may your pods always find the perfect persistent volume to call home! 🏠🐳

For further reading, check out the official Kubernetes documentation on [Storage](https://kubernetes.io/docs/concepts/storage/) and [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/). These resources will provide even more depth on the topics we've covered here.

Remember, practice makes perfect. Try setting up different storage configurations in a test cluster to get hands-on experience. And don't hesitate to reach out to the Kubernetes community if you have questions - it's a friendly neighborhood always willing to help fellow residents!