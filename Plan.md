#  Uptycs Architecture – POC Plan by Component

## 🎯 Goal
Understand and validate how each feature works across the Uptycs architecture by simulating real-world behaviors, configurations, and data flows in a controlled test environment (or staging replica).

---

## 🧩 Suggested Execution Plan

| Phase | Description                               | Duration | Owner     |
|-------|-------------------------------------------|----------|-----------|
| 1     | Prepare lab environment (test VMs, creds) | 1 week   | Infra/SRE |
| 2     | POC NGINX, Kafka                          | 1 week   | Platform  |
| 3     | POC Consumers, Rules, and Alerts          | 1–2 weeks| Security  |
| 4     | POC Presto/Spark + File Store             | 1 week   | Data Team |
| 5     | POC Databases + Monitoring                | 1 week   | Ops/Infra |
| 6     | Debrief + Document Findings               | 3 days   | All       |

---

## 🗂️ Component POC Breakdown

### 1. **NGINX Nodes**
**Objective:**  
Validate how ingress traffic is routed, terminated, and load balanced.

**POC Tasks:**
- Deploy NGINX with dummy TLS certs and simulate agent/UI/API traffic.
- Observe how requests are routed to frontend pods (e.g., `/login`, `/api/v1/queries`).
- Test health checks and failure handling.

**Success Criteria:**
- All simulated traffic reaches target services correctly.
- NGINX logs show accurate reverse proxy behavior.

---

### 2. **Kafka Cluster**
**Objective:**  
Understand telemetry flow from producer (agent) to consumers.

**POC Tasks:**
- Simulate a Kafka topic called `telemetry.process_events`.
- Push mock agent events into Kafka.
- Observe messages being consumed by mock Kafka consumer or `kafka-console-consumer`.

**Success Criteria:**
- Producers and consumers are decoupled and operate asynchronously.
- Kafka topics reflect partitioning and retention configs.

---

### 3. **Processing / Kafka Consumer Nodes**
**Objective:**  
Understand how telemetry is processed, enriched, and converted into alerts/incidents.

**POC Tasks:**
- Run `go-kafka-consumer` or similar mock service to consume a Kafka topic.
- Emulate logic for a `rule-engine` or `evaluator` using sample threat detection rules.
- Test batch jobs like `query-runner` or `alertSQL`.

**Success Criteria:**
- Rule logic is triggered on ingested telemetry.
- Output matches expected alert/enrichment schema.

---

### 4. **Frontend Kubernetes Services**
**Objective:**  
Validate user interactions, API endpoints, and SSO flows.

**POC Tasks:**
- Deploy mock versions of `login`, `UI`, and `API` pods.
- Test API endpoints (e.g., POST `/queries`, GET `/assets`).
- Simulate SSO flow via OAuth2 with fake identity provider.

**Success Criteria:**
- Authenticated users can submit queries and view results.
- API returns expected data structures.

---

### 5. **Spark / Presto Clusters**
**Objective:**  
Validate large-scale processing and query execution.

**POC Tasks:**
- Run sample SQL queries via Presto CLI against mock data (e.g., JSON logs in HDFS).
- Submit a batch job to Spark using telemetry data (e.g., file event count by host).
- Benchmark query time and memory usage.

**Success Criteria:**
- Queries return correct results over mock datasets.
- Presto and Spark can access file store or local HDFS.

---

### 6. **File Store (HDFS / S3)**
**Objective:**  
Understand storage structure, query integration, and retention.

**POC Tasks:**
- Simulate a directory hierarchy (e.g., `/telemetry/year/month/day/`).
- Ingest sample log data.
- Query it via Presto/Spark.

**Success Criteria:**
- File layout supports partition-based access.
- Presto/Spark can retrieve and filter data efficiently.

---

### 7. **Databases (Postgres / Redis / Mongo)**
**Objective:**  
Understand schema structure and usage of each DB type.

**POC Tasks:**
- Connect to `postgresql` and inspect tables (e.g., `rules`, `user_sessions`, `assets`).
- Use Redis CLI to inspect ephemeral keys (e.g., `agent:state:hostname123`).
- Load mock JSON data into MongoDB and simulate search.

**Success Criteria:**
- Each DB serves its respective function clearly (relational, ephemeral, document-based).
- Queries respond quickly and reflect expected data shape.

---

### 8. **Monitoring Stack**
**Objective:**  
Test observability and alerting for each node/service.

**POC Tasks:**
- Set up `node-exporter` and `prometheus`.
- Scrape metrics from key components (e.g., Kafka, NGINX, Spark).
- Build a Grafana dashboard (e.g., CPU by role, job status heatmap).

**Success Criteria:**
- Metrics are ingested, displayed, and alertable.
- Dashboards are intuitive and reflect real health status.

---
