# Cassandra 3-Node Cluster Tutorial

This project provides a local development environment for learning **Apache Cassandra**. It spins up a 3-node peer-to-peer cluster using Docker Compose, optimized for local resource constraints.

---

## 🚀 Quick Start

### 1. Prerequisites
* **Docker** and **Docker Compose** installed.
* **Resources:** Ensure Docker is allocated at least **4GB of RAM** (Settings > Resources > Memory).

### 2. Startup Sequence
Cassandra is sensitive to startup timing. To avoid "Token Collisions" where nodes fight for the same spot on the virtual ring, follow this manual sequence:

```bash
# 1. Clean up any old data and volumes
docker-compose down -v

# 2. Start the Seed node
docker-compose up -d cassandra-1

# 3. Wait 45 seconds for cassandra-1 to initialize
# Then start the follower nodes one by one
docker-compose up -d cassandra-2
sleep 30
docker-compose up -d cassandra-3
```

### 3. Verify Cluster Health
Run the following command to ensure all nodes are in the UN (Up/Normal) state:
```bash
docker exec -it cassandra-1 nodetool status
```

## 🛠 Project Configuration
Architecture Notes
Replication Factor (RF): Configured for an RF of 3 (data is copied to every node for high availability).

Memory Limits: Each node is limited to 512MB Heap (MAX_HEAP_SIZE) to prevent system crashes (Exit Code 137).

Token Management: CASSANDRA_NUM_TOKENS is set to 1 to simplify the ring architecture and reduce gossip overhead during local startup.


## 📖 Hands-On Tutorial
### Step 1: Access the Shell
Connect to the Cassandra Query Language (CQL) shell on the seed node:
```
docker exec -it cassandra-1 cqlsh
```

### Step 2: Create a Schema
Cassandra requires a Keyspace to define replication strategy before creating tables.

```sql
-- Create a Keyspace (similar to a Database)
CREATE KEYSPACE tutorial_keyspace 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 3};

USE tutorial_keyspace;

-- Create a Table (Optimized for Querying by User ID)
CREATE TABLE users (
    user_id uuid PRIMARY KEY,
    name text,
    email text,
    created_at timestamp
);
```

### Step 3: Insert and Query

```sql
INSERT INTO users (user_id, name, email, created_at) 
VALUES (uuid(), 'Gemini User', 'user@example.com', toTimestamp(now()));

SELECT * FROM users;
```

### ❓ Troubleshooting
Issue	Cause	Fix
Exit Code 137	Out of Memory (OOM)	Increase Docker Desktop RAM or decrease MAX_HEAP_SIZE in the yaml.
Token Collision	Nodes starting too fast	Use docker-compose down -v and start nodes one-by-one with delays.
Connection Refused	Node still booting	Cassandra takes ~60s to start. Check docker logs cassandra-1.

### 🧹 Cleanup
To stop the cluster and delete all data (including the databases you created):

```bash
docker-compose down -v
```