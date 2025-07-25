
# Kafka POC for Uptycs Architecture – SRE Transition

## Introduction

Apache Kafka is a distributed event streaming platform used to build real-time data pipelines and streaming applications. It is designed to be fast, scalable, fault-tolerant, and durable.

### What is Kafka?
Kafka acts as a **publish-subscribe messaging system**, where:
- **Producers** publish data to **topics**.
- **Consumers** subscribe to those topics to read data.
- Kafka **brokers** handle the storage and distribution of these messages.
- **Zookeeper** coordinates and manages broker state.

### Why Kafka in Uptycs Architecture?

In Uptycs, Kafka is used to:
- Decouple ingestion components from processing pipelines.
- Handle high-throughput event data (agent logs, telemetry, audit data).
- Buffer and queue incoming data before it's consumed by analytic systems like **Rules Engine**, **Presto**, or **Spark**.
- Act as a fault-tolerant transport layer between producers (agents, ingestion services) and consumers (rules, alerting, analytics, etc).

Kafka nodes are typically deployed as part of the **streaming layer**.

---

## Kafka POC – Local Setup (Single Node)

### Step 1: Install Java

Kafka requires Java 8 or higher.

Check:
```bash
java -version
