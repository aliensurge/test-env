
# Spark POC for Uptycs Architecture

## What is Apache Spark?

Apache Spark is an open-source distributed processing system used for big data workloads. It provides a unified analytics engine for large-scale data processing with built-in modules for SQL, streaming, machine learning, and graph processing.

---

## Why Use Spark?

- **Speed**: Spark processes data in-memory, making it significantly faster than traditional MapReduce.
- **Ease of Use**: Offers high-level APIs in Java, Scala, Python, and R.
- **Versatility**: Supports SQL queries, streaming data, machine learning, and graph processing.
- **Integration**: Works well with Hadoop, Kafka, Hive, Cassandra, and more.

---

## Role of Spark in Uptycs Architecture

In the Uptycs architecture, Spark can be used to:

- Perform distributed processing on large datasets (e.g., agent logs or telemetry).
- Power advanced analytics jobs, such as behavior analytics or anomaly detection.
- Process real-time data streams when combined with Spark Streaming and Kafka.
- Batch-transform incoming telemetry before insertion into long-term storage or Presto-accessible databases.

---

## Setup Instructions

### Step 1: Install Java

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version
```

### Step 2: Download and Install Apache Spark

```bash
wget https://dlcdn.apache.org/spark/spark-3.5.6/spark-3.5.6-bin-hadoop3.tgz
tar -xzf spark-3.5.6-bin-hadoop3.tgz
mv spark-3.5.6-bin-hadoop3.tgz spark
```

### Step 3: Set Environment Variables

Add to  `~/.bashrc`

```bash
export SPARK_HOME=<installation path>/spark
export PATH=$SPARK_HOME/bin:$PATH
```

Apply the changes:

```bash
source ~/.bashrc
```

### Step 4: Verify Installation

```bash
spark-shell
```

You should see the Spark shell prompt indicating successful setup.

---

## Run a Sample Spark Job (Python)

### Step 5: Write a Sample Spark Job

sample_spark_job.py:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("SampleThreatCountJob") \
    .getOrCreate()

# Simulate telemetry logs
data = [
    ("endpoint01", "malware.exe", "high"),
    ("endpoint02", "unknown_process", "low"),
    ("endpoint03", "exploit_attempt", "critical"),
    ("endpoint04", "brute_force_attempt", "critical"),
    ("endpoint05", "malware.exe", "high"),
]

columns = ["host", "threat", "severity"]

df = spark.createDataFrame(data, columns)
df.createOrReplaceTempView("threat_logs")

# Run SQL query
result = spark.sql("""
    SELECT severity, COUNT(*) as count
    FROM threat_logs
    GROUP BY severity
    ORDER BY count DESC
""")

result.show()
```

### Step 6: Run the Job

```bash
spark-submit sample_spark_job.py
```

Expected output:

```text
+---------+-----+
| severity|count|
+---------+-----+
|    high |    2|
| critical|    2|
|     low |    1|
+---------+-----+
```

---

# Real-Time Threat Scoring Engine (Spark + Kafka)

This proof-of-concept demonstrates how Apache Spark can be used to consume real-time logs from a Kafka topic, apply threat scoring logic, and write enriched logs to another Kafka topic. This extends the previous Kafka POC and simulates how Uptycs might score or prioritize threats in-flight before further analysis.

---

## Use Case

- Kafka Topic: `threat-events`
- Incoming Fields: `host`, `threat`, `severity`
- Spark Logic:
  - Assigns scores based on severity
  - Creates a new Kafka topic: `threat-scores`
- Output Fields: `host`, `threat`, `severity`, `score`

---

## Prerequisites

- Apache Kafka (running locally)
- Apache Spark (3.0+ with Kafka support)
- Python 3 with `pyspark`

Install dependencies:

```bash
pip install pyspark
```

---

## Sample Kafka Threat Logs

Kafka Topic: `threat-events`

```json
{"host": "endpoint01", "threat": "malware.exe", "severity": "high"}
{"host": "endpoint02", "threat": "unknown_process", "severity": "low"}
{"host": "endpoint03", "threat": "exploit_attempt", "severity": "critical"}
{"host": "endpoint04", "threat": "brute_force_attempt", "severity": "critical"}
```

---

### Step 1: Start Kafka and Create Topics

```bash
# Start Kafka and Zookeeper as before
bin/kafka-topics.sh --create --topic threat-events --bootstrap-server localhost:9092
bin/kafka-topics.sh --create --topic threat-scores --bootstrap-server localhost:9092
```

---

### Step 2: Simulate Kafka Logs (Producer)

```bash
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic threat-events
```

Paste the sample logs above. Press `Ctrl+D` when done.

---

### Step 3: Create Spark Streaming Script

Save as `spark_threat_scoring.py`:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, when, from_json
from pyspark.sql.types import StructType, StringType

# Define schema
schema = StructType() \
    .add("host", StringType()) \
    .add("threat", StringType()) \
    .add("severity", StringType())

# Initialize Spark
spark = SparkSession.builder \
    .appName("ThreatScoring") \
    .getOrCreate()

# Read from Kafka
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "threat-events") \
    .load()

# Parse and transform
value_df = df.selectExpr("CAST(value AS STRING)") \
    .withColumn("json", from_json("value", schema)) \
    .select("json.*")

# Add scoring logic
scored_df = value_df.withColumn(
    "score",
    when(col("severity") == "critical", 10)
    .when(col("severity") == "high", 7)
    .when(col("severity") == "medium", 4)
    .otherwise(1)
)

# Write to another Kafka topic
query = scored_df.selectExpr("to_json(struct(*)) AS value") \
    .writeStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("topic", "threat-scores") \
    .option("checkpointLocation", "/tmp/threat-checkpoint") \
    .start()

query.awaitTermination()
```

---

### Step 4: Run the Scoring Script

```bash
python3 spark_threat_scoring.py
```

---

### Step 5: View Enriched Logs in Kafka

```bash
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic threat-scores --from-beginning
```

Sample output:

```json
{"host":"endpoint01","threat":"malware.exe","severity":"high","score":7}
{"host":"endpoint02","threat":"unknown_process","severity":"low","score":1}
{"host":"endpoint03","threat":"exploit_attempt","severity":"critical","score":10}
{"host":"endpoint04","threat":"brute_force_attempt","severity":"critical","score":10}
```

---

This POC simulates how a security analytics platform like Uptycs could apply real-time scoring logic to streaming threat data. Apache Spark acts as a stream processor that consumes logs from Kafka, enriches them with logic, and produces actionable data back into the Kafka pipeline

