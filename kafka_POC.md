
# Kafka POC for Uptycs Architecture – SRE Transition

## Introduction

Apache Kafka is a distributed event streaming platform used to build real-time data pipelines and streaming applications. It is designed to be fast, scalable, fault-tolerant, and durable.

### What is Kafka?
Kafka acts as a **publish-subscribe messaging system**, where:
- **Producer**: Sends messages to topics  
- **Consumer**: Subscribes to topics and reads messages  
- **Broker**: Kafka server that handles reads/writes  
- **Topic**: Logical channel of messages, split into partitions  
- **Partition**: Ordered, immutable sequence of messages  
- **Consumer Group**: Set of consumers sharing a subscription load  
- **Offset**: Position of a message in a partition  


### Why Kafka in Uptycs Architecture?

In Uptycs, Kafka is used to:
- Decouple ingestion components from processing pipelines.
- Handle high-throughput event data (agent logs, telemetry, audit data).
- Buffer and queue incoming data before it's consumed by analytic systems like **Rules Engine**, **Presto**, or **Spark**.
- Act as a fault-tolerant transport layer between producers (agents, ingestion services) and consumers (rules, alerting, analytics, etc).

Kafka nodes are typically deployed as part of the **streaming layer**.

---

## Kafka POC – Local Setup (Single Node)

1. Download and Install Kafka
    ```bash
    wget https://downloads.apache.org/kafka/3.6.0/kafka_2.13-3.6.0.tgz
    tar -xzf kafka_2.13-3.6.0.tgz
    cd kafka_2.13-3.6.0
    ```
2. Configure Brokers:
    
    ```
    #config/server1.properties
    brokerid=1
    port=9092
    log.dir=/tmp/kafka-logs-1
    zookeeper.connect=localhost:2181
    
    ```
    *More info:* [Config Params](http://kafka.apache.org/08/configuration.html)
   
4. Start Zookeeper (if not running):
    
    ```shell
    bin/zookeeper-server-start.sh config/zookeeper.properties
    ```
5. Start the brokers in seperate shells:
    
    ```shell
    env JMX_PORT=9999  bin/kafka-server-start.sh config/server1.properties
    ```
6. Create a kafka topic (with replication factor of 3):

    ```shell
    bin/kafka-create-topic.sh --topic mytopic --replica 1 --zookeeper localhost:2181
    ```
7. Send test messages (producer):
    
    ```shell
    bin/kafka-console-producer.sh --broker-list localhost:9092 --sync --topic mytopic
    ```
8. Start a consumer and recieve data:
    
    ```shell
    bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic mytopic --from-beginning
    ```
    Note: `--from-beginning` will read data from entire topic

## Kafka Threat Analyzer ##

This proof of concept demonstrates how to use Apache Kafka to stream simulated threat logs and analyze them using a custom Python script. It mimics how Uptycs might consume logs from Kafka and analyze them using EventSQL or AlertSQL-like logic.

Install the required Python package:

```bash
pip3 install kafka-python
```

### Step 1: Create Kafka Topic

```bash
bin/kafka-topics.sh --create --topic threat-events --bootstrap-server localhost:9092
```

### Step 2: Simulate Threat Logs via Kafka Producer

Start the Kafka console producer:

```bash
sudo bin/kafka-console-producer.sh --broker-list localhost:9092 --topic threat-events
```

Paste these sample JSON logs:

```json
{"host": "endpoint01", "threat": "malware.exe", "severity": "high"}
{"host": "endpoint02", "threat": "unknown_process", "severity": "low"}
{"host": "endpoint03", "threat": "exploit_attempt", "severity": "critical"}
{"host": "endpoint04", "threat": "brute_force_attempt", "severity": "critical"}
```

Press `Ctrl+D` to exit the producer.

### Step 3: Verify Kafka Received the Logs

```bash
sudo bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic threat-events --from-beginning
```

You should see the same logs replayed in the console.

### Step 4: Create the Python Analyzer Script

Save the following as `threat_analyzer.py`:

```python
from kafka import KafkaConsumer
import json

# Kafka consumer initialization
consumer = KafkaConsumer(
    'threat-events',
    bootstrap_servers='localhost:9092',
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    group_id='threat-analyzer-group',
    value_deserializer=lambda m: m.decode('utf-8')
)

print("[+] Listening for threat logs...\n")

for message in consumer:
    try:
        data = json.loads(message.value)

        host = data.get("host", "Unknown")
        threat = data.get("threat", "Unknown")
        severity = data.get("severity", "Unknown").lower()

        if severity == "high":
            print(f"[ALERT] Host: {host} | Threat: {threat} | Severity: HIGH")
        elif severity == "critical":
            print(f"[ALERT] Host: {host} | Threat: {threat} | Severity: CRITICAL")
        else:
            print(f"[DEBUG] Host: {host} | Threat: {threat} | Severity: {severity}")

    except json.JSONDecodeError:
        print(f"[ERROR] Received malformed JSON: {message.value}")
```

### Step 5: Run the Threat Analyzer

```bash
sudo python3 threat_analyzer.py
```

You should see alerts like:

```text
[ALERT] Host: endpoint01 | Threat: malware.exe | Severity: HIGH
[DEBUG] Host: endpoint02 | Threat: unknown_process | Severity: low
[ALERT] Host: endpoint03 | Threat: exploit_attempt | Severity: CRITICAL
[ALERT] Host: endpoint04 | Threat: brute_force_attempt | Severity: CRITICAL
```

Malformed JSONs will be caught:

```text
[ERROR] Received malformed JSON: 
```

This demonstrates that Python can successfully read messages from Kafka topics. A practical example would be implementing rule-based matching, where specific tags or keywords are defined in Python to detect relevant events. Once a match is found, the data can either be published to a new Kafka topic (e.g., matched_hits), routed to a storage destination such as an S3 bucket or HDFS, or sent to another processing endpoint.

---

Next POC:  Simulate Query Flow with Presto

Where: Data Flows to Presto

Test: Output Kafka data to a CSV/Parquet file → ingest into Presto or SQLite → run queries

