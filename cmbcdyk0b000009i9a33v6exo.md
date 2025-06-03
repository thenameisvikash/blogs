---
title: "Apache Kafka Goes Zookeeper-Free: Your Complete Guide to KRaft Mode (Part 2)"
seoTitle: "Kafka Without Zookeeper: KRaft Mode Guide"
seoDescription: "Learn how to transition from Zookeeper to KRaft mode in Apache Kafka for improved performance and simplified architecture"
datePublished: Sat May 31 2025 15:27:37 GMT+0000 (Coordinated Universal Time)
cuid: cmbcdyk0b000009i9a33v6exo
slug: apache-kafka-goes-zookeeper-free-your-complete-guide-to-kraft-mode-part-2
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1748704482826/5f8bfde3-d04f-4a59-a4a5-0bab97dc6ea6.png
tags: apache-kafka-kraft, remove-zookeeper, docker-compose-kraft-kafka-cluster, production-grade-kafka-kraft-cluster, apache-kafak-zero-to-hero, apache-kafka-kraft-for-dev, step-by-step-guide-of-apache-kafka

---

Hello look who's back, Kafka enthusiasts! Remember me from our last adventure when we explored the world of Apache Kafka? If not, you can go back and read the previous post here [https://devopswizard.hashnode.dev/apache-kafka-for-devops-from-confusion-to-confidence-a-beginners-tutorial](https://devopswizard.hashnode.dev/apache-kafka-for-devops-from-confusion-to-confidence-a-beginners-tutorial) . Well, grab your favorite drink (mine's still coffee, of course), because we're about to start an even more exciting journey. Today, we're moving away from Zookeeper and stepping into the future with Kafka KRaft mode.

If you thought regular Kafka was impressive, just wait until you see what happens when we remove the Zookeeper dependency. Spoiler alert: it's like watching your favorite sports car shed 500 pounds – suddenly everything is faster, smoother, and much more fun to drive.

## From Zookeeper Dependency to Freedom: Understanding Kafka KRaft

Let me paint you a picture. It's 3 AM (again – why is it always 3 AM?), and your Zookeeper ensemble decides to have a meltdown right before Black Friday. Sound familiar? Yeah, we've all been there. That's exactly why the Apache Kafka team introduced KRaft (Kafka Raft) – to save us from these 3 AM panic attacks.

### What is KRaft?

KRaft stands for **Kafka Raft**, and it's basically Kafka saying, "You know what? I'm grown up now. I don't need Zookeeper to hold my hand anymore." Instead of relying on an external coordination service, Kafka now uses its own internal consensus algorithm based on the Raf[t protocol.](https://raft.github.io/)

