# Company Complex JSON Processing using PySpark

## 📌 Overview

This project demonstrates how to **read, flatten, and transform a deeply nested company JSON structure** using **Apache Spark (PySpark)** in a **Databricks environment**.

The pipeline reads a complex JSON file containing company, department, team, employee, project, and metrics data, then converts it into a **fully flattened, analytics-ready DataFrame**.

---

## 🏗️ Architecture & Flow

```
JSON File (Nested)
   ↓
Spark Read (inferSchema, multiline)
   ↓
explode_outer (Departments → Teams → Members → Projects)
   ↓
Column Selection & Aliasing
   ↓
Flattened DataFrame (Reporting / Analytics Ready)
```

---

## 📂 Data Source

* **Format:** JSON
* **Structure:** Highly nested (company → departments → teams → members → projects)
* **Location (Databricks Volume):**

```
/Volumes/workspace/pysparkcsv/companycomplex
```

---

## 🧰 Technologies Used

* **Apache Spark (PySpark)**
* **Databricks Notebook**
* **Spark SQL Functions**

---

## 🚀 How It Works

### 1️⃣ Read Complex JSON

```python
from pyspark.sql.functions import *
from pyspark.sql.types import *

 df = spark.read.format('json') \
   .option("inferschema", True) \
   .option("multiline", True) \
   .load("/Volumes/workspace/pysparkcsv/companycomplex")
```

* Automatically infers schema
* Supports multi-line JSON objects

---

### 2️⃣ Flatten Nested Arrays

The following nested arrays are exploded **safely using `explode_outer`** to avoid data loss when arrays are null:

| Level      | JSON Path           |
| ---------- | ------------------- |
| Department | company.departments |
| Team       | departments.teams   |
| Member     | teams.members       |
| Project    | members.projects    |

```python
df = df.withColumn("departments", explode_outer("company.departments")) \
       .withColumn("teams", explode_outer("departments.teams")) \
       .withColumn("members", explode_outer("teams.members")) \
       .withColumn("projects", explode_outer("members.projects"))
```

---

### 3️⃣ Column Selection & Normalization

All required fields are selected and **renamed using meaningful aliases** to create a clean tabular dataset.

#### 🔹 Company Information

* `companyName`
* `foundedYear`

#### 🔹 Headquarters Information

* Street, City, State, Country
* Latitude & Longitude

#### 🔹 Department Information

* Department ID & Name
* Annual Budget
* Budget Breakdown (Salaries, Equipment, Training, Misc)

#### 🔹 Team Information

* Team ID & Name
* Team Lead (Employee ID, Name, Email, Slack)

#### 🔹 Employee Information

* Employee ID, Name, Role
* Skills (comma-separated)

#### 🔹 Project Information

* Project ID
* Project Name
* Allocation Percentage

#### 🔹 Company Metrics

**Employees:**

* Total Employees
* By Region (North America, Europe, Asia)
* By Type (Full-time, Part-time, Contractor)

**Revenue:**

* Quarterly Revenue for 2024 & 2025

---

## 📊 Final Output

* **One row per project per employee per team per department**
* Fully flattened schema
* Ready for:

  * BI tools
  * Data Warehousing
  * Reporting
  * Advanced analytics

```python
display(df)
```

---

## ✅ Key Design Decisions

* **`explode_outer` instead of `explode`** → prevents row loss when arrays are empty or null
* **Explicit column aliasing** → improves readability & downstream usability
* **Single flattened DataFrame** → simplifies joins and reporting

---

## ⚠️ Assumptions

* JSON structure remains consistent
* Databricks environment is properly configured
* Source data volume fits Spark cluster capacity

---

## 📈 Possible Enhancements

* Add schema validation
* Write output to Delta table
* Partition by department or year
* Add streaming support (Auto Loader)

---


## 🏁 Conclusion

This project is a **real-world example of handling enterprise-level nested JSON** using PySpark, demonstrating best practices in data flattening, schema design, and analytics readiness.
