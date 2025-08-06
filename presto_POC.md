# Presto POC: Distributed SQL Engine 

## What is Presto?

[Presto](https://prestodb.io/) is an open-source, distributed SQL query engine designed for fast analytical queries over large-scale datasets. It supports querying data from multiple sources (like Hive, MySQL, PostgreSQL, S3, Kafka, and more) using standard ANSI SQL.

Originally developed by Facebook, Presto is optimized for interactive query workloads and is capable of querying data from multiple sources in a single query — making it ideal for modern data platforms.

---

## Why Use Presto?

- **Fast & Interactive**: Low-latency, high-throughput SQL engine
- **Federated Queries**: Query across multiple sources (e.g., Kafka + S3 + MySQL)
- **Scalable**: Can scale from a single node to hundreds
- **Extensible**: Wide range of connectors (Kafka, Elasticsearch, HDFS, Prometheus, etc.)

---

## Role of Presto in Uptycs Architecture

Presto can serve as a flexible query layer over structured and semi-structured data flowing through the pipeline.

Possible usages:
- Query real-time threat events from **Kafka**
- Join data from **PostgreSQL** and **Kafka**
- Execute SQL-based rules for threat correlation (similar to Uptycs AlertSQL/EventSQL)
- Enable ad-hoc search for threat hunters or analysts without exposing the raw Kafka consumer layer

---

## Step-by-Step Installation

### Step 1: Install Java

Check if Java is installed:

```bash
java -version
```

If not, install OpenJDK:

```bash
sudo yum install java-1.8.0-openjdk
```

---

### Step 2: Download and Extract Presto

```bash
wget https://repo1.maven.org/maven2/io/prestosql/presto-server/341/presto-server-341.tar.gz
tar -xvzf presto-server-341.tar.gz
mv presto-server-341 presto
cd presto
```

---

### Step 3: Configure Presto

Create necessary directories:

```bash
mkdir -p etc/catalog
```

Create `etc/node.properties`:

```properties
node.environment=presto-dev
node.id=presto-coordinator
node.data-dir=/tmp/presto/data
```

Create `etc/jvm.config`:

```text
-server
-Xmx1G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-Djava.awt.headless=true
```

Create `etc/config.properties`:

```properties
coordinator=true
node-scheduler.include-coordinator=true
http-server.http.port=8080
query.max-memory=512MB
query.max-memory-per-node=128MB
discovery-server.enabled=true
discovery.uri=http://localhost:8080
```

Create `etc/log.properties`:

```properties
com.facebook.presto=INFO
```

---

### Step 4: Add Kafka Connector

Create file `etc/catalog/kafka.properties`:

```properties
connector.name=kafka
kafka.nodes=localhost:9092
kafka.default-schema=default
kafka.table-names=threat-events
kafka.hide-internal-columns=false
```

Ensure Kafka is running on `localhost:9092`. You can test using:

```bash
netstat -plnt | grep 9092
```

If Kafka is not running, start it:

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties
```

---

### Step 5: Create Kafka Topic and Send Messages

In a separate shell, create the topic:

```bash
bin/kafka-topics.sh --create --topic threat-events --bootstrap-server localhost:9092
```

Send sample logs:

```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic threat-events
```

Paste in sample messages:

```json
{"host": "endpoint01", "threat": "malware.exe", "severity": "high"}
{"host": "endpoint02", "threat": "unknown_process", "severity": "low"}
{"host": "endpoint03", "threat": "exploit_attempt", "severity": "critical"}
{"host": "endpoint04", "threat": "brute_force_attempt", "severity": "critical"}
```

Press `Ctrl+D` when done.

---

### Step 6: Start Presto CLI

Download the Presto CLI:

```bash
wget https://repo1.maven.org/maven2/io/prestosql/presto-cli/341/presto-cli-341-executable.jar
mv presto-cli-341-executable.jar presto
chmod +x presto
```

Run it:

```bash
./presto --server localhost:8080 --catalog kafka --schema default
```
<img width="949" height="467" alt="image" src="https://github.com/user-attachments/assets/4ed37080-bcda-4b04-9140-b1871b55c945" />

---

### Step 7: Validate Connection to Kafka

Inside the Presto CLI:

Check tables:

```sql
SHOW TABLES;
```


Check Kafka schema:

```sql
DESCRIBE "threat-events";
```

<img width="948" height="246" alt="image" src="https://github.com/user-attachments/assets/1966f23c-5f86-4322-95a5-e03d097e2eaa" />



If Presto shows errors like `No nodes available to run query` or `No mapping for host`, it usually indicates either:
- Kafka was not running
- The topic wasn't created
- Presto catalog was misconfigured

**Resolution steps taken**:
- Verified Kafka topic was created
- Confirmed messages exist using Kafka consumer:
  ```bash
  bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic threat-events --from-beginning
  ```
- Ran CLI again and validated `_message` field separately

---

### Step 8: Run a Query

```sql
SELECT _message FROM "threat-events" LIMIT 10;
```

```json
{"host": "endpoint01", "threat": "malware.exe", "severity": "high"}
{"host": "endpoint02", "threat": "unknown_process", "severity": "low"}
{"host": "endpoint03", "threat": "exploit_attempt", "severity": "critical"}
{"host": "endpoint04", "threat": "brute_force_attempt", "severity": "critical"}
```

<img width="944" height="360" alt="image" src="https://github.com/user-attachments/assets/d3b488c4-f1c2-449b-971e-1f3e758bb8c6" />


This POC shows how Presto can be used to query real-time data from Kafka using simple SQL. It highlights how powerful Presto is when it comes to exploring streaming data without needing to write custom consumers or ETL jobs.

By connecting Presto to Kafka, we were able to stream simulated threat logs and run SQL queries directly on them — just like you would with any database table. This demonstrates how Presto can play a key role in threat analysis by letting analysts quickly search, filter, and investigate live data using familiar SQL commands.

In short, this setup proves that Presto can serve as a flexible, scalable query engine for real-time security data — making it easier to investigate and respond to threats as they happen.
