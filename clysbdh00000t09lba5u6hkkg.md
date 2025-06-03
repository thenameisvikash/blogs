---
title: "Ultimate How-To for Kubernetes Backup and Restore Without Additional Tools"
datePublished: Fri Jul 19 2024 06:23:22 GMT+0000 (Coordinated Universal Time)
cuid: clysbdh00000t09lba5u6hkkg
slug: ultimate-how-to-for-kubernetes-backup-and-restore-without-additional-tools
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1721369421572/b3f6350d-bb72-44a6-92a0-db69155a02f0.png
tags: kubernetes-backup, kubernetes-backup-and-restore, how-to-for-kubernetes-backup, kubernetes-data, backing-up-etcd, restore-kubernetes

---

Hey there, Kubernetes fans! Remember me? The guy who was up at 2 AM, surrounded by empty coffee mugs, trying to fix a misbehaving production cluster? Well, I'm back, and this time we're diving deep into the exciting world of Kubernetes backup and restore strategies—without using any third-party tools. Buckle up, it's going to be a wild ride!

If you missed my previous article on cluster maintenance (where have you been?), you can catch up [here](https://devopswizard.hashnode.dev/how-to-easily-maintain-your-kubernetes-cluster-beginners-edition). Go ahead, I'll wait. *whistles tunelessly*

Back? Great! Now, let's talk about why you're here. You've set up your Kubernetes cluster, you're maintaining it like a boss (thanks to my previous article, of course), but there's still this nagging feeling in the back of your mind. It's that little voice that whispers, "What if everything goes wrong?" Well, my friend, that's where backups come in.

## Why Backups Are Your Cluster's Best Friend

Picture this: It's another late night at the office. You're just about to push that big update you've been working on for weeks. Your finger hovers over the enter key, and then... *boom*! The power goes out. When everything comes back online, your cluster is about as responsive as a teenager on a Saturday morning. Panic sets in. Your career flashes before your eyes. But wait! You remember that you've set up a robust backup system. Crisis averted! You're not just a DevOps engineer; you're a DevOps superhero.

That, my friends, is why we backup. It's not just good practice; it's your get-out-of-jail-free card when Murphy's Law decides to pay your cluster a visit.

## What to Backup: The Holy Trinity of Kubernetes Data

When it comes to Kubernetes backups, there are three key areas you need to focus on:

1. **etcd**: This is the brain of your operation. It's where Kubernetes stores all its cluster data. Lose this, and it's like your cluster got a lobotomy. Not pretty.
    
2. **Persistent Volumes**: This is where your stateful application data lives. You know, the important stuff that makes your applications actually useful.
    
3. **Kubernetes Objects**: All those YAML files you've crafted with the precision of a Swiss watchmaker? Yeah, you'll want to keep those safe too.
    

Now, let's break these down and see how we can backup each one without losing our sanity (or what's left of it).

## Backing Up etcd: Saving Your Cluster's Memory

Backing up etcd is like creating a save point in a video game. If things go south, you can always return to this point. But here's the kicker: depending on how your cluster is set up, you might need to approach this differently. Let's cover all our bases:

### Method 1: Using kubectl (for etcd running as a pod)

If etcd is running as a pod in your cluster (common in some managed Kubernetes services), you can use kubectl to interact with it:

1. First, let's find our etcd pod:
    
    ```bash
    kubectl get pods -n kube-system | grep etcd
    ```
    
    You should see something like `etcd-minikube` or `etcd-master-0`.
    
2. Now, let's create a snapshot using kubectl exec:
    
    ```bash
    kubectl exec -it -n kube-system etcd-minikube -- etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /var/lib/etcd/snapshot-$(date +%Y-%m-%d-%H-%M-%S).db
    ```
    
    Replace `etcd-minikube` with the name of your etcd pod.
    
3. Copy the snapshot to your local machine:
    
    ```bash
    kubectl cp -n kube-system etcd-minikube:/var/lib/etcd/snapshot-2023-07-19-14-30-00.db ./etcd-snapshot-2023-07-19-14-30-00.db
    ```
    
    Again, replace `etcd-minikube` with your pod name, and adjust the filename as necessary.
    

### Method 2: Using crictl (for etcd running as a container)

If etcd is running as a container managed by the kubelet (common in kubeadm-created clusters), you can use crictl:

1. First, find the etcd container:
    
    ```bash
    sudo crictl ps | grep etcd
    ```
    
    Note the container ID.
    
