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

## Phase 1: Standalone Presto Setup

In this phase, we install and configure Presto as a single-node (coordinator-only) server. This will lay the groundwork before connecting it to Kafka or other data sources.

---

## Prerequisites

- Java 8 or higher (OpenJDK recommended)
- Linux/macOS
- Python 3 (optional, for tooling/scripts)
- Terminal access

---

## Step-by-Step Installation

### Step 1: Install Java

Check if Java is installed:

```bash
java -version
