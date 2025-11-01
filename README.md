# Spotify End-to-End Azure Data Engineering Project

## 🧭 Overview

This project demonstrates how to build a **fully automated, dynamic, and reusable end-to-end data engineering pipeline** using **Azure Data Factory (ADF)**, **Azure Data Lake Storage (ADLS)**, **Azure SQL Database**, and **Azure Databricks** — following the **Medallion Architecture**.

The pipeline is **parameter-driven** and supports **incremental loading** with **Change Data Capture (CDC)** logic. It automatically identifies and loads only the new or updated records since the last successful run.  

The overall goal is to create a **scalable, reusable, and secure data ecosystem** that supports real-time ingestion, transformation, and analytics — all integrated with **GitHub CI/CD** for automated deployments.

---

## 🏗️ Architecture Style: Medallion Architecture

This project is structured around the **Medallion Architecture**, a layered approach that improves data reliability, quality, and traceability within a data lake.

### 🥉 Bronze Layer (Raw Data)
- Stores raw, unprocessed data directly from source systems (Azure SQL).
- Preserves full fidelity of source data for audit and recovery.

### 🥈 Silver Layer (Cleaned Data)
- Transforms and validates raw data.
- Handles schema mapping, joins, and basic transformations using **Databricks** and **Delta Live Tables (DLT)**.

### 🥇 Gold Layer (Curated Data)
- Stores business-ready, aggregated data optimized for reporting and dashboards (via **Azure Warehouse / Synapse**).

This layered structure ensures data lineage, historical traceability, and reusability across analytics workloads.

---

## ☁️ Azure Resources and Tools Used

| Resource | Purpose |
|-----------|----------|
| **Azure Data Factory (ADF)** | Orchestrates and automates ETL workflows. |
| **Azure Data Lake Storage (ADLS)** | Central repository for Bronze, Silver, and Gold data layers. |
| **Azure SQL Database** | Source system containing structured transactional data. |
| **Azure Databricks** | Handles data transformation and aggregation using Apache Spark. |
| **Delta Live Tables (DLT)** | Automates incremental transformations with lineage and versioning. |
| **Azure Synapse / Fabric Warehouse** | Used for reporting, dashboards, and analytics queries. |
| **Apache Spark** | Provides distributed and scalable data processing. |
| **Azure Key Vault / Managed Identity** | Secures credentials and secrets across services. |
| **GitHub Actions (CI/CD)** | Enables continuous integration and deployment for ADF, Databricks, and configurations. |

---

## ⚙️ Step-by-Step Process Flow

### 1. **Setup Phase**
Create all necessary Azure resources:
- Azure Data Factory
- Azure Data Lake Storage
- Azure SQL Database and Server
- Azure Databricks workspace
- Azure Key Vault
- GitHub Repository for CI/CD

Establish linked services:
- **ADF ↔️ ADLS**
- **ADF ↔️ Azure SQL Database**

---

### 2. **Dynamic Parameterized Pipeline**

The pipeline is built to handle multiple tables dynamically using parameters:
- **Table name**
- **Source query**
- **Destination path**
- **CDC date (watermark)**

This approach makes the pipeline reusable, modular, and easy to maintain.

---

### 3. **Incremental Loading Logic**

Incremental loading ensures efficiency by loading only **new or updated data** since the last pipeline execution.  
The logic is controlled by a **cdc.json** file, which stores the latest CDC timestamp.

#### 🔄 Pipeline Activities:
1. **Lookup Activity** – Fetches last CDC value from `cdc.json`.  
2. **Set Variable Activity** – Captures current execution timestamp.  
3. **Copy Data Activity** – Loads delta data from Azure SQL to ADLS.  
4. **If Condition Activity** –  
   - If data > 0 rows → update CDC value and overwrite `cdc.json`.  
   - If data = 0 rows → delete empty file created by Copy activity.  
5. **Script Activity** – Extracts maximum CDC for next load.  
6. **Copy Data Activity** – Updates new CDC value in `container/bronze/cdc/cdc.json`.

   <img width="2530" height="1056" alt="image" src="https://github.com/user-attachments/assets/1f34fc70-f7b6-404c-919f-b109764a2587" />
   
   <img width="1952" height="498" alt="image" src="https://github.com/user-attachments/assets/c7044403-6680-4d22-884d-d5567dd183cb" />

   <img width="1630" height="424" alt="image" src="https://github.com/user-attachments/assets/dfb1076b-882a-45be-9378-d80f3a89040b" />

---

## 🧩 Technical Architecture Diagram

```text
         ┌────────────────────────────┐
         │    Azure SQL Database      │
         │ (Transactional Source)     │
         └──────────────┬─────────────┘
                        │
                        │  Linked Service (SQL → ADF)
                        ▼
         ┌────────────────────────────┐
         │   Azure Data Factory (ADF) │
         │  Parameterized Pipeline    │
         │                            │
         │  1. Lookup Last CDC        │
         │  2. Set Current Date       │
         │  3. Copy Incremental Data  │
         │  4. Update CDC JSON        │
         └──────────────┬─────────────┘
                        │
                        │  Linked Service (ADF → ADLS)
                        ▼
         ┌────────────────────────────┐
         │ Azure Data Lake Storage    │
         │    (Medallion Layers)      │
         │                            │
         │  ├── Bronze (Raw)          │
         │  │     └── cdc/cdc.json    │
         │  ├── Silver (Cleaned)      │
         │  └── Gold (Curated)        │
         └──────────────┬─────────────┘
                        │
                        │  Databricks (Spark + DLT)
                        ▼
         ┌────────────────────────────┐
         │   Azure Databricks         │
         │   Delta Live Tables (DLT)  │
         │   Transform & Merge Data   │
         └──────────────┬─────────────┘
                        │
                        ▼
         ┌────────────────────────────┐
         │   Azure Warehouse / BI     │
         │   (Reporting & Analytics)  │
         └────────────────────────────┘
