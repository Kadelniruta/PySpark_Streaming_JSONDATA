# Complex JSON Processing & IoT Sensor Streaming using PySpark

## 📌 Overview

This repository contains **two real-world PySpark data engineering implementations** executed in a **Databricks environment**:

1. **Company Complex JSON Processing** – Batch-style processing of deeply nested enterprise company data
2. **IoT Sensor Streaming JSON Processing** – Near real-time style processing of hierarchical IoT sensor data

Both pipelines demonstrate **best practices for handling complex nested JSON**, schema normalization, and building **analytics-ready flat tables** using Apache Spark.

---

## 🏗️ Overall Architecture

```
Source JSON (Nested)
      ↓
Spark Read (JSON, inferSchema, multiline)
      ↓
explode_outer (Hierarchical Arrays)
      ↓
Column Selection & Aliasing
      ↓
Flattened DataFrames
      ↓
Analytics / BI / Data Warehouse
```

---

## 🧰 Technologies Used

* Apache Spark (PySpark)
* Databricks Notebooks
* Spark SQL Functions
* Databricks Volumes

---

# 📘 Module 1: Company Complex JSON Processing

## 🎯 Objective

To transform a **deeply nested company JSON structure** containing departments, teams, employees, projects, and metrics into a **single flattened DataFrame** suitable for reporting and analytics.

---

## 📂 Data Source (Company)

* **Format:** JSON
* **Nature:** Batch / Historical
* **Location:**

```
/Volumes/workspace/pysparkcsv/companycomplex
```

---

## 🧩 JSON Structure (Logical)

```
company
 ├── headquarters
 ├── departments[]
 │     ├── teams[]
 │     │     ├── members[]
 │     │     │     └── projects[]
 └── metrics
       ├── employees
       └── revenue
```

---

## 🔄 Processing Steps

### 1️⃣ Read Complex JSON

```python
df = spark.read.format('json') \
    .option("inferschema", True) \
    .option("multiline", True) \
    .load("/Volumes/workspace/pysparkcsv/companycomplex")
```

---

### 2️⃣ Flatten Nested Arrays

Arrays are expanded using **`explode_outer`** to ensure no data loss when arrays are empty or null.

| Level      | Path                |
| ---------- | ------------------- |
| Department | company.departments |
| Team       | departments.teams   |
| Member     | teams.members       |
| Project    | members.projects    |

---

### 3️⃣ Data Normalization

The pipeline selects and aliases fields into logical groups:

#### 🔹 Company & Headquarters

* Company Name, Founded Year
* Address & Geo Coordinates

#### 🔹 Department

* ID, Name
* Annual Budget
* Budget Breakdown

#### 🔹 Team

* Team ID, Name
* Team Lead Details

#### 🔹 Employee

* Employee ID, Name, Role
* Skills (comma-separated)

#### 🔹 Project

* Project ID, Name
* Allocation

#### 🔹 Company Metrics

* Employee distribution (Region & Type)
* Quarterly revenue (2024–2025)

---

## 📊 Output Characteristics (Company)

* Grain: **One row per employee-project-team-department**
* Fully denormalized
* BI-ready & warehouse-friendly

---

# 📗 Module 2: IoT Sensor Streaming JSON Processing

## 🎯 Objective

To process **IoT sensor JSON data** containing facilities, sensors, readings, alerts, and thresholds, converting it into a **time-series friendly flattened dataset**.

---

## 📂 Data Source (IoT)

* **Format:** JSON
* **Nature:** Streaming-style / Incremental
* **Location:**

```
/Volumes/workspace/pysparkcsv/iot_sensor
```

---

## 🧩 JSON Structure (Logical)

```
facility
 ├── location
 ├── sensors[]
 │     ├── manufacturer
 │     ├── thresholds
 │     └── readings[]
 │            └── metadata
 │                 └── alerts[]
```

---

## 🔄 Processing Steps

### 1️⃣ Read IoT JSON

```python
df = spark.read.format('json') \
    .option("inferschema", True) \
    .option("multiline", True) \
    .load("/Volumes/workspace/pysparkcsv/iot_sensor")
```

---

### 2️⃣ Hierarchical Explosion

Nested arrays are flattened in sequence:

| Level   | Path                     |
| ------- | ------------------------ |
| Sensor  | facility.sensors         |
| Reading | sensors.readings         |
| Alert   | readings.metadata.alerts |

---

### 3️⃣ Data Normalization

#### 🔹 Facility Information

* Facility ID & Name
* Building, Floor, Zone

#### 🔹 Sensor Information

* Sensor ID & Type
* Manufacturer & Model
* Calibration Details
* Technician Info

#### 🔹 Readings Information

* Timestamp
* Value & Unit
* Quality & Confidence

#### 🔹 Alerts Information

* Alert ID
* Severity
* Message

#### 🔹 Thresholds

* Min / Max
* Critical Min / Max

---

## 🚨 Alert Handling (IoT Module)

Alerts are generated when sensor readings **violate defined thresholds** or when abnormal behavior is detected by the sensor metadata.

### 🔹 Alert Source

Alerts are nested inside the following JSON path:

```
readings.metadata.alerts[]
```

Each alert is associated with a **specific sensor reading**, ensuring accurate traceability.

---

### 🔹 Alert Attributes

The following alert-related fields are extracted and flattened:

| Column           | Description                                     |
| ---------------- | ----------------------------------------------- |
| `alert_id`       | Unique identifier for the alert                 |
| `alert_severity` | Severity level (Low / Medium / High / Critical) |
| `alert_message`  | Human-readable description of the issue         |

---

### 🔹 Why Alert Processing Is Important

* Enables **real-time monitoring & incident response**
* Helps identify **sensor failures or abnormal conditions**
* Supports **SLA tracking and compliance**
* Critical for **predictive maintenance and safety systems**

---

### 🔹 Data Grain Impact

Including alerts changes the output grain to:

> **One row per sensor → per reading → per alert**

If a reading has multiple alerts, multiple rows are generated. If no alerts exist, `explode_outer` ensures the record is still retained.

---

## 📊 Output Characteristics (IoT)

* Grain: **One row per sensor-reading-alert**
* Time-series optimized
* Suitable for monitoring dashboards, anomaly detection & alerting systems

---

## ✅ Key Design Principles (Both Pipelines)

* `explode_outer` to prevent data loss
* Explicit column aliasing for clarity
* Flat schema for analytics efficiency
* Modular & reusable transformations

---

## ⚠️ Assumptions

* JSON schema remains consistent
* Databricks runtime configured correctly
* Data volume fits Spark cluster capacity

---

## 🚀 Future Enhancements

* Convert batch to Structured Streaming
* Write output to Delta Lake
* Add schema enforcement
* Implement data quality checks
* Partition by date / facility / department

---

## 👤 Author

**PySpark Data Engineering Project**
Designed to demonstrate enterprise-grade handling of complex JSON and IoT data using Apache Spark.

---

## 🏁 Conclusion

This repository showcases **realistic enterprise and IoT data engineering use cases**, highlighting how PySpark can efficiently transform complex hierarchical JSON into actionable, analytics-ready datasets.
