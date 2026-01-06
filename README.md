📊 Complex JSON Flattening in Databricks using PySpark
📌 Overview

This project demonstrates how to read, process, and flatten deeply nested JSON data using PySpark in Databricks.
The source JSON represents a company structure with multiple nested levels including departments, teams, members, and projects.

The goal is to transform hierarchical JSON data into a flat, analytics-ready tabular format suitable for reporting, SQL analytics, and downstream processing.

🛠️ Technology Stack

Platform: Databricks

Language: PySpark

Data Format: JSON

Processing Type: Batch Processing

Libraries Used:

pyspark.sql.functions

pyspark.sql.types

📂 Input Data Description

The input JSON contains the following nested structure:

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

📥 Reading the JSON Data

The JSON file is read in batch mode with schema inference enabled.

df = spark.read.format('json') \
    .option("inferschema", True) \
    .option("multiline", True) \
    .load("/Volumes/workspace/pysparkcsv/companycomplex")


multiline = true allows reading formatted JSON

inferschema = true automatically detects nested structures

🔄 Transformation Logic
1️⃣ Exploding Nested Arrays

To flatten the hierarchical structure, multiple explode_outer() operations are used:

df = df.withColumn("departments", explode_outer("company.departments")) \
       .withColumn("teams", explode_outer("departments.teams")) \
       .withColumn("members", explode_outer("teams.members")) \
       .withColumn("projects", explode_outer("members.projects"))


This results in:

One row per company → department → team → member → project

Preserves rows even if nested arrays are empty (explode_outer)

2️⃣ Selecting and Renaming Fields

Nested fields are extracted using dot notation and renamed for clarity:

.select(
    col("company.name").alias("companyName"),
    col("company.founded").alias("foundedYear"),
    col("company.headquarters.address.city").alias("hqCity"),
    col("departments.id").alias("departmentId"),
    col("teams.teamId").alias("teamId"),
    col("members.employeeId").alias("employeeId"),
    col("projects.projectId").alias("projectId")
)

3️⃣ Handling Arrays Inside Fields

Employee skills (ARRAY) are converted into a readable string:

concat_ws(", ", col("members.skills")).alias("employeeSkills")

📤 Output Data

The final output is a fully flattened DataFrame containing:

Company-level attributes

Headquarters location details

Department budget information

Team and team lead details

Employee and skill information

Project allocation details

Company-wide employee and revenue metrics

This structure is ideal for:

BI tools

SQL analytics

Delta Lake storage

Reporting dashboards

⚠️ Important Notes

Multiple explode_outer() operations can lead to row multiplication

For large datasets, consider:

Storing raw JSON in a Bronze layer

Partial flattening in Silver

Full flattening only in Gold

🏗️ Best Practice Architecture
Raw JSON → Bronze (Nested) → Silver (Partial Flatten) → Gold (Fully Flattened)

✅ Conclusion

This Databricks notebook demonstrates:

Efficient handling of deeply nested JSON

Safe flattening using explode_outer

Transformation of complex structures into analytics-ready tables

The approach follows industry best practices for large-scale data processing using PySpark.
