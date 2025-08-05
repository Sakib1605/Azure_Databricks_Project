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

## 🧪 Step-by-Step Implementation

### 1. **Cloud Infrastructure Setup**
- Created a **Resource Group** in Azure
- Provisioned **ADLS Gen2** with Hierarchical Namespace enabled
- Created containers: `/bronze`, `/silver`, `/gold`

### 2. 🚀 Databricks Workspace Configuration

To enable a secure and scalable compute environment, I deployed an **Azure Databricks Workspace** in the same region as the ADLS Gen2 storage account. This ensured optimal performance and compliance with regional data policies. The workspace acts as the central development and processing hub for the entire data pipeline.

#### ✅ a. Created a Databricks Access Connector
- Provisioned an **Azure Databricks Access Connector** using the Azure Portal.
- This connector acts as a secure identity bridge between Databricks and other Azure services (e.g., ADLS Gen2).
- It uses **Managed Identity**, eliminating the need to store secrets or keys in notebooks.

#### ✅ b. Assigned IAM Role for Storage Access
- Granted the Access Connector the `Storage Blob Data Contributor` role via Azure **Role-Based Access Control (RBAC)**.
- The role was scoped specifically to the ADLS Gen2 storage account used in the project.
- This gave Databricks compute clusters the ability to read/write to the `/bronze`, `/silver`, and `/gold` containers.

#### ✅ c. Enabled Credential Passthrough for Secure Access
- **Credential passthrough** was enabled on user clusters, allowing Azure Active Directory identities to be used for data access.
- Users interact with ADLS Gen2 using their **own AAD credentials**, ensuring fine-grained data access and complete audit trails.
- This was configured by:
  - Using **Single User access mode** on clusters
  - Setting the Spark config:  
    ```bash
    spark.databricks.passthrough.enabled true
    ```

> 🔐 **Why this matters**:  
> Credential passthrough enables enterprise-level security and governance by removing hardcoded credentials, adhering to the principle of least privilege, and allowing access control to be managed centrally in Azure AD.

---

**✅ Result**:  
The Databricks workspace is now securely connected to the Data Lake, with cluster-level and user-level access controls in place. All ingestion, transformation, and streaming operations are executed under governed and auditable conditions.



### 3. **Unity Catalog & Governance**
- Created a **Unity Catalog metastore** and attached it to the workspace
- Defined **external locations** referencing ADLS paths
- Created **catalogs**, **schemas**, and registered **external & managed tables**
- Assigned roles and privileges via SQL (GRANT/REVOKE)

### 4. **Data Ingestion with Auto Loader**
- Used **Databricks Auto Loader** to detect and ingest new files into `/bronze`
- Defined:
  - `cloudFiles.format = "csv"`
  - `schemaLocation` and `checkpointLocation`
- Enabled **schema inference** and **streaming mode**

### 5. **Data Transformation (Bronze → Silver)**
- Created PySpark notebooks to clean raw data
  - Dropped nulls
  - Casted data types
  - Renamed columns
- Wrote results to **Delta tables** in `/silver`

### 6. **Data Aggregation (Silver → Gold)**
- Built aggregation logic for business-ready data
- Joined multiple silver tables
- Produced KPIs, summary stats, grouped aggregates
- Stored results in `/gold` Delta tables

### 7. **Real-Time Processing with Structured Streaming**
- Implemented `readStream` and `writeStream` pipelines
- Configured append/update triggers
- Used checkpointing to ensure fault tolerance

### 8. **SQL Analytics Layer**
- Launched a **SQL Warehouse** in Databricks
- Queried Delta tables using SQL Editor
- Created dashboards and explored datasets interactively

### 9. **ETL Orchestration with Databricks Workflows**
- Used **Jobs UI** to automate pipeline:
  - Task 1: Bronze ingestion
  - Task 2: Bronze → Silver transformation
  - Task 3: Silver → Gold aggregation
- Defined dependencies, retries, parameters
- Scheduled job to run daily

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
