# Amazon UK Sales Analytics in Microsoft Fabric

An end-to-end data engineering and analytics project demonstrating a modern translytical architecture built entirely within the Microsoft Fabric ecosystem.

![Fabric Workspace](https://github.com/user-attachments/assets/cbf5fab7-a135-49e5-9480-5902b6b06d0f)

## Project Overview

This repository documents an end-to-end data analytics pipeline that ingests Amazon UK sales data from Kaggle, processes it with PySpark, structures it in a Lakehouse Warehouse, and visualizes it in a Power BI report. The entire workflow is orchestrated within Microsoft Fabric, highlighting its integrated capabilities for data engineering, data science, and business intelligence.

The final output is a 3-page analytical report designed to provide the Customer Success team with actionable insights into sales performance, product trends, and customer behavior.

## Solution Architecture

The project is built on a **translytical data architecture**, which combines transactional and analytical processing in a single, unified platform. This approach, enabled by Microsoft Fabric, streamlines the data flow from raw ingestion to final analysis, reducing latency and complexity.

![Fabric Architecture](https://github.com/user-attachments/assets/8206a865-6ba8-40df-b297-571a74c3ea4e)

## Tech Stack

*   **Platform:** Microsoft Fabric
*   **Data Ingestion & Processing:** Apache Spark (via PySpark in Fabric Notebooks), Kaggle API
*   **Data Warehousing:** Fabric Lakehouse & Warehouse
*   **Data Modeling & Querying:** SQL Analytics Endpoint, T-SQL
*   **Data Visualization:** Power BI
*   **Orchestration:** Fabric Data Pipelines

## Project Workflow

The pipeline is executed in a series of sequential steps:

1.  **Data Ingestion:**
    *   Data is programmatically sourced from a Kaggle dataset using the **Kaggle API** within a Fabric **PySpark Notebook**. This ensures a repeatable and automated ingestion process.

2.  **Exploratory Data Analysis (EDA) & Transformation:**
    *   The raw data undergoes comprehensive exploration and cleaning using PySpark. This stage includes handling missing values, correcting data types, standardizing formats, and engineering new features to enrich the dataset.

3.  **Data Loading:**
    *   The cleaned and transformed PySpark DataFrame is loaded into a **Fabric Warehouse**. This provides a robust, scalable, and high-performance foundation for SQL-based analytics and reporting.

4.  **Database Development:**
    *   Using the **SQL Analytics Endpoint**, a relational schema is developed on top of the warehouse data. To encapsulate business logic and simplify data access, the following T-SQL objects were created:
        *   **5 Stored Procedures:** For complex operations like data aggregation and transformations.
        *   **5 Views:** To provide simplified, pre-joined, and secure access to the underlying tables.
        *   **5 Functions:** For reusable calculations and business logic.

5.  **Semantic Modeling & Lineage:**
    *   A **Power BI Semantic Model** is built directly on the Fabric Warehouse. This model defines table relationships, hierarchies, and key business metrics using DAX (Data Analysis Expressions).
    *   Fabric's automatic **data lineage view** is utilized to track the flow of data from the initial source through every transformation step to the final report, ensuring transparency and governance.

6.  **Reporting & Visualization:**
    *   A comprehensive **3-page Power BI report** is created from the semantic model. The report is tailored for the Customer Success team and delivers insights through interactive visuals. The report includes:
        *   **Page 1: Sales Performance Overview:** High-level KPIs, sales trends over time, and regional performance maps.
        *   **Page 2: Product Analysis:** Deep dive into best-selling products, category performance, and profitability analysis.
        *   **Page 3: Customer Insights:** Analysis of customer segments, order patterns, and key account activity.
