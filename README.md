# DE3_Project-LMS-Data-Engineering-using-Microsoft-Fabric

## Learning Management System (LMS) Data Engineering 📚📊

### 📑 Contents:
- **Code**
- **Project Resources:** Data files
- **Project Architecture**

The **LMS Lakehouse Project** is built using **Microsoft Fabric** to ingest, process, and layer incremental LMS data using **Medallion Architecture** and **Azure Data Lake Gen-2 (ADLS)**. The solution handles **daily ingestion** of LMS records with automated pipelines, modular notebooks, UPSERT-enabled transformations, and CI/CD integration for scalable operations.

---

### 🚀 Key Features:

- **Tech Stack:** Microsoft Fabric, ADLS Gen-2, Fabric Pipelines, Lakehouse, PySpark, Spark SQL, Azure DevOps  
- **Security:** Role-Based Access Control (RBAC) for secure access to ADLS Gen-2 (same account for both MS Fabric & Azure)
- **Lakehouse Layers:** Separate **Bronze**, **Silver**, and **Gold** Lakehouses in a Workspace for structured data flow  
- **Storage Structure in Data Lake:**
  - **Container:** `fabricproject`
  - **Raw Folder:** Initial file dump from LMS
  - **Landing Folder:** Incremental load zone

---

### ⚡ Data Processing & Ingestion:

- **Daily Incremental Ingestion:**
  - LMS system sends **one delta file per day**
  - Files first land in **Raw folder**, then **appended** to the **Landing folder** in ADLS Gen-2
  - **Append-only logic** used in Landing Zone
  - **UPSERT logic** used in the Medallion architecture layers to create **managed delta tables** in the lakehouses
  - Data loading based on **partition-based (date) + batch window (pipelines)** incremental strategy

- **Lakehouse Transformation Layers:**
  1. **Raw → Landing:** Appends new daily files without deduplication or MERGE logic
  2. **Landing → Bronze:** UPSERT logic to insert new and update existing records
  3. **Bronze → Silver:** Refined transformations like handling duplicates, null values, imputation, standardization & checking logical inconsistencies with UPSERT handling
  4. **Silver → Gold:** 
     - Performed **data modeling** by breaking a unified dataset into:
       - `dim_course`
       - `dim_student`
       - `fact_student_performance`
     - Applied **UPSERT logic** individually for all three tables
     - Enabled creation of a **semantic model** on top of `LH_Gold` Lakehouse for **Power BI integration**

---

### 🔄 Automation & CI/CD:

- **Pipelines:**
  1. **Raw to Landing Pipeline:** Automates daily ingestion with `Get Metadata` and `ForEach` with parameters: `file_name` and `processed_date`
  2. **Layered Notebooks:** Separate notebooks for each transformation step across Bronze, Silver, and Gold with parameters: `processed_date` and `workspace_name`
  3. **Master Orchestration Pipeline:** Controls full medallion flow with parameterization

- **CI/CD Workflow:**
  - **Continuous Integration:** Configured using **Azure DevOps Repos**
  - **Continuous Deployment:** Managed with **Fabric Deployment Pipelines** for automated releases

---

## 📌 How to Run the Project (without using Fabric Pipelines)

1. **Set up Azure Environment:**
   - Create **ADLS Gen2** container named `fabricproject`
   - Configure your **Microsoft Fabric workspace**

2. **Prepare Folder Structure in ADLS and create a workspace in Fabric:**
   - `raw/` — for incoming LMS files (upload one file each day)
   - `landing/` — append-only zone for partitioned daily data
   - `fabric_DEV` - for creating all the notebooks, pipelines and lakehouses

3. **Import and Configure Notebooks in Fabric:**
   - 01. Raw To Landing: Raw → Landing  
   - 02. Landing To Bronze: Landing → Bronze (Default Lakehouse: LH_Bronze)
   - 03. Silver Transformation: Bronze → Silver  (Default Lakehouse: LH_Silver)
   - 04. Gold Layer: Silver → Gold  (Default Lakehouse: LH_Gold)

4. **Run the notebooks in sequence while hardcoding the following parameters in each file:**
   - `today_file`: file name
   - `processed_date`: today's date
   - `workspace_name`: Fabric workspace name

---

This project showcases a robust, scalable **Lakehouse architecture** built for automated and incremental data ingestion, transformation, **data modeling**, and analytics using Microsoft Fabric—fully enabled for downstream **Power BI reporting**. 🎓📈