[**Unde**](https://raft.github.io/)**rstanding the Raft Consensus Algorithm:**

The Raft protocol ens[ures that all](https://raft.github.io/) controller nodes maintain consistent metadata through a leader-follower model. Here's how it works:

* **Leader Election**: When controllers start up or the current leader fails, nodes vote for a new leader
    
* **Log Replic\*\*\*\*ation**: The leader receives all metadata changes and replicates them to follower controllers
    
* **Safety**: Only nodes with the most up-to-date logs can become leaders, ens[uring consist](https://raft.github.io/)ency
    

When the leader controller fails, the remaining controllers quickly elect a new leader through a majority vote (that's why we need an odd number of controllers), ensuring cluster operations continue without interruption – typically in milliseconds r[at](https://raft.github.io/)her than seconds.

### Architecture Comparison: Before and After

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748585933348/ba94330f-327a-414d-93ca-f28a261c6a0c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748585974343/a86b9a2a-b58b-4325-b361-808f5201a22c.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1748586048423/4a33fdbe-c4f3-4eb1-82cb-b20d0b60c06f.png align="center")

### Key Benefits of KRaft

Here's what KRaft brings to the party:

**🏗️ Simplified Architecture**: No more managing separate Zookeeper clusters – one less moving part to worry about

**⚡ Better Performance**: Faster metadata operations and quicker recovery times (controller failover in milliseconds)

**�\*\*\*\*� Enhanced Scalability**: Support for millions of partitions (yes, millions!) compared to Zookeeper's practical limits

**🔒 Improved Secur\*\*\*\*ity**: More granular control over authentication and authorization

**🛠️ Easier Operations**: One unified system to monitor, backup, and maintain

Think of it this way: if traditional Kafka with Zookeeper was like having a personal assistant who needed their own personal assistant, KRaft is like finally getting that super-efficient AI assistant who handles everything themselves.

## Why KRaft is a Game-Changer for DevOps Teams

Look, I'm not going to sugarcoat it – when KRaft was first announced, I was skeptical. "Another architectural change?" I thought. "Great, more things to learn." But after running KRaft in production for several months, I'm convinced it's one of the best things to happen to Kafka since... well, since Kafka itself.

Here's why DevOps teams are falling in love with KRaft:

**Operational Simplicity**: Remember when you had to monitor both Kafka brokers AND Zookeeper ensemble health? Those days are gone. With KRaft, you've got one unified system to monitor, back[up, and](https://raft.github.io/) maintain.

**Resourc**[**e Efficien**](https://raft.github.io/)**cy**: No more dedicating sepa[ra](https://raft.github.io/)te machines for Zookeeper. Your Kafka brokers now pull double duty as both data handlers and cluster coordinato[rs. It's like](https://raft.github.io/) getting a Swiss Army knife instead of carrying separate tools.

**Lightning-Fast R\*\*\*\*ecovery**: Controller f[ailover in KR](https://raft.github.io/)aft happens in milliseconds, not seconds. I've seen clusters bounce back from controller failures so fast [that monitori](https://raft.github.io/)ng systems barely register the blip.

**Massive Scale Support**: Need to h[andle millio](https://raft.github.io/)ns of partitions? KRaft laughs at your cute little scaling chall[enges. We're](https://raft.github.io/) talking about clusters that can handle enterprise-grade workloads that would make traditional Kafka+Zookeeper setups weep.

## Migrating from Zookeeper to KRaft: What You Need to Know

If you're currently running Kafka with Zo[okeeper, here](https://raft.github.io/)'s your migration planning checkl[ist:](https://raft.github.io/)

### [Pre-Mig](https://raft.github.io/)ration Assessment

* \[ \] **Kafka Version**: Ensure you're running Kafka 2.8+ (KRaft stable in 3.3+)
    
* \[ \] **Custom C\*\*\*\*onfigurations**: Audit any Zookeeper-specific configurations
    
* \[ \] **Monitoring**: Update monitoring systems to track KRaft-specific metrics
    
* \[ \] **Client Compatibility**: Verify all client applicati[ons work with](https://raft.github.io/) KRaft
    

### Migration Strategies

**Op\*\*\*\*tion 1: Blue-Green** [**Depl**](https://raft.github.io/)**oyment** (Recommended for Production)

1. Set up parallel KRaft cluster
    
2. Migrate topics using MirrorMaker 2.0
    
3. Switch clients to new cluster
    
4. Decommission old cluster
    

**Option 2: In-Place** [**Migration** (L](https://raft.github.io/)imited Support)

* Currently experimental
    
* Requires careful planning and testing
    
* Recommended only for development environments
    

### Rollback Planning

Always have a rollback plan:

* Keep Zookeeper cluster running during transition period
    
* Maintain topic-level replication for critical data
    
* Test rollback procedures in staging environment
    

## Setting Up a Production-Grade KRaft Cluster with SASL Authentication

Alright, e[nough t](https://raft.github.io/)heory – let's get our hands dirty! We're going to set up a KRaft cluster that's not just functional, but production-ready with proper authentication. Because let's face it, nobody wants to be the person who left their Kafka cluster wide open to the internet.

### What You'll Need This Time

* Docker [and Docker C](https://raft.github.io/)ompose (our trusty companions)
    
* A basic understanding of SASL authentication ([don't worry,](https://raft.github.io/) I'll explain)
    
* [About 30 mi](https://raft.github.io/)nutes of your time
    
* A strong cup of coffee (optional but highly [recommended)](https://raft.github.io/)
    

### Understanding SASL and JAAS

Before we dive in, let's cla[rify some sec](https://raft.github.io/)urity terms:

**SASL (Simple Aut\*\*\*\*hentication and Security Layer)** is [a framework](https://raft.github.io/) for authentication and data security. SASL\_PLAINTEXT provides u[sername/passw](https://raft.github.io/)ord authe[ntication ove](https://raft.github.io/)r unencrypted connections. While the authentication i[s secure, t](https://raft.github.io/)he data transmission [isn't](https://raft.github.io/) encrypted – suitable for internal [networks but](https://raft.github.io/) not for internet-fa[cing deployme](https://raft.github.io/)nts.

**JAAS (Jav**[**a Authenticat**](https://raft.github.io/)**ion and Authorization Service)** is Jav[a's pluggable](https://raft.github.io/) authenticati[on framework.](https://raft.github.io/) In Kafka, it defines how cl[ients and bro](https://raft.github.io/)kers authenticate with each other, s[upporting var](https://raft.github.io/)ious m[echanisms lik](https://raft.github.io/)e PLAIN, SCRAM, and [Kerberos.](https://raft.github.io/)

### [T](https://raft.github.io/)he Magic Behind Our KRaft Setup

Before we div[e into the co](https://raft.github.io/)nfiguration, let me explain what makes th[is setup spec](https://raft.github.io/)ial:

* **No Zookeeper**: Obviously, but i[t still feels](https://raft.github.io/) weird saying it
    
* **SASL Authentication**: Proper username/[password auth](https://raft.github.io/)entication
    
* **Multiple Security Profiles**: Different users with different permissions
    
* **Web UI Integration**: Because clicking buttons is sometimes better than typing commands
    
* **Production-Ready Configs**: Settings that won't embarrass you in front of your infrastruc[t](https://raft.github.io/)ure team
    

## Step 1: The Produ[ction-Grade D](https://raft.github.io/)ocker Compose Configuration

Here's our KRaft clus[ter](https://raft.github.io/) configuration. Don't let the size intimidate you – I'll break [it do](https://raft.github.io/)wn piece by piece:

```yaml
git clone https://github.com/thenameisvikash/kafka_kraft_cluster.git
cd kafka_kraft_cluster
# Setup and Start the Cluster
./automate.sh --setup

```

This command will:

1. Create necessary directories
    
2. Generate configuration files
    
3. Start all containers
    
4. Create test topics
    
5. Set up ACLs
    
6. Run comprehensive tests
    
    ### Check KRaft Controller Status
    
    To view the current KRaft controller status and quorum information:
    
7. ./[automate.sh](http://automate.sh) --controller-status
    
    ### Destroy the Environment
    
    To stop all containers and clean up:
    
8. ./[automate.sh](http://automate.sh) --destroy
    

## Step 2: Understanding the KRaft Configuration Magic

Let me break down the key differences from our previous Zookeeper-based setup:

### Process Roles Deep Dive

```yaml
KAFKA_PROCESS_ROLES: 'broker,controller'
```

This tells each node to act as both:

* **Broker**: Handles data storage, replication, and client requests
    
* **Controller**: Manages cluster metadata, partition assignments, and leader elections
    

In traditional Kafka, these roles were separate – brokers handled data while Zookeeper managed metadata. KRaft combines them, reducing complexity and improving performance.

### Controller Quorum Configuration

```yaml
KAFKA_CONTROLLER_QUORUM_VOTERS: >
  1@kafka-kraft-node-1:29093,
  2@kafka-kraft-node-2:29093,
  3@kafka-kraft-node-3:29093
```

This defines which nodes participate in the controller election. Think of it as a democratic election where these three nodes vote on who gets to be the cluster leader. We use an odd number (3) to prevent split-brain scenarios.

### Cluster Identity

```yaml
CLUSTER_ID: 'kLuXZqjeTrSxmQe1OlKn1Q'
```

Unlike Zookeeper setups, KRaft requires a predefined cluster ID. This ensures cluster identity consistency from day one. Generate yours using:

```bash
kafka-storage random-uuid
```

### Metadata Storage

```yaml
KAFKA_METADATA_LOG_DIR: '/var/lib/kafka/metadata'
KAFKA_METADATA_LOG_SEGMENT_BYTES: '10485760'
```

The `KAFKA_METADATA_LOG_DIR` is where KRaft stores its internal metadata logs – essentially replacing what Zookeepe[r used to han](https://raft.github.io/)dle. This includes topic configurations, [partition as](https://raft.github.io/)signments, ACLs, and controller state.

### JVM Performance Tuning Expl[ained](https://raft.github.io/)

```yaml
KAFKA_JVM_PERFORMANCE_OPTS: >
  -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 
  -XX:InitiatingHeapOccupancyPercent=35
```

* **\-XX:+UseG1GC**: Enables the G1 (Garbage First) [collector, op](https://raft.github.io/)timized for low-latency applications
    
* **\-XX:MaxGCPauseM\*\*\*\*illis=20**: Targets [garbage coll](https://raft.github.io/)ection pauses under 20 milliseconds
    
* **\-XX:InitiatingHeapOccupancyPercent=35**: Starts concurrent GC when heap is 35% full
    

## Step 3: Security First - Implementing [SASL\_PLAINTE](https://raft.github.io/)XT

Now, let's talk security. Because in 2024, running an unsecured Kafka cluster is like leaving your front door wide open with a sign that says ["Free stuff](https://raft.github.io/) inside!"

### Creating Your JAAS Configuration

Create a directory structure:

```bash
mkdir -p config
```

Create the JAAS configuration file `config/kafka_server_jaas.conf`:

```bash
# JAAS configuration for Kafka servers
KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin-secret"
    user_admin="admin-secret"
    user_kafka="kafka-secret"
    user_client="client-secret"
    user_producer="producer-secret"
    user_consumer="consumer-secret";
};

# JAAS configuration for clients
Client {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="client"
    password="client-secret";
};
```

This configuration creates multiple users with different roles:

* **admin**: Your cluster superhero with full perm[issions](https://raft.github.io/)
    
* [**ka**](https://raft.github.io/)**fka**: For internal Kafka operations
    
* **client**: For regular application connections
    
* **producer**: Specialized for producing messages
    
* **consumer**: Sp[ecialized f](https://raft.github.io/)or consuming messages
    

### Setting Up Client Authentication Profiles

Create these clie[nt prope](https://raft.github.io/)rty files in your `config` directory:

**Adm\*\*\*\*in Client (**`config/`[`admin.properties`](http://admin.properties)):

```bash
# Admin client configuration with full cluster access
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="admin" password="admin-secret";
```

**Regular Client (**`config/`[`client.properties`](http://client.properties)):

```bash
# Standard client configuration for applications
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="client" password="client-secret";
```

**Producer Client (**`config/`[`producer.properties`](http://producer.properties)):

```bash
# Specialized producer configuration
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="producer" password="producer-secret";
```

**Consumer Client (**`config/`[`consumer.properties`](http://consumer.properties)):

```bash
# Specialized consumer configuration
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="consumer" password="consumer-secret";
```

**Unauthor**[**ized Client (**](https://raft.github.io/)`config/`[`unauthorized.properties`](http://unauthorized.properties)[) - for te](https://raft.github.io/)sting:

```bash
# This should fail authentication - useful for security testing
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required \
    username="unauthorized" password="wrong-secret";
```

## Step 4: Bringing Your KRaft Cluster to Life

Time for the moment of truth! Navigate to your project directory and start the cluster:

```bash
# Create the necessary directories
mkdir -p data/kafka-kraft-node-{1,2,3}
mkdir -p data/kafka-kraft-node-{1,2,3}-meta

# Start the cluster
docker-compose up -d

# Watch the startup process
docker-compose logs -f
```

Watch the magic happen as your KRaft cluster comes online. No Zookeeper, no external dependencies – just pure Kafka goodness.

### Verification Checklist

Let's make sure everything is wor[king correctl](https://raft.github.io/)y:

```bash
# Check container status
docker-compose ps

# Verify all containers are healthy
docker-compose ps | grep Up

# Check cluster metadata
docker exec -it kafka-kraft-node-1 kafka-log-dirs \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties \
  --describe

# Test the UI (open http://localhost:7095 in your browser)
curl -s http://localhost:7095/actuator/health
```

If you see all containers running and can access the Kafka UI, congratulations! You've just deployed a production-grade KRaft cluster.

### Healt[h Check Imple](https://raft.github.io/)mentation

Our Docker Compose includes health checks for each node:

```yaml
healthcheck:
  test: ["CMD-SHELL", "kafka-topics --bootstrap-server localhost:9092 --list --command-config /etc/kafka/admin.properties || exit 1"]
  interval: 30s
  timeout: 10s
  retries: 3
```

This ensures that:

* The Kafka service is respondi[ng](https://raft.github.io/)
    
* [Authent](https://raft.github.io/)ication is working
    
* The node c[an participat](https://raft.github.io/)e in cluster operations
    

## Step 5: Real-World Testing - Producer-Consumer Operations with Authentication

Now for the fun part – let's actually use our authenticated cluster:

### Creating Topics with Authentication

```bash
# Create a topic for secure orders
docker exec -it kafka-kraft-node-1 kafka-topics \
  --create \
  --topic secure-orders \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties \
  --partitions 6 \
  --replication-factor 3

# Create a topic for user events
docker exec -it kafka-kraft-node-1 kafka-topics \
  --create \
  --topic user-events \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties \
  --partitions 12 \
  --replication-factor 3

# List all topics to verify creation
docker exec -it kafka-kraft-node-1 kafka-topics \
  --list \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties
```

### Setting Up Access Control Lists (ACLs)

For production security, set up proper ACLs:

```bash
# Allow producer user to write to secure-orders topic
docker exec -it kafka-kraft-node-1 kafka-acls \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties \
  --add \
  --allow-principal User:producer \
  --operation Write \
  --topic secure-orders

# Allow consumer user to read from secure-orders topic
docker exec -it kafka-kraft-node-1 kafka-acls \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties \
  --add \
  --allow-principal User:consumer \
  --operation Read \
  --topic secure-orders \
  --group secure-orders-consumers

# List ACLs to verify
docker exec -it kafka-kraft-node-1 kafka-acls \
  --bootstrap-server localhost:9092 \
  --command-config /etc/kafka/admin.properties \
  --list
```

### Producing Messages with Authentication

```bash
# Start an authenticated producer
docker exec -it kafka-kraft-node-1 kafka-console-producer \
  --topic secure-orders \
  --bootstrap-server localhost:9092 \
  --producer.config /etc/kafka/client.properties
```

### C[onsuming Mess](https://raft.github.io/)ages with Authentication

```bash
# Start an authenticated consumer
docker exec -it kafka-kraft-node-1 kafka-console-consumer \
  --topic secure-orders \
  --bootstrap-server localhost:9092 \
  --consumer.config /etc/kafka/client.properties \
  --from-beginning
```

### Testing Security (The Fun Part)

Try connecting with wrong credentials:

```bash
# This should fail beautifully
docker exec -it kafka-kraft-node-1 kafka-console-consumer \
  --topic secure-orders \
  --bootstrap-server localhost:9092 \
  --consumer.config /etc/kafka/unauthorized.properties
```

If you see authentication errors, that's actually good news – your security is working!

## Performance Tuning Your KRaft Cluster

Here are some production-ready optimizations I've learned the hard way:

### Memory Configuration

```yaml
environment:
  KAFKA_HEAP_OPTS: "-Xmx4G -Xms4G"  # Adjust based on your available RAM
  KAFKA_JVM_PERFORMANCE_OPTS: "-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35"
```

### Metadata Log Tuning

```yaml
environment:
  KAFKA_METADATA_LOG_SEGMENT_BYTES: '10485760'  # 10MB segments
  KAFKA_METADATA_LOG_RETENTION_MS: '604800000'   # 7 days retention
```

### Network Optimization

```yaml
environment:
  KAFKA_SOCKET_SEND_BUFFER_BYTES: '102400'      # 100KB
  KAFKA_SOCKET_RECEIVE_BUFFER_BYTES: '102400'   # 100KB
  KAFKA_SOCKET_REQUEST_MAX_BYTES: '104857600'   # 100MB
```

## Troubleshooting Common KRaft Issues

After months of running KRaft in production, here are the issues I've encountered and how to fix them:

### Controller Election Problems

**Symptom**: Cluster seems stuck, no leader election happening **Solution**: Check your `KAFKA_CONTROLLER_QUORUM_VOTERS` configuration and ensure at least 2 out of 3 controllers are healthy.

### Authentication Failures

**Symptom**: Clients can't connect despite correct credentials **Solution**: Verify your JAAS configuration is properly mounted and the usernames/passwords match exactly.

### Slow Metadata Operations

**Symptom**: Topic creation or deletion takes forever **Solution**: Check your `KAFKA_METADATA_LOG_DIR` has enough disk space and consider tuning `KAFKA_METADATA_LOG_SEGMENT_BYTES`.

## What's Next: Advanced KRaft Configurations

We've covered the basics, but KRaft has so much more to offer:

* **Multi-Region Deployments**: Setting up KRaft across different availability zones
    
* **Advanced Security**: Implementing SSL/TLS encryption on top of SASL
    
* **Performance Monitoring**: Deep-dive into KRaft-specific metrics
    
* **Disaster Recovery**: Backup and restore strategies for KRaft clusters
    
* **Integration Patterns**: Connecting KRaft clusters with your existing infrastructure
    

## Wrapping Up Your KRaft Journey

And there you have it – you've successfully migrated from the Zookeeper era into the brave new world of KRaft! Your cluster is now more resilient, performs better, and is easier to manage. Plus, you'll never have to debug Zookeeper connection issues at 3 AM again (trust me, your future self will thank you).

The transition to KRaft isn't just about removing a dependency – it's about embracing a more mature, scalable, and maintainable approach to distributed streaming. As someone who's been through the trenches of both architectures, I can confidently say that KRaft is the future of Kafka.

Remember, like any powerful tool, KRaft requires respect and understanding. Start with development environments, get comfortable with the new concepts, and gradually roll it out to production. Don't rush – good architecture decisions are like good coffee, they take time to brew properly.

Until next time, happy streaming! And remember, in the world of distributed systems, the only constant is change – but at least now it's change without Zookeeper!

---

## Additional Resources for Your KRaft Journey

* [Apache Kafka KRaft Documentation](https://kafka.apache.org/documentation/#kraft)
    
* [KRaft Migration Guide](https://cwiki.apache.org/confluence/display/KAFKA/KIP-833%3A+Mark+KRaft+as+Production+Ready)
    
* [Security Best Practices for KRaft](https://docs.confluent.io/platform/current/kafka/authentication_sasl/index.html)
    
* [KRaft Performance Tuning Guide](https://kafka.apache.org/documentation/#kraftmode)
    

*Got questions about KRaft? Hit me up in the comments below – I love talking shop about distributed systems!*