# 🚀 Azure Databricks End-to-End Project

This project demonstrates the implementation of a complete data engineering pipeline using **Azure Databricks**, based on the **Medallion Architecture** and incorporating real-time ingestion, data transformation, governance, and automation features.

---

## 🏗️ Architecture Overview

The pipeline is built using the **Medallion Architecture**, which organizes data into three progressive layers stored in **Azure Data Lake Storage Gen2**:


- **Bronze**: Raw ingestion using Auto Loader or batch
- **Silver**: Cleaned and enriched Delta tables
- **Gold**: Aggregated data for analytics and reporting

![Medallion Architecture](diagrams/medallion_architecture.png)

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

### 1.☁️ **Cloud Infrastructure Setup**

To support this Databricks project, I provisioned a scalable and secure cloud environment on **Microsoft Azure** using the following steps:

---

#### 📦 a. Resource Group

- Created a new **Azure Resource Group** to logically organize all resources related to the Databricks project.
- This includes storage accounts, Databricks workspace, networking, and access control resources.

---

#### 📁 b. Azure Data Lake Storage Gen2 (ADLS Gen2)

- Provisioned an **ADLS Gen2** storage account with:
  - **Hierarchical Namespace** enabled (crucial for directory-like structure and fine-grained file operations)
  - Enabled **Secure Transfer** and **RBAC**-based access

---

#### 🗂️ c. Containers Created

- Set up three logical containers to implement the **Medallion Architecture**:

  | Container | Purpose                            |
  |----------|------------------------------------|
  | `/bronze`| Ingest raw data using Auto Loader   |
  | `/silver`| Store cleaned and transformed data  |
  | `/gold`  | Store aggregated data ready for analytics and reporting |

---

#### 📦 d. Source Container for Raw Datasets

- Additionally created a `/source` container to upload raw input datasets.
- This acts as the **landing zone** for the following datasets:

  | Dataset      | Description                           |
  |--------------|---------------------------------------|
  | `customers`  | Customer information CSV/Parquet      |
  | `orders`     | Order records with timestamps         |
  | `products`   | Product catalog including categories  |

- These datasets are ingested into the `/bronze` layer dynamically using **Databricks Auto Loader** with parameterized workflows.

---
---

### 2. 🚀 Databricks Workspace Configuration

To enable a secure and scalable compute environment, I deployed an **Azure Databricks Workspace** in the same region as the ADLS Gen2 storage account. This ensured optimal performance and compliance with regional data policies. The workspace acts as the central development and processing hub for the entire data pipeline.

#### a. Created a Databricks Access Connector
- Provisioned an **Azure Databricks Access Connector** using the Azure Portal.
- This connector acts as a secure identity bridge between Databricks and other Azure services (e.g., ADLS Gen2).
- It uses **Managed Identity**, eliminating the need to store secrets or keys in notebooks.

####  b. Assigned IAM Role for Storage Access
- Granted the Access Connector the `Storage Blob Data Contributor` role via Azure **Role-Based Access Control (RBAC)**.
- The role was scoped specifically to the ADLS Gen2 storage account used in the project.
- This gave Databricks compute clusters the ability to read/write to the `/bronze`, `/silver`, and `/gold` containers.

####  c. Enabled Credential Passthrough for Secure Access
- **Credential passthrough** was enabled on user clusters, allowing Azure Active Directory identities to be used for data access.
- Users interact with ADLS Gen2 using their **own AAD credentials**, ensuring fine-grained data access and complete audit trails.
- This was configured by:
  - Using **Single User access mode** on clusters
  - Setting the Spark config:  
    ```bash
    spark.databricks.passthrough.enabled true
    ```

>  **Why this matters**:  
> Credential passthrough enables enterprise-level security and governance by removing hardcoded credentials, adhering to the principle of least privilege, and allowing access control to be managed centrally in Azure AD. The Databricks workspace is now securely connected to the Data Lake, with cluster-level and user-level access controls in place. All ingestion, transformation, and streaming operations are executed under governed and auditable conditions.

---
---

###  3. Unity Catalog & Governance (Configured via Databricks UI)

To ensure secure, centralized governance over all data assets in the pipeline, I configured **Unity Catalog** directly through the **Databricks UI**. This setup allowed me to organize data across layers, enforce access control, and integrate permissions with Azure Active Directory — all without writing SQL code.

####  What I Configured
- Created a **Unity Catalog Metastore** and attached it to the Databricks workspace
- Registered **external locations** pointing to ADLS Gen2 containers (`bronze`, `silver`, `gold`)
- Defined **catalogs** and **schemas** to organize data by zone and purpose
- Assigned access control to users and groups through UI-based **GRANT/REVOKE** settings

---

#### ⚙️ How I Did It (UI Workflow)

1. **Created the Metastore**
   - Opened the **Databricks Account Console**
   - Navigated to the **Data** section → Clicked **Create Metastore**
   - Chose the correct Azure region and assigned an ADLS Gen2 root path
   - Linked the metastore to my Databricks workspace

2. **Set Up Storage Credential & External Locations**
   - In the **Unity Catalog UI**, created a **Storage Credential** using an Azure Access Connector
   - Defined **External Locations** for each data layer by selecting the correct ADLS path (e.g., `abfss://bronze@...`)
   - Ensured secure, keyless access using role-based identity

3. **Defined Catalogs and Schemas**
   - Created a **Catalog** (e.g., `project_catalog`) to house all datasets
   - Added **Schemas** (e.g., `raw_data`, `processed_data`) to organize data zones
   - Tables were auto-tracked when written in Delta format or registered through the UI

4. **Configured Permissions**
   - Used the **Permissions tab** in Databricks to assign access
     - Granted `SELECT`, `USAGE`, and `MODIFY` rights to appropriate user groups
     - Followed least-privilege principles (e.g., read-only for analysts, full access for engineers)

---

####  Why It Matters

-  Unified access control across workspaces and catalogs
-  Secure integration with Azure Active Directory
-  Centralized metadata and table management
-  Fine-grained governance without writing SQL

This setup ensured that data was not only ingested and transformed securely, but also governed in a way that supports compliance, auditability, and collaboration across teams — all configured visually through Databricks’ intuitive interface.

---
---

### 4. **Data Ingestion with Auto Loader (Parameterized, Dynamic, and Workflow-Driven)**

To create a flexible and scalable ingestion solution, I implemented **Databricks Auto Loader** with dynamic path resolution and parameterization, orchestrated via **Databricks Workflows (Jobs)**. This allows ingestion of multiple datasets (e.g., `customers`, `orders`, `products`) using a single reusable notebook triggered through a job loop.

---

#### Objective

- Ingest multiple datasets into the **Bronze layer** using Auto Loader
- Parameterize the ingestion logic with dynamic values (e.g., file name)
- Enable orchestration and looping through **Databricks Jobs**
- Automate schema tracking and checkpointing
- Ensure fault-tolerance and scalability

---

#### 📘 Notebook 1: `AutoLoader_Ingestion.py` (Ingests a Single Dataset)

This notebook is designed to ingest one dataset at a time using `dbutils.widgets` to dynamically receive the dataset name (`file_name`) from a controller or job.

```python
# Step 1: Accept dynamic parameter
dbutils.widgets.text("file_name", " ")
file_name_parameter = dbutils.widgets.get("file_name")

# Step 2: Build dynamic paths
input_path       = f"abfss://source@databricksstrgeaccount.dfs.core.windows.net/{file_name_parameter}"
schema_location  = f"abfss://bronze@databricksstrgeaccount.dfs.core.windows.net/checkpoint_{file_name_parameter}"
output_path      = f"abfss://bronze@databricksstrgeaccount.dfs.core.windows.net/{file_name_parameter}"

# Step 3: Read from source using Auto Loader
df = (spark.readStream
      .format("cloudFiles")
      .option("cloudFiles.format", "parquet")
      .option("cloudFiles.schemaLocation", schema_location)
      .load(input_path))

# Step 4: Write to Bronze layer
df.writeStream
   .format("parquet")
   .outputMode("append")
   .option("checkpointLocation", schema_location)
   .option("path", output_path)
   .trigger(once=True)
   .start()
```

#### 🔁 Notebook 2: `Controller.py` (Prepares Dataset List)

This controller notebook defines the list of datasets and passes them as a task value to downstream job tasks.

```python
# Define dataset list for ingestion
datasets_array = [
    {"file_name": "customers"},
    {"file_name": "orders"},
    {"file_name": "products"}
]

# Set this list as a task value to be used in looping
dbutils.jobs.taskValues.set("dataset_output", datasets_array)
```

#### ⚙️ Workflow Configuration in Databricks Jobs

In the **Databricks Workflows (Jobs)** UI:

- **Task 1: `Controller`**
  - Runs the `Controller.py` notebook
  - Sets `dataset_output` as the task output

- **Task 2: `AutoLoader_Ingestion`**
  - Depends on Task 1
  - Configured with:
    - **Loop over array**: `dataset_output`
    - **Parameter**: `file_name = {{item.file_name}}`

- **Schedule & Alerts (Optional)**
  - Schedule the job to run **daily**, **weekly**, or **on-demand**
  - Enable **retry policies** and **failure alert notifications** as needed

---

#### 🧠 Execution Flow

```text
Databricks Workflow (Job)
├── Controller Notebook
│   └── Output: dataset_output = [customers, orders, products]
│
└── AutoLoader_Ingestion Notebook (Looped)
    ├── file_name = "customers"
    ├── file_name = "orders"
    └── file_name = "products"
```
---
---

### 5. **Data Transformation (Bronze → Silver)**
## 🥈 Silver Layer — Cleaned & Structured Delta Tables

The **Silver Layer** in this project serves as the **refined data zone** in the Medallion Architecture.  
Data from the **Bronze Layer** (raw ingestion) is read, optionally transformed, and stored as clean, structured **Delta tables** in **Azure Data Lake Storage Gen2 (ADLS Gen2)**.  
These Delta tables are then registered in **Unity Catalog** for governed access.

---

### 📂 Notebooks Implemented
- `silver_orders`
- `silver_customers`
- `silver_products`
- `silver_regions`


## Silver Layer Workflow (Bronze → Silver Transformation)

The Silver Layer refines Bronze Layer data into clean, query-ready Delta tables with Unity Catalog governance.

### Workflow Steps
1. **Read Bronze Data**  
   - Source: Parquet files stored in Bronze container (`orders`, `customers`, `products`, `regions`).  
   - Location: Azure Data Lake Storage Gen2.

2. **Apply Business Logic**  
   - `silver_orders`: Applied ranking functions (`dense_rank`, `rank`, `row_number`) to identify top-performing orders by year.  
   - `silver_customers`, `silver_products`, `silver_regions`: Direct promotion without transformation for faster processing.

3. **Write to Silver Layer**  
   - Format: Delta Lake for ACID transactions, schema enforcement, and time travel.  
   - Storage: Silver container in ADLS Gen2.

4. **Register in Unity Catalog**  
   - Created tables in `silver` schema (`orders_silver`, `customers_silver`, `products_silver`, `regions_silver`).  
   - Enables centralized governance, fine-grained permissions, and lineage tracking.

### Outcome
- Raw Bronze data → Cleaned, structured Silver Delta tables.
- Ready for downstream analytics, BI tools, and Gold Layer aggregation.


### 🔄 Processing Details

#### `silver_orders`
- **Source:** `/bronze/orders` (Parquet)  
- **Transformation:** Applied **window functions** (`dense_rank`, `rank`, `row_number`) using an **OOP class** for clean, reusable code design.
  - Partitioned by `year`
  - Ordered by `total_amount` in descending order
- **Output:** `/silver/orders` (Delta format)  
- **Catalog Entry:** `databricks_catalog.silver.orders_silver`  

#### `silver_customers`
- **Source:** `/bronze/customers` (Parquet)  
- **Transformation:** None (direct migration)  
- **Output:** `/silver/customers` (Delta format)  
- **Catalog Entry:** `databricks_catalog.silver.customers_silver`  

#### `silver_products`
- **Source:** `/bronze/products` (Parquet)  
- **Transformation:** None (direct migration)  
- **Output:** `/silver/products` (Delta format)  
- **Catalog Entry:** `databricks_catalog.silver.products_silver`  

#### `silver_regions`
- **Source:** `/bronze/regions` (Parquet)  
- **Transformation:** None (direct migration)  
- **Output:** `/silver/regions` (Delta format)  
- **Catalog Entry:** `databricks_catalog.silver.regions_silver`  

---

### 🧱 Example: `silver_orders` Implementation

```python
# Step 1: Read from Bronze
df = spark.read.format("parquet").load(
    "abfss://bronze@databricksstrgeaccount.dfs.core.windows.net/orders"
)

# Step 2: Apply window functions via OOP class
class windows:
    def dense_rank(self, df):
        return df.withColumn(
            "flag",
            dense_rank().over(Window.partitionBy("year").orderBy(desc("total_amount")))
        )

    def rank(self, df):
        return df.withColumn(
            "rank_flag",
            rank().over(Window.partitionBy("year").orderBy(desc("total_amount")))
        )

    def row_number(self, df):
        return df.withColumn(
            "rownum_flag",
            row_number().over(Window.partitionBy("year").orderBy(desc("total_amount")))
        )

# Step 3: Write to Silver in Delta format
df.write.format("delta").mode("overwrite").save(
    "abfss://silver@databricksstrgeaccount.dfs.core.windows.net/orders"
)

# Step 4: Register in Unity Catalog
%sql
CREATE TABLE IF NOT EXISTS databricks_catalog.silver.orders_silver
USING DELTA
LOCATION 'abfss://silver@databricksstrgeaccount.dfs.core.windows.net/orders';
```



---
---

## Gold Layer Workflow 

The Gold Layer represents the **business-ready, analytics-optimized** data in the Medallion Architecture.  
In this project, the `gold_customers` table implements **Slowly Changing Dimension Type 1 (SCD1)** logic to maintain up-to-date customer dimension records.

---

### Workflow Steps

1. **Read Source Data**  
   - Source: `silver.customers_silver` Delta table from the Silver Layer.  
   - Purpose: Ensure we work with clean, enriched customer data for dimension building.

2. **Identify Initial Load vs Incremental Load**  
   - Used `init_load_flag` parameter to determine load type:  
     - **Initial Load (`init_load_flag=1`)**: Creates an empty placeholder for old records.  
     - **Incremental Load (`init_load_flag=0`)**: Reads existing `DimCustomers` table from the Gold Layer for comparison.

3. **Compare New vs Existing Records**  
   - Performed **left join** on `customer_id` between current Silver data and existing Gold table.  
   - Segregated:  
     - **New Records** → Not present in Gold table.  
     - **Existing Records** → Already in Gold table, eligible for update.

4. **Prepare Existing Records**  
   - Retained existing surrogate keys (`DimCustomersKey`).  
   - Updated `update_date` to current timestamp.  
   - Preserved `create_date`.

5. **Prepare New Records**  
   - Dropped redundant columns from join output.  
   - Generated surrogate keys using `monotonically_increasing_id()` + offset from the max existing key in Gold table.  
   - Set both `create_date` and `update_date` to current timestamp.

6. **Merge Data (SCD Type 1 Logic)**  
   - If `DimCustomers` table exists:  
     - Used **Delta Lake `MERGE`** to **update matching records** and **insert new records**.  
   - If table does not exist:  
     - Created new Delta table in Gold Layer.

---

### Why SCD Type 1?  
- **Overwrites old values** with new data when changes occur.  
- Ideal for maintaining **only the latest** customer details, without history tracking.  
- Reduces storage needs compared to SCD Type 2.

---

### Outcome
- **Gold.DimCustomers** table with fully up-to-date customer records.
- Centralized in **Unity Catalog** for governance and accessible to BI tools like Power BI and Databricks SQL.
- Ensures consistent surrogate key management for future fact table joins.

  
---

## 🥇 Gold Layer — DimProducts (Delta Live Tables, SCD Type 2)

### 1. Overview
The `DimProducts` Gold Layer table is built using **Delta Live Tables (DLT)** to implement a fully automated, streaming-enabled **Slowly Changing Dimension Type 2 (SCD-2)** process.  
The pipeline consumes product records from the Silver Layer (`silver.products_silver`), enforces strict data quality checks, and maintains complete historical tracking of changes for analytical use cases.

---

### 2. Why Delta Live Tables?
**Delta Live Tables** is a declarative ETL framework in Databricks that simplifies data pipeline creation by:
- Automatically managing dependencies, streaming checkpoints, and orchestration.
- Enforcing **data quality** rules directly within the pipeline definition.
- Providing **end-to-end lineage** and operational monitoring in the Databricks UI.
- Supporting **SCD Type 1 & 2** operations without manually writing complex `MERGE` statements.
- Enabling both **batch** and **streaming** ingestion with the same code.

In this implementation, DLT was selected because:
- The source is a **streaming** Silver table.
- Historical product changes must be tracked in an SCD-2 format.
- Built-in quality enforcement is required before data is persisted in Gold.

---

### 3. Implementation Details

#### **Step 1 — Define Data Quality Rules**
A set of rules was declared to ensure:
- `product_id` is not null.
- `product_name` is not null.  
These rules are applied via `@dlt.expect_all_or_drop`, ensuring invalid records are **excluded** before reaching Gold.

#### **Step 2 — Create a Quality-Checked Staging Table**
A **DLT Table** `DimProducts_stage` was created to:
- Continuously read from `silver.products_silver` in streaming mode.
- Apply the defined data quality rules.
- Serve as a validated staging layer for downstream processing.

#### **Step 3 — Create an Intermediate Streaming View**
A **DLT View** `DimProducts_view` was created to:
- Read from `DimProducts_stage`.
- Provide a clean, quality-assured dataset for the SCD-2 load process.
- Maintain modularity between quality enforcement and historical tracking.

#### **Step 4 — Load the Gold Table with SCD Type 2 Logic**
The `dlt.apply_changes` function was used to:
- Target the Gold table `DimProducts`.
- Use `product_id` as the business key.
- Sequence records by `product_id` (in production, this is typically a timestamp field).
- Persist changes using **SCD Type 2** rules:
  - Insert new records with new surrogate keys.
  - Expire old versions when changes occur.
  - Retain the full change history for analytics.

---

### 4. Data Flow

```plaintext
Silver Layer: products_silver
        │
        ▼
DLT Staging Table (DimProducts_stage)
  │  - Apply quality rules
  │  - Streaming ingestion
        ▼
DLT View (DimProducts_view)
  │  - Decoupled transformation
        ▼
DLT SCD2 Apply Changes (DimProducts)
  │  - Maintain history
  │  - Track changes in product attributes
        ▼
Gold Layer: DimProducts (Analytics-ready, historical data)


```
---

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