2. Now, let's find where etcd stores its data:
    
    ```bash
    sudo crictl inspect <container-id> | grep -A 2 VolumesFrom
    ```
    
    Replace `<container-id>` with the ID you found in step 1.
    
3. Create the snapshot:
    
    ```bash
    sudo crictl exec -it <container-id> etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /var/lib/etcd/snapshot-$(date +%Y-%m-%d-%H-%M-%S).db
    ```
    

### Method 3: Direct etcdctl (for etcd running directly on the host)

If etcd is running directly on the host (less common, but possible in some custom setups), you can use etcdctl directly:

1. Create the snapshot:
    
    ```bash
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /path/to/backup/etcd-snapshot-$(date +%Y-%m-%d-%H-%M-%S).db
    ```
    
    Pro tip: Replace `/path/to/backup` with an actual path, like `/home/user/etcd-backups`. I once spent an hour wondering why my backup wasn't working, only to realize I had literally used `/path/to/backup` in my command. Don't be like me.
    

Regardless of which method you use, always verify your snapshot:

```bash
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /path/to/your/etcd-snapshot-2023-07-19-14-30-00.db
```

This command will show you details about your snapshot, including the total key-value pairs and the size of the backup. If this command doesn't make you feel like a hacker in a 90s movie, I don't know what will.

Remember, the exact paths and pod/container names might vary depending on your specific setup. Always double-check these details in your own environment.

## Persistent Volumes: Because Data Loss is Not an Option

Backing up Persistent Volumes (PVs) is like packing for a trip – you want to make sure you have everything you need, just in case. With Kubernetes, we need to get a bit creative here. Remember, PVs are often backed by cloud provider storage or network-attached storage, so our backup strategy needs to account for that.

1. First, let's identify our PVs:
    
    ```bash
    kubectl get pv
    ```
    
    This will list all your PVs. Make a note of the ones you want to backup.
    
2. For each PV, we need to create a snapshot. The exact method depends on your storage provider. For example, if you're using AWS EBS:
    
    ```bash
    aws ec2 create-snapshot --volume-id <volume-id> --description "Backup for PV on $(date +%Y-%m-%d)"
    ```
    
    Replace `<volume-id>` with the actual volume ID of your PV.
    
3. If you're using a different storage provider, you'll need to use their specific CLI or API to create snapshots. For example, for Google Cloud Persistent Disks:
    
    ```bash
    gcloud compute disks snapshot <disk-name> --snapshot-names=pv-snapshot-$(date +%Y-%m-%d)
    ```
    
4. For on-premises solutions, you might need to use your storage system's native backup capabilities. For instance, if you're using NFS, you could use `rsync`:
    
    ```bash
    rsync -avz /path/to/pv/data /path/to/backup/location
    ```
    
5. Verify your backups. For cloud providers, you can usually see your snapshots in the web console. For on-premises solutions, check your backup location:
    
    ```bash
    ls -l /path/to/backup/location
    ```
    

Remember, PV backups can be tricky and depend heavily on your specific setup. Always test your backup and restore process thoroughly!

## Kubernetes Objects: YAML You Glad You Backed These Up?

Backing up your Kubernetes objects is like keeping a copy of your favorite recipe. Sure, you could probably recreate it from memory, but why risk it?

1. Use kubectl to export all objects:
    
    ```bash
    kubectl get all --all-namespaces -o yaml > all-deploy-services-$(date +%Y-%m-%d-%H-%M-%S).yaml
    ```
    
    This command is the "select all" of Kubernetes. Use it wisely.
    
2. For a more selective approach:
    
    ```bash
    kubectl get deployment,service,configmap,secret -n my-namespace -o yaml > my-namespace-objects-$(date +%Y-%m-%d-%H-%M-%S).yaml
    ```
    
    Because sometimes, you don't need to backup *everything*. Just most things.
    
3. Verify your backup:
    
    ```bash
    grep -c "kind:" my-namespace-objects-$(date +%Y-%m-%d-%H-%M-%S).yaml
    ```
    
    This will count the number of resources in your backup file. If it's zero, you might want to check if you're in the right namespace or if you've angered the Kubernetes gods somehow.
    
4. For extra safety, you can also backup your kube-system namespace separately:
    
    ```bash
    kubectl get all -n kube-system -o yaml > kube-system-backup-$(date +%Y-%m-%d-%H-%M-%S).yaml
    ```
    
    This includes critical components like CoreDNS, kube-proxy, and your CNI plugin.
    

