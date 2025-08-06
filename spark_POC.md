
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
wget https://dlcdn.apache.org/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
tar -xzf spark-3.5.1-bin-hadoop3.tgz
mv spark-3.5.1-bin-hadoop3 spark
```

### Step 3: Set Environment Variables

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
export SPARK_HOME=~/spark
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

## Summary

Apache Spark is an incredibly powerful tool for large-scale data processing. In the context of Uptycs, it can help transform, aggregate, and analyze telemetry logs at scale.

This POC demonstrates how Spark can:
- Simulate a telemetry dataset
- Execute distributed queries using Spark SQL
- Deliver aggregated insights for threat severity analytics

Future enhancements can include:
- Real-time streaming with Kafka
- Output to persistent storage like S3 or Parquet
- Use MLlib for pattern or anomaly detection
