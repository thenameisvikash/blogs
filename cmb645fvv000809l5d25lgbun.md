---
title: "Step-by-Step Guide to Build a Stable MySQL NDB Cluster Using Docker"
seoTitle: "Build MySQL NDB Cluster with Docker: A Guide"
seoDescription: "Build a reliable MySQL NDB Cluster with Docker using this guide on architecture, setup, and scalable database practices"
datePublished: Tue May 27 2025 06:06:25 GMT+0000 (Coordinated Universal Time)
cuid: cmb645fvv000809l5d25lgbun
slug: step-by-step-guide-to-build-a-stable-mysql-ndb-cluster-using-docker
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1748281817548/e601949b-0ead-4749-af57-29b0495ebf67.png
tags: mysql-ndb-cluster, ndb-cluster-using-docker, what-exactly-is-mysql-ndb-cluster, traditional-mysql-vs-ndb-cluster, from-zero-to-running-cluster

---

*Remember that feeling when you first heard about distributed databases and thought "that sounds impossible to set up"? Well, I just spent weeks diving deep into MySQL NDB Cluster, and spoiler alert: it's actually pretty amazing once you get past the initial complexity. Let me share what I learned.*

## Why I Decided to Tackle MySQL NDB Cluster

After my Kafka journey - starting with [Apache Kafka for DevOps: From Confusion to Confidence](https://devopswizard.hashnode.dev/apache-kafka-for-devops-from-confusion-to-confidence-a-beginners-tutorial) and then diving deep into [production-grade Kafka cluster deployment with Ansible](https://devopswizard.hashnode.dev/step-by-step-guide-to-kafka-cluster-deployment-using-ansible) - got some amazing response from the community (the [GitHub repo](https://github.com/thenameisvikash/kafkacluseterplaybook/tree/master) has been forked and starred way more than I expected!), I realized there's something magical about sharing these complex infrastructure setups in a way that doesn't make people want to throw their laptops out the window.

MySQL NDB Cluster has been on my "someday I'll figure this out" list for ages. When you're dealing with applications that need to handle 150,000 transactions per second and process 60 crore (600 million) records daily, traditional MySQL replication starts showing its limitations. That's when you start looking at horizontal scaling solutions like NDB Cluster.

**The challenge?** Most tutorials assume you have a PhD in distributed systems or skip the "why" entirely.

**My goal?** Create a setup that's beginner-friendly but production-ready, with all the gotchas and real-world considerations documented.

## What Exactly is MySQL NDB Cluster? (And Why Should You Care?)

Think of MySQL NDB Cluster as MySQL's answer to "what if we could make the database itself horizontally scalable?" It's not just replication – it's true distributed computing for your data.

Here's the deal:

### Traditional MySQL vs NDB Cluster

**Traditional MySQL:**

* One primary server, multiple read replicas
    
* If the primary goes down, you need failover procedures
    
* Scaling reads? Add more replicas. Scaling writes? Good luck.
    

**MySQL NDB Cluster:**

* Multiple data nodes, each handling both reads and writes
    
* Automatic failover (no manual intervention needed)
    
* Linear scaling: add more nodes = more capacity
    
* ACID compliance across the entire distributed system
    

### The "Aha!" Moment

The breakthrough came when I understood **node groups**. Picture this:

```bash
Node Group 0: [Data Node 1] ←→ [Data Node 2]
Node Group 1: [Data Node 3] ←→ [Data Node 4]
```

Each node group contains a complete copy of your data. Data is automatically partitioned across node groups, and within each group, it's replicated for high availability. Lose one node? No problem. Lose an entire node group? Still running (though you'll want to fix that quickly).

## The Architecture: What We're Building

My setup includes everything you'd need for a production-like environment:

* **2 Management Nodes**: The "brain" that coordinates everything
    
* **4 Data Nodes**: Organized into 2 node groups for optimal performance
    
* **2 ProxySQL Instances**: Load balancing and connection pooling
    
* **2 MySQL API Nodes**: Your application's entry point
    

All wrapped up in Docker Compose because, let's be honest, nobody wants to manually configure 10 different services.

### Why This Configuration?

