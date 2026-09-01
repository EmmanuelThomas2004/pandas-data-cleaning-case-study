# ✦ E-Commerce Data Quality Engine

### Transforming messy operational data into trusted business intelligence.

<p align="center">

**Python · Pandas · Data Cleaning · Data Quality · Feature Engineering**

</p>

<p align="center">

[![Python](https://img.shields.io/badge/Python-3.x-111827?style=for-the-badge\&logo=python\&logoColor=white)]()
[![Pandas](https://img.shields.io/badge/Pandas-2.x-111827?style=for-the-badge\&logo=pandas\&logoColor=white)]()
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-111827?style=for-the-badge\&logo=jupyter\&logoColor=white)]()
[![Status](https://img.shields.io/badge/Project-Completed-111827?style=for-the-badge)]()

</p>

---

## ◈ Executive Overview

**E-Commerce Data Quality Engine** is a practical data-cleaning and preparation project built with **Python and Pandas**.

The project simulates a real-world analytics environment where raw e-commerce order data contains inconsistencies, missing values, invalid records, duplicate transactions, mixed date formats, and improperly formatted financial values.

The objective is simple:

> **Convert unreliable raw data into a clean, consistent, analysis-ready dataset.**

This project focuses not only on *how to clean data*, but also on **why each cleaning decision matters from a business perspective**.

---

## ◇ The Data Quality Challenge

Raw operational data often looks like this:

```text
" Priya "       →     "Priya"
"kochi"         →     "Kochi"
"₹500"          →     500
"delivered"     →     "DELIVERED"
"-5"            →     NaN
"10/01/2026"    →     2026-01-10
Duplicate row   →     Removed
```

Individually, these issues appear small.

Collectively, they can create:

* Incorrect revenue calculations
* Inflated order counts
* Inconsistent reporting
* Broken aggregations
* Unreliable dashboards
* Poor downstream analytics

---

# ✦ Project Architecture

```text
                 ┌────────────────────────┐
                 │   RAW ORDER DATA       │
                 │      orders.csv        │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   DATA PROFILING       │
                 │ Missing • Types • QA   │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   DATA CLEANING        │
                 │ Text • Numeric • Date  │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │   DATA VALIDATION      │
                 │ Rules • Duplicates     │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │ FEATURE ENGINEERING     │
                 │     total_amount       │
                 └────────────┬───────────┘
                              │
                              ▼
                 ┌────────────────────────┐
                 │ ANALYSIS-READY DATA    │
                 └────────────────────────┘
```

---

# ◈ Data Dictionary

| Field           | Business Meaning        | Data Type   |
| --------------- | ----------------------- | ----------- |
| `order_id`      | Unique order identifier | Integer     |
| `customer_name` | Customer name           | String      |
| `age`           | Customer age            | Numeric     |
| `city`          | Customer location       | String      |
| `order_date`    | Order placement date    | Datetime    |
| `product`       | Purchased product       | String      |
| `quantity`      | Units purchased         | Integer     |
| `price`         | Unit selling price      | Float       |
| `email`         | Customer email          | String      |
| `status`        | Current order status    | Categorical |
| `total_amount`  | Calculated order value  | Float       |

---

# ✦ Data Quality Framework

The project follows a structured cleaning framework:

| Layer            | Objective                       | Example             |
| ---------------- | ------------------------------- | ------------------- |
| **Completeness** | Identify missing information    | Missing age/email   |
| **Consistency**  | Standardize representations     | `Kochi` vs `kochi`  |
| **Validity**     | Detect logically invalid values | Negative age        |
| **Uniqueness**   | Remove duplicate records        | Duplicate orders    |
| **Accuracy**     | Preserve usable business values | Currency cleaning   |
| **Usability**    | Prepare analysis-ready fields   | Datetime conversion |

---

# ◇ Cleaning Pipeline

## 01 — Profile the Dataset

```python
import pandas as pd

df = pd.read_csv("orders.csv")

df.head()
df.info()
df.describe(include="all")
```

The first stage establishes the structure, data types, and potential quality issues within the dataset.

---

## 02 — Missing Value Analysis

```python
df.isnull().sum()
```

Missing-value percentages:

```python
(df.isnull().sum() / len(df)) * 100
```

This helps identify columns requiring further investigation.

---

## 03 — Text Normalization

Customer names:

```python
df["customer_name"] = df["customer_name"].str.strip()
```

City names:

```python
df["city"] = (
    df["city"]
    .str.strip()
    .str.title()
)
```

This ensures consistent categorical grouping.

---

## 04 — Financial Data Cleaning

Raw values may contain currency symbols:

```text
₹500
55000
1500
```

Cleaning logic:

```python
df["price"] = (
    df["price"]
    .astype(str)
    .str.replace("₹", "", regex=False)
    .str.strip()
    .astype(float)
)
```

The result is a numerical field suitable for calculations and aggregation.

---

## 05 — Business Rule Validation

Age values are validated against a basic business rule.

```python
df.loc[df["age"] < 0, "age"] = pd.NA
```

Invalid records are converted to missing values rather than allowing logically impossible data to enter downstream analysis.

---

## 06 — Date Standardization

```python
df["order_date"] = pd.to_datetime(
    df["order_date"],
    errors="coerce"
)
```

Using `errors="coerce"` ensures invalid date values become `NaT` rather than breaking the transformation pipeline.

---

## 07 — Category Standardization

Order status values are normalized:

```python
df["status"] = (
    df["status"]
    .str.strip()
    .str.upper()
)
```

Example:

```text
Delivered
delivered
DELIVERED
```

becomes:

```text
DELIVERED
```

---

## 08 — Duplicate Detection

Potential duplicate orders are identified using business-relevant fields:

```python
duplicates = df.duplicated(
    subset=[
        "customer_name",
        "product",
        "quantity",
        "price"
    ]
)

df[duplicates]
```

Duplicates are then removed:

```python
df = df.drop_duplicates(
    subset=[
        "customer_name",
        "product",
        "quantity",
        "price"
    ],
    keep="first"
)
```

---

# ✦ Feature Engineering

A new business metric is created:

```python
df["total_amount"] = (
    df["quantity"] * df["price"]
)
```

Total sales:

```python
total_sales = df["total_amount"].sum()

print("Total Sales:", total_sales)
```

This converts cleaned transactional data into a business-ready sales metric.

---

# ◈ Before → After

| Data Quality Issue | Raw State     | Clean State    |
| ------------------ | ------------- | -------------- |
| Customer name      | `" Priya "`   | `Priya`        |
| City               | `kochi`       | `Kochi`        |
| Price              | `₹500`        | `500.0`        |
| Age                | `-5`          | `NaN`          |
| Date               | Mixed formats | `datetime`     |
| Status             | `delivered`   | `DELIVERED`    |
| Duplicate order    | Repeated      | Removed        |
| Sales value        | Not available | `total_amount` |

---

# ✦ Business Impact

A clean dataset creates a reliable foundation for:

### Revenue Intelligence

Accurate order-value and revenue calculations.

### Customer Analytics

Reliable customer and demographic segmentation.

### Product Analytics

Consistent product-level aggregation.

### Operational Reporting

Reliable order-status reporting.

### BI & Visualization

Cleaner inputs for Power BI and other visualization platforms.

### Advanced Analytics

A stronger foundation for statistical analysis and machine-learning workflows.

---

# ◇ Technology Stack

```text
┌─────────────────────────────────────┐
│             DATA LAYER              │
│                                     │
│          CSV / Tabular Data         │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│         PROCESSING LAYER            │
│                                     │
│              Python                 │
│              Pandas                 │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│          ANALYSIS LAYER             │
│                                     │
│     Clean Data • KPIs • Metrics     │
└─────────────────────────────────────┘
```

---

# ✦ Repository Structure

```text
ecommerce-data-cleaning-pandas/
│
├── 📄 orders.csv
│
├── 📓 pandas_data_cleaning_case_study.ipynb
│
├── 📓 pandas_data_cleaning_case_study_solution.ipynb
│
└── 📘 README.md
```

---

# ◈ How to Run

### Clone the repository

```bash
git clone <your-repository-url>
```

### Navigate to the project

```bash
cd ecommerce-data-cleaning-pandas
```

### Install dependencies

```bash
pip install pandas jupyter
```

### Launch Jupyter

```bash
jupyter notebook
```

Open:

```text
pandas_data_cleaning_case_study_solution.ipynb
```

---

# ✦ Key Skills Demonstrated

```text
Python
  │
  ├── Pandas
  │     ├── Data Loading
  │     ├── Data Inspection
  │     ├── Missing Values
  │     ├── String Cleaning
  │     ├── Type Conversion
  │     ├── Datetime Processing
  │     ├── Duplicate Detection
  │     └── Feature Engineering
  │
  └── Data Quality
        ├── Completeness
        ├── Consistency
        ├── Validity
        └── Uniqueness
```

---

# ◇ Future Roadmap

This project can evolve into a complete analytics pipeline:

```text
Pandas Cleaning
      ↓
EDA
      ↓
SQL Analytics
      ↓
Power BI Dashboard
      ↓
Automated Data Quality Checks
      ↓
ETL Pipeline
      ↓
Production Analytics Workflow
```

Planned enhancements:

* Exploratory Data Analysis
* SQL-based analysis
* Power BI executive dashboard
* Automated data-quality validation
* Advanced KPI framework
* Customer segmentation
* Product profitability analysis
* Automated ETL workflow

---

# ✦ Final Takeaway

Data analytics does not begin with a dashboard.

It begins with **trustworthy data**.

This project demonstrates the fundamental workflow required to take imperfect operational data and transform it into a reliable analytical asset using **Python and Pandas**.

> **Clean Data → Trusted Metrics → Better Insights → Better Decisions**

---

## 👤 Author

### Emmanuel Thomas

**Aspiring Data Analyst**

`Python` · `Pandas` · `SQL` · `Power BI` · `Data Analytics`

---

<p align="center">

### ⭐ If you found this project useful, consider starring the repository.

</p>