## Restore: Bringing Your Cluster Back from the Brink

Now, backing up is only half the battle. The real test comes when you need to restore. It's like those fire drills you did in school – you hope you never need it, but you'll be glad you practiced when the time comes.

### Restoring etcd:

1. Stop the API server:
    
    ```bash
    sudo systemctl stop kube-apiserver
    ```
    
2. Restore from the snapshot:
    
    ```bash
    ETCDCTL_API=3 etcdctl snapshot restore /path/to/backup/etcd-snapshot-2023-07-19-14-30-00.db \
    --data-dir /var/lib/etcd-restored
    ```
    
3. Update the etcd manifest to point to the new data directory:
    
    ```bash
    sudo sed -i 's/\/var\/lib\/etcd/\/var\/lib\/etcd-restored/g' /etc/kubernetes/manifests/etcd.yaml
    ```
    
4. Restart the kubelet to pick up the changes:
    
    ```bash
    sudo systemctl restart kubelet
    ```
    
5. Verify the restore:
    
    ```bash
    kubectl get nodes
    kubectl get pods --all-namespaces
    ```
    
    If you see your nodes and pods, congratulations! You've just performed time travel for your cluster.
    

### Restoring Persistent Volumes:

1. If you're using cloud provider snapshots, create new volumes from these snapshots. For AWS:
    
    ```bash
    aws ec2 create-volume --snapshot-id snap-1234567890abcdef0 --availability-zone us-west-2a
    ```
    
2. Create a new PV that references this volume:
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: restored-pv
    spec:
      capacity:
        storage: 5Gi
      accessModes:
        - ReadWriteOnce
      awsElasticBlockStore:
        volumeID: <volume-id>
        fsType: ext4
    ```
    
    Replace `<volume-id>` with the ID of the volume you just created.
    
3. Create a PVC to bind to this PV:
    
    ```yaml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: restored-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 5Gi
    ```
    
4. Update your deployment to use this new PVC.
    
5. For on-premises solutions, you might need to manually copy data back to your PV location:
    
    ```bash
    rsync -avz /path/to/backup/location /path/to/pv/data
    ```
    

### Restoring Kubernetes Objects:

1. Simply apply your backed-up YAML file:
    
    ```bash
    kubectl apply -f all-deploy-services-2023-07-19-14-30-00.yaml
    ```
    
2. Verify the restore:
    
    ```bash
    kubectl get all --all-namespaces
    ```
    
    This will show you all the resources in your cluster. Compare this with what you expect to see based on your backup.
    

It's like CTRL+Z for your entire cluster!

## Conclusion: Sleep Soundly, Backup Regularly

And there you have it, folks! A comprehensive guide to backing up and restoring your Kubernetes cluster using only native tools. Remember, a good backup strategy is like a good insurance policy – you hope you never need it, but you'll be eternally grateful when you do.

Here are some final tips to keep in mind:

1. **Automate your backups**: Set up a cron job to run your backup commands regularly.
    
2. **Test your restores**: Regularly practice restoring from your backups in a non-production environment.
    
3. **Keep multiple backups**: Don't rely on just one backup. Keep several, preferably in different locations.
    
4. **Document your process**: Write down your backup and restore procedures. Your future self (or your replacement when you're on vacation) will thank you.
    

So, backup early, backup often, and may your clusters always be resilient. And hey, the next time someone asks you what you do for a living, you can proudly say, "I'm a data superhero. I save clusters." It might not impress everyone at parties, but it'll certainly impress me.

Until next time, keep those clusters running and those backups current! 🚀

P.S. If you're wondering why I'm so passionate about backups, let's just say I've learned my lessons the hard way. But that's a story for another time, preferably over a strong cup of coffee or an even stronger adult beverage.

## Additional Resources

For those of you who want to dive even deeper into the world of Kubernetes backups and restores, here are some excellent resources:

1. [Kubernetes Documentation on Backing Up an etcd Cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
    
2. [Kubernetes Best Practices: Day 2 Operations](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-day-2-operations)
    
3. [etcd Documentation on Disaster Recovery](https://etcd.io/docs/v3.5/op-guide/recovery/)
    
4. [Kubernetes Documentation on Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
    

Remember, in the world of Kubernetes, knowledge is power. The more you know, the less likely you are to