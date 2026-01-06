📘 Complex JSON Flattening in Databricks (PySpark)
📌 Project Overview

This project demonstrates how to read and flatten a deeply nested JSON file using PySpark in Databricks.
The JSON represents a complex company organizational structure containing departments, teams, members, projects, and metrics.

The objective is to convert hierarchical JSON data into a flat, analytics-ready tabular format.

🛠️ Tech Stack

Platform: Databricks

Language: PySpark

Framework: Apache Spark

Data Format: JSON

Processing Type: Batch Processing

📂 Input Data Description

The input JSON contains nested and complex structures:
company
 ├── name
 ├── founded
 ├── headquarters
 │    ├── address
 │    └── coordinates
 ├── departments (ARRAY)
 │    ├── id
 │    ├── name
 │    ├── budget
 │    └── teams (ARRAY)
 │         ├── teamId
 │         ├── lead
 │         └── members (ARRAY)
 │              ├── employeeId
 │              ├── skills (ARRAY)
 │              └── projects (ARRAY)
 └── metrics
      ├── employees
      └── revenue
      
📥 Reading the JSON File

The JSON file is read using Spark’s batch JSON reader with schema inference enabled:
df = spark.read.format('json') \
    .option("inferschema", True) \
    .option("multiline", True) \
    .load("/Volumes/workspace/pysparkcsv/companycomplex")