**Two Management Nodes**: Redundancy. If one management node dies, the other keeps the show running.

**Four Data Nodes in Two Groups**: This gives you both high availability and performance. You can lose up to one node per group and still serve all your data.

**ProxySQL**: This is where the magic happens for application developers. ProxySQL handles:

* Connection pooling (your app doesn't need to manage connections to 4+ nodes)
    
* Load balancing (automatically distributes queries)
    
* Query routing (sends reads and writes to optimal nodes)
    
* Failover detection (automatically routes around failed nodes)
    

## The Setup: From Zero to Running Cluster

### Prerequisites (The Boring But Important Stuff)

You'll need:

* Docker and Docker Compose (obviously)
    
* At least 8GB RAM (NDB Cluster is memory-hungry)
    
* Basic understanding of MySQL (you should know what a PRIMARY KEY is)
    

### Getting Started

```bash
git clone https://github.com/thenameisvikash/mysql_ndb_cluster.git
cd mysql_ndb_cluster
docker-compose up -d
docker exec management1 ndb_mgm --ndb-connectstring=localhost:1186 -e "show"
```

*Yes, it's really that simple to get started. But let's talk about what's happening under the hood.*

**Expected output:**

## `Connected to Management Server at: mysql-mgm1:1186 Cluster Configuration`

`[ndbd(NDB)] 4 node(s) id=3 @mysql-data1 (mysql-5.7.33 ndb-7.6.17, Nodegroup: 0, *) id=4 @mysql-data2 (mysql-5.7.33 ndb-7.6.17, Nodegroup: 0) id=5 @mysql-data3 (mysql-5.7.33 ndb-7.6.17, Nodegroup: 1) id=6 @mysql-data4 (mysql-5.7.33 ndb-7.6.17, Nodegroup: 1)`

`[ndb_mgmd(MGM)] 2 node(s) id=1 @mysql-mgm1 (mysql-5.7.33 ndb-7.6.17) id=2 @mysql-mgm2 (mysql-5.7.33 ndb-7.6.17)`

`[mysqld(API)] 2 node(s) id=7 @mysql1 (mysql-5.7.33 ndb-7.6.17) id=8 @mysql2 (mysql-5.7.33 ndb-7.6.17)`

```bash
# Run the comprehensive test suite which will auto test cluster
./mysql_cluster_test_suite.sh



# or Connect through ProxySQL (recommended)
mysql -h localhost -P 6033 -u root -p
```

## What Makes This Repository Special

Looking at other MySQL Cluster tutorials and setups online, most give you the bare minimum - a basic config file and a "good luck figuring out the rest" attitude. Here's what makes this different:

### 1\. **Complete Documentation Ecosystem**

* **MySQL\_Cluster\_DevOps\_Guide.md (23KB)**: Everything from architecture to troubleshooting
    
* **High\_Throughput\_Architecture.md**: Specifically for high-performance scenarios (like that 150K TPS requirement)
    
* **Troubleshooting\_Guide.md**: Real issues and real solutions
    
* **Backup\_and\_Recovery.md**: Operational procedures you'll actually need
    

### 2\. **Production-Ready Components**

* **Comprehensive test suite**: 23KB of testing scripts that validate everything
    
* **Architecture visualization**: Interactive HTML viewer plus SVG diagrams
    
* **Load testing tools**: Python scripts for performance validation
    
* **Automation scripts**: ProxySQL setup, cluster management, etc.
    

### 3\. **Real-World Focus**

Every piece of documentation addresses actual challenges you'll face:

* Node group expansion procedures
    
* Upgrade strategies
    
* Partitioning limitations and workarounds
    
* Performance optimization techniques
    

### 4\. **Visual Learning**

The `architecture.svg` and `view_architecture.html` files let you actually *see* how the components interact. Sometimes a diagram is worth a thousand lines of configuration!

### The Heart: config.ini

```ini
[ndb_mgmd default]
DataDir=/var/lib/mysql-cluster

[ndb_mgmd]
NodeId=1
HostName=mysql-mgm1
ArbitrationRank=1    # Primary arbitrator

[ndb_mgmd]
NodeId=2
HostName=mysql-mgm2
ArbitrationRank=2    # Secondary arbitrator (higher priority)

[ndbd default]
NoOfReplicas=2
DataMemory=80M       # Increase for production
IndexMemory=18M
Arbitration=Default  # Prevents split-brain scenarios
HeartbeatIntervalDbDb=1500
TimeBetweenWatchDogCheck=6000

# Node Group 0
[ndbd]
NodeId=3
HostName=mysql-data1
NodeGroup=0

[ndbd]
NodeId=4
HostName=mysql-data2
NodeGroup=0

# Node Group 1  
[ndbd]
NodeId=5
HostName=mysql-data3
NodeGroup=1

[ndbd]
NodeId=6
HostName=mysql-data4
NodeGroup=1
```

### Docker Compose Magic

The `docker-compose.yml` orchestrates everything:

```yaml
services:
  mysql-mgm1:
    image: mysql/mysql-cluster:8.0
    command: ndb_mgmd --config-file=/etc/mysql-cluster.cnf
    volumes:
      - ./config/config.ini:/etc/mysql-cluster.cnf
    
  mysql-data1:
    image: mysql/mysql-cluster:8.0  
    command: ndbd
    depends_on:
      - mysql-mgm1
      - mysql-mgm2
```

### ProxySQL Configuration

The repository includes a complete ProxySQL setup that handles the user management complexity I mentioned earlier:

```sql
-- proxysql_config.sql (included in the repo)
INSERT INTO mysql_servers(hostgroup_id, hostname, port, weight) VALUES
(0, 'mysql1', 3306, 900),
(0, 'mysql2', 3306, 900);

-- Health checks to automatically handle node failures
INSERT INTO mysql_users(username, password, default_hostgroup) VALUES
('myapp', 'password', 0);
```

### Automated Scripts

I've included several utility scripts that I wish I'd had when starting:

**cluster\_status.sh** - Quick health check:

```bash
#!/bin/bash
echo "=== Cluster Status ==="
docker exec mysql-mgm1 ndb_mgm -e "show"
echo "=== ProxySQL Status ==="
docker exec proxysql1 mysql -u admin -padmin -h 127.0.0.1 -P 6032 -e "SELECT * FROM runtime_mysql_servers;"
```

**backup\_cluster.sh** - Consistent backups:

```bash
#!/bin/bash
BACKUP_ID=$(docker exec mysql-mgm1 ndb_mgm -e "START BACKUP" | grep "Backup.*started" | awk '{print $2}')
echo "Backup started with ID: $BACKUP_ID"
```

## The Split-Brain Problem (And How We Solve It)

Here's where things get interesting from a DevOps perspective. Imagine your cluster gets split due to a network partition:

```bash
Partition A: [Management1] [Data1] [Data2]
     X  <-- Network Split
Partition B: [Management2] [Data3] [Data4]
```

Without arbitration, both partitions think they're the "real" cluster and start accepting writes. When the network heals, you have conflicting data – a split-brain scenario that can corrupt your entire dataset.

**MySQL NDB's Solution: Arbitration**

The management nodes act as arbitrators. When a split occurs:

1. Each partition checks if it has a majority of nodes
    
2. The arbitrator decides which partition survives
    
3. The "losing" partition shuts down gracefully
    
4. When the network heals, stopped nodes rejoin automatically
    

This is configured with:

```ini
[ndbd default]
Arbitration=Default
ArbitrationTimeout=7500
```

### Node Groups: The Key to Understanding Performance

Here's something that confused me initially: **why do we need node groups?**

Data in NDB Cluster is partitioned using a hash function across node groups, not individual nodes. So with 2 node groups:

* Partition 0 → Node Group 0 (stored on both nodes in the group)
    
* Partition 1 → Node Group 1 (stored on both nodes in the group)
    
* Partition 2 → Node Group 0
    
* Partition 3 → Node Group 1
    
* And so on...
    

**The performance implication**: Adding more node groups increases both capacity and throughput. Adding more nodes to existing groups only increases redundancy.

For high-throughput applications (like that 150K TPS requirement), you want more node groups, not just more nodes.

## ProxySQL: The Unsung Hero

While NDB Cluster handles the distributed database magic, your applications still need a clean way to connect. That's where ProxySQL shines.

### What ProxySQL Does for You

1. **Connection Multiplexing**: Your app opens a few connections to ProxySQL, which manages hundreds of connections to the backend
    
2. **Load Balancing**: Automatically distributes queries across all available MySQL nodes
    
3. **Health Checking**: Detects failed nodes and routes around them
    
4. **Query Caching**: Can cache frequent queries for even better performance
    

### The Configuration

```sql
-- ProxySQL automatically discovers all MySQL nodes
INSERT INTO mysql_servers(hostgroup_id, hostname, port, weight) VALUES
(0, 'mysql1', 3306, 900),
(0, 'mysql2', 3306, 900);

-- Load balance read and write queries
INSERT INTO mysql_query_rules(rule_id, active, match_pattern, destination_hostgroup, apply) VALUES
(1, 1, '^SELECT.*', 0, 1),
(2, 1, '^INSERT|UPDATE|DELETE.*', 0, 1);
```

## Production Considerations (The Real-World Stuff)

### Memory Sizing

The development setup uses tiny memory allocations. For production, you'll want:

```ini
[ndbd default]
DataMemory=4G        # Start with 4GB per node
IndexMemory=1G       # 25% of DataMemory is a good starting point
```

**Rule of thumb**: Size DataMemory to hold your entire working dataset in RAM. NDB Cluster is designed for in-memory operation.

### Network Configuration

NDB Cluster is chatty between nodes. You'll want:

* Dedicated network interfaces for cluster communication
    
* Low latency between nodes (&lt; 1ms ideally)
    
* Redundant network paths to prevent split-brain
    

### Monitoring and Alerting

Set up monitoring for:

* Node status (`ndb_mgm -e "show"`)
    
* Memory usage per node
    
* Network latency between nodes
    
* Query response times through ProxySQL
    

## Partitioning: The Performance Multiplier

One of NDB Cluster's coolest features is automatic partitioning, but there are some gotchas:

### What's Supported

* **KEY partitioning**: Hash-based partitioning using primary key
    
* **LINEAR KEY partitioning**: Better for adding/removing nodes
    

### What's NOT Supported

* RANGE partitioning
    
* LIST partitioning
    
* Standard HASH partitioning
    

### Maximum Partitions

The formula is: `8 × LDM_threads × node_groups`

With multi-threaded data nodes (ndbmtd) and 2 node groups: `8 × 4 × 2 = 64 partitions maximum`

### Best Practices for Keys

```sql
-- Good: Evenly distributed
CREATE TABLE orders (
    order_id VARCHAR(36) NOT NULL,  -- UUID
    customer_id INT,
    order_date DATETIME,
    PRIMARY KEY (order_id)
) ENGINE=NDB;

-- Avoid: Sequential keys create hot spots
CREATE TABLE bad_orders (
    order_id INT AUTO_INCREMENT,    -- All writes go to one partition
    customer_id INT,
    PRIMARY KEY (order_id)
) ENGINE=NDB;
```

## When to Use NDB Cluster (And When Not To)

### Perfect Use Cases

* High-throughput OLTP applications
    
* Applications requiring 99.99%+ uptime
    
* Workloads that need linear scaling
    
* Real-time applications (gaming, trading, IoT)
    

### Maybe Not the Best Fit

* Data warehouse / OLAP workloads
    
* Applications with complex queries and joins
    
* Small applications (the complexity isn't worth it)
    
* Budget-constrained projects (it's resource-intensive)
    

## My "Aha" Moments (And the Gotchas That Got Me)

### The User Management Surprise

Here's something that caught me completely off-guard: **when you create a user on one MySQL node, it doesn't automatically appear on the other nodes!**

I spent hours debugging why my application could connect to `mysql1` but not `mysql2` with the same credentials. Turns out, MySQL user accounts and permissions are stored in the `mysql` system database, which still uses InnoDB, not the NDB storage engine.

**What this means in practice:**

* Your application tables (using ENGINE=NDB) are automatically synchronized across all nodes
    
* User accounts, grants, and MySQL system tables remain local to each MySQL server
    
* You need to create the same users on all MySQL nodes, or use ProxySQL for centralized connection management
    

```sql
-- You have to do this on EACH MySQL node, or...
CREATE USER 'myapp'@'%' IDENTIFIED BY 'password';
GRANT ALL ON mydb.* TO 'myapp'@'%';

-- ...use ProxySQL to manage connections centrally (recommended)
```

This is actually why ProxySQL becomes so valuable - it acts as a single point of authentication and routes connections to healthy MySQL nodes automatically.

### Other Gotchas That Tripped Me Up

### 1\. Foreign Keys Are Limited

NDB supports foreign keys, but only within the same node group. Cross-node-group foreign keys aren't supported. I learned this the hard way when trying to migrate an existing schema.

### 2\. Transaction Size Limits

Large transactions can fail with cryptic error messages. The solution? Break them into smaller chunks. I had to rewrite some of my bulk insert operations to process data in batches.

### 3\. The Memory Reality Check

Everything in NDB Cluster lives in RAM. When I first tried to import a 2GB dataset into my development cluster with 80MB DataMemory... well, let's just say it didn't go well. Size your memory properly from the start.

### 4\. Backup Strategy

Use `ndb_mgm -e "START BACKUP"` for consistent cluster-wide backups. Regular mysqldump won't give you consistency across nodes. I created a simple script that's included in the repository:

```bash
#!/bin/bash
# backup_cluster.sh - Included in the repository
echo "Starting NDB Cluster backup..."
docker exec mysql-mgm1 ndb_mgm -e "START BACKUP"
echo "Backup completed. Check cluster logs for backup ID and location."
```

## Getting Started with My Setup

The [repository](https://github.com/thenameisvikash/mysql_ndb_cluster.git) includes everything you need:

* **Complete Docker Compose configuration** with all services
    
* **Sample data and queries** to test the setup
    
* **Monitoring scripts** for health checks and status monitoring
    
* **Production tuning guidelines** in the `docs/` folder
    
* **Troubleshooting guide** with common issues and solutions
    
* **Backup and recovery scripts** for operational tasks
    

### Quick Start Commands

```bash
# Clone the comprehensive setup
git clone https://github.com/thenameisvikash/mysql_ndb_cluster.git
cd mysql_ndb_cluster

# Start the entire cluster
docker-compose up -d

# Wait for cluster to initialize (check logs)
docker-compose logs -f mysql-mgm1

# Run the comprehensive test suite I created
./mysql_cluster_test_suite.sh

# Check cluster status using management commands
docker exec mysql-mgm1 ndb_mgm -e "show"

# Connect through ProxySQL (handles the user sync issue)
mysql -h localhost -P 6033 -u root -p

# Or connect directly to a MySQL node
mysql -h localhost -P 3306 -u root -p

# Initialize with sample data
mysql -h localhost -P 6033 -u root -p < init.sql

# Run performance testing with the included Python script
python3 test-load.py

# Set up ProxySQL with the automation script
./proxysql-setup.sh

# View the architecture diagram
# Open view_architecture.html in your browser to see the visual layout
```

The `mysql_cluster_test_suite.sh` script (23KB of comprehensive testing!) will validate:

* Cluster connectivity and node health
    
* Data consistency across nodes
    
* Failover scenarios
    
* Performance benchmarks
    
* ProxySQL configuration
    

**Want to see the architecture visually?** Open `view_architecture.html` in your browser - I've included an interactive diagram that shows how all the components connect.

### Repository Structure (The Real Deal)

Here's what you actually get in the [repository](https://github.com/thenameisvikash/mysql_ndb_cluster.git) - and honestly, it's way more comprehensive than most production setups I've seen:

```bash
mysql_ndb_cluster/
├── 📁 config/                           # All configuration files
├── 📁 docs/                             # Comprehensive documentation
├── 📁 scripts/                          # Automation and utility scripts
├── 📄 docker-compose.yml                # Complete orchestration
├── 📄 MySQL_Cluster_DevOps_Guide.md     # 23KB of deep technical insights
├── 📄 High_Throughput_Architecture.md    # For high-performance setups
├── 📄 Optimized_Architecture.md         # Production optimization guide
├── 📄 Node_Groups_and_Split_Brain.md    # Architecture deep-dive
├── 📄 Arbitration_Configuration.md      # Split-brain prevention
├── 📄 Partitioning_Limitations.md       # NDB-specific constraints
├── 📄 Node_Group_Expansion.md           # Scaling strategies
├── 📄 Backup_and_Recovery.md            # Operational procedures
├── 📄 Troubleshooting_Guide.md          # Common issues and solutions
├── 📄 Upgrade_Procedures.md             # Maintenance and upgrades
├── 📄 MySQL_Cluster_Management_Guide.md # Day-to-day operations
├── 📄 MySQL_Cluster_Testing_Documentation.md # Testing strategies
├── 📄 production_ready.md               # Production deployment guide
├── 🐍 test-load.py                      # Python load testing script
├── 🧪 mysql_cluster_test_suite.sh       # Comprehensive test suite (23KB!)
├── 🎨 architecture.svg                  # Visual architecture diagram
├── 🌐 view_architecture.html            # Interactive architecture viewer
├── ⚙️ proxysql-setup.sh                # ProxySQL automation
├── ⚙️ management-config.ini             # Management node config
├── ⚙️ proxysql.cnf                      # ProxySQL configuration
├── 🗃️ init.sql                          # Database initialization
└── 📋 my.cnf                            # MySQL server configuration
```

**This isn't just a "hello world" setup** - it's a complete production-ready toolkit with:

* **DevOps guide** covering everything from basics to advanced concepts
    
* **Comprehensive test suite** (23KB of testing scripts!)
    
* **Architecture diagrams** you can actually view and understand
    
* **Production deployment guides** for real-world scenarios
    
* **Load testing tools** to validate your setup
    
* **Operational procedures** for backup, recovery, and upgrades
    

**Pro tip**: Start with the `README.md` for quick setup, then dive into `MySQL_Cluster_DevOps_Guide.md` for the complete picture. The `test-load.py` script is perfect for validating your setup can handle real traffic!

## What's Next?

This setup gives you a solid foundation, but there's always more to explore:

1. **Kubernetes Deployment**: Running NDB Cluster on Kubernetes
    
2. **Geographic Distribution**: Multi-datacenter setups
    
3. **Performance Tuning**: Optimizing for your specific workload
    
4. **Monitoring Integration**: Prometheus + Grafana dashboards
    

Just like I progressed from [basic Kafka concepts](https://devopswizard.hashnode.dev/apache-kafka-for-devops-from-confusion-to-confidence-a-beginners-tutorial) to [production-grade Ansible automation](https://devopswizard.hashnode.dev/step-by-step-guide-to-kafka-cluster-deployment-using-ansible), the next step here might be containerizing this entire setup with Kubernetes operators or adding comprehensive monitoring with Prometheus.

## Wrapping Up

MySQL NDB Cluster isn't the easiest technology to wrap your head around, but once you understand the core concepts – node groups, partitioning, and arbitration – it becomes a powerful tool for building truly scalable applications.

The setup in my repository gives you everything you need to start experimenting and learning. Fork it, break it, fix it, and make it your own. That's how we all learn.

If you found this helpful, give the [repository](https://github.com/thenameisvikash/mysql_ndb_cluster.git) a star, and let me know what you'd like to see next. I'm thinking about tackling Redis Cluster or maybe diving into Kubernetes operators.

**Remember**: The best way to learn distributed systems is to build them, break them, and figure out how to fix them. This setup gives you a safe playground to do exactly that.

---

*Got questions or found issues with the setup? Open an issue on GitHub or reach out. The DevOps community thrives on shared knowledge and helping each other navigate these complex technologies.*

**What's your experience with distributed databases? Have you run into the MySQL user synchronization gotcha like I did? Drop a comment below – I'd love to hear about your challenges and solutions!**

**Questions for the community:**

* How do you handle user management in your MySQL Cluster setups?
    
* What's been your biggest "gotcha" moment with distributed databases?
    
* Are you using ProxySQL or managing connections differently?