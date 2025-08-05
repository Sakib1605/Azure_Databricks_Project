# 🚀 Azure Databricks End-to-End Project

This project demonstrates the implementation of a complete, production-ready data engineering pipeline using **Azure Databricks**, based on the **Medallion Architecture** and incorporating real-time ingestion, data transformation, governance, and automation features.

---

## 🏗️ Architecture Overview

The pipeline is built using the **Medallion Architecture**, which organizes data into three progressive layers stored in **Azure Data Lake Storage Gen2**:


- **Bronze**: Raw ingestion using Auto Loader or batch
- **Silver**: Cleaned and enriched Delta tables
- **Gold**: Aggregated data for analytics and reporting

---

## 🧱 Components Used

| Layer                  | Tool / Service                                  |
|------------------------|--------------------------------------------------|
| Storage                | Azure Data Lake Storage Gen2 (ADLS)              |
| Compute                | Azure Databricks                                 |
| Ingestion              | Databricks Auto Loader, Spark Structured Streaming |
| Data Format            | Delta Lake                                       |
| Governance             | Unity Catalog                                    |
| Scheduling             | Databricks Jobs / Workflows                      |
| Query & BI             | SQL Warehouses, Databricks SQL                   |
| Programming Languages  | PySpark, SQL                                     |

---

## ✅ Features Implemented

### 🔹 Cloud Infrastructure
- Created Azure Resource Group
- Provisioned ADLS Gen2 with containers: `bronze`, `silver`, `gold`

### 🔹 Unity Catalog & Governance
- Set up Unity Catalog metastore
- Defined external locations for each container
- Created catalogs, schemas, and tables
- Applied fine-grained access control with GRANT/REVOKE

### 🔹 Data Ingestion
- Ingested raw files (CSV, JSON) from ADLS using Auto Loader
- Configured schema inference, checkpointing, and file notification
- Supported batch & streaming ingestion

### 🔹 Data Transformation
- Applied cleansing, validation, and joins using PySpark
- Wrote transformed outputs to Delta tables (`silver`, `gold`)

### 🔹 Real-Time Streaming
- Built streaming pipelines using `readStream` / `writeStream`
- Handled schema evolution and fault tolerance

### 🔹 Workflow Orchestration
- Used Databricks Workflows (Jobs) to automate notebook execution
- Created task dependencies and scheduled daily ETL runs
- Passed parameters dynamically between tasks

### 🔹 BI & SQL Layer
- Created SQL Warehouses for querying curated tables
- Enabled ad hoc analysis and dashboard integration

---

## 📊 Sample Flow Diagram

        ┌─────────────┐
        │ Source Data │
        └─────┬───────┘
              │
        (Auto Loader / Batch)
              ↓
        ┌────────────┐
        │  Bronze    │
        │  (Raw)     │
        └────┬───────┘
             │ PySpark ETL
        ┌────▼───────┐
        │  Silver    │
        │ (Cleaned)  │
        └────┬───────┘
             │ Aggregation
        ┌────▼───────┐
        │   Gold     │
        │ (Analytics)│
        └────────────┘
