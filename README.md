

## 📈 Azure Data Engineering Project Breakdown

This project demonstrates a robust, scalable solution for ingesting, transforming, and analyzing large volumes of on-premises data using Azure cloud services, following the medallion architecture pattern (Bronze, Silver, Gold).

---

<img width="1287" height="697" alt="image" src="https://github.com/user-attachments/assets/194b474c-37cc-4bd7-9b6e-674c64cc93fe" />



## 1. ⚙️ Components and Data Flow

The architecture employs a fully managed, serverless, and distributed computing approach to move and transform data.

| Component | Role in Project | Key Functionality |
| :--- | :--- | :--- |
| **On-Premises SQL Server** | **Source Database** | Hosts the structured **AdventureWorksLT2017** dataset. Connected via a **Self-Hosted Integration Runtime (SHIR)**. |
| **Azure Data Factory (ADF)** | **Data Ingestion & Orchestration** | Uses automated pipelines to ingest data from SQL Server, converts it to **Parquet format** with Snappy compression, and stores it in the Bronze layer. |
| **Azure Key Vault** | **Security** | Stores credentials for the SQL Server and the Databricks access token, ensuring secure access for ADF. |
| **Azure Data Lake Storage Gen2 (ADLS Gen2)** | **Storage Layer** | Implements the **Bronze**, **Silver**, and **Gold** containers. The Silver and Gold layers use the **Delta Lake** abstraction for versioning and ACID properties. |
| **Azure Databricks** | **Core Transformation Engine** | Uses **Apache Spark** to perform two main stages of transformation: **Bronze -> Silver** (e.g., date format conversion) and **Silver -> Gold** (e.g., column renaming for reporting). |
| **Azure Synapse Analytics** | **Analytical Data Warehouse** | Provides a unified analytics platform, using its **Serverless SQL Pool** to query the Gold-layer Delta Lake files directly, creating a structured database view for reporting. |
| **Power BI** | **Visualization & Reporting** | Connects to the Synapse Serverless SQL endpoint via **Direct Query** to create visualizations and dashboards for business insights. |



---

## 2. 🧱 Medallion Architecture (ADLS Gen2)

The project leverages three distinct layers within ADLS Gen2 to manage data quality and readiness:

| Layer | Format/Abstraction | Purpose | Example Transformation |
| :--- | :--- | :--- | :--- |
| **Bronze** (Landing) | Parquet | Raw, untransformed data ingested directly from the source. | Converting raw SQL table data to **Parquet files**. |
| **Silver** (Cleansed/Conformed) | Delta Lake | Data cleaned, standardized, and conformed to basic business rules. | Converting `ModifiedDate` from UTC timestamp to **YYYY-MM-DD** format. |
| **Gold** (Curated/Reporting) | Delta Lake | Highly aggregated, dimensional, and ready for direct consumption by business intelligence tools. | Changing column names (e.g., `ProductName` to `Product_Name`). |

---

## 3. 🔄 Transformation Details (Databricks)

Databricks was used to apply business logic and structure to the data efficiently.

### Bronze to Silver Transformation
* **Objective:** Initial cleaning and standardization.
* **Action:** Converted the `ModifiedDate` column from its **UTC timestamp** format to a more human-readable **YYYY-MM-DD** date format.

### Silver to Gold Transformation
* **Objective:** Final preparation for reporting and analysis.
* **Action:** Renamed columns to a **snake\_case** convention (e.g., `TwoWordsJoinedTogether` to `two_words_joined_together`) for better readability and integration with BI tools.

---

## 4. 📊 Consumption and Reporting

The final Gold layer data is made accessible and visualizable.

* **Azure Synapse (Serverless SQL):** Created an external database and views directly over the **Gold layer Delta files**. This avoids moving the final data while providing a high-performance SQL endpoint.
* **Power BI Connection:** Connected directly to the Synapse **Serverless SQL Endpoint** using the Azure Active Directory sign-in method.
* **Data Modeling:** Critical step involved manually establishing the correct **relationships** between the tables in Power BI and setting the **cross-filter direction to **both** for comprehensive reporting.

This streamlined flow ensures data is traceable, scalable, and prepared for high-speed analysis.
