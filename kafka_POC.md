
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
    
    #config/server2.properties
    brokerid=2
    port=9093
    log.dir=/tmp/kafka-logs-2
    
    #config/server3.properties
    brokerid=3
    port=9094
    log.dir=/tmp/kafka-logs-3

    ```
    *More info:* [Config Params](http://kafka.apache.org/08/configuration.html)
   
4. Start Zookeeper (if not running):
    
    ```shell
    bin/zookeeper-server-start.sh config/zookeeper.properties
    ```
5. Start the brokers in seperate shells:
    
    ```shell
    env JMX_PORT=9999  bin/kafka-server-start.sh config/server1.properties
    env JMX_PORT=10000 bin/kafka-server-start.sh config/server2.properties
    env JMX_PORT=10001 bin/kafka-server-start.sh config/server3.properties
    ```
6. Create a kafka topic (with replication factor of 3):

    ```shell
    bin/kafka-create-topic.sh --topic mytopic --replica 3 --zookeeper localhost:2181
    ```
7. Send test messages (producer):
    
    ```shell
    bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093,localhost:9094 --sync --topic mytopic
    ```
8. Start a consumer and recieve data:
    
    ```shell
    bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic mytopic --from-beginning
    ```
    Note: `--from-beginning` will read data from entire topic




