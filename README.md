# School Enrollment & Education Performance Analytics Platform

> **Capstone Project** — End-to-End Data Analytics using Python, Pandas, Databricks, Apache Airflow, and Databricks SQL Dashboards

---

## Architecture Overview

```
enrollment_data_raw.csv
        │
        ▼  [Airflow DAG — Local ETL]
        │   ingest → transform → validate → analytics
        │
        ▼  [Databricks Notebook: eda]
        │   Exploratory Data Analysis — understand raw data, identify 12 issues
        │
        ▼  [Databricks Notebook: bronze]
        │   education_db.bronze_enrollment (Raw Delta table — zero transformation)
        │
        ▼  [Databricks Notebook: silver]
        │   education_db.silver_enrollment (All 12 issues fixed — clean Delta table)
        │
        ▼  [Databricks Notebook: gold]
        │   6 Gold KPI Delta tables (pre-aggregated for dashboards)
        │
        ▼  [Databricks SQL Dashboard]
            9 widgets — KPI cards, charts, risk table, funnel
```

---

## Technology Stack

| Component            | Technology                              |
|---------------------|-----------------------------------------|
| Language             | Python 3.11                             |
| Data Processing      | Pandas, NumPy, PyArrow                  |
| Big Data             | Databricks Community Edition            |
| Distributed Engine   | PySpark (inside Databricks notebooks)   |
| Storage              | Delta Lake (Bronze / Silver / Gold)     |
| Workflow Orchestration | Apache Airflow 2.9 (Standalone, native Mac) |
| Visualization        | Databricks SQL Dashboards               |
| Version Control      | Git & GitHub                            |

---

## Project Structure

```
education-analytics-platform/
│
├── data/
│   ├── raw/
│   │   └── enrollment_data_raw.csv        ← 10,738 rows, 12 quality issues
│   ├── processed/
│   │   ├── bronze_raw.csv                 ← Airflow ingest output
│   │   ├── silver_clean.csv               ← Airflow transform output
│   │   └── silver_clean.parquet
│   └── gold/
│       ├── gold_yoy_trend.csv             ← Airflow analytics output
│       ├── gold_dropout_risk.csv
│       └── gold_kpi_summary.csv
│
├── airflow/
│   └── dags/
│       └── education_analytics_dag.py     ← Airflow DAG (6 tasks)
│
├── notebooks/                             ← Uploaded to Databricks Workspace
│   ├── notebook_01_eda.py                 ← EDA reference copy
│   ├── notebook_02_bronze.py              ← Bronze reference copy
│   ├── notebook_03_silver.py              ← Silver reference copy
│   └── notebook_04_gold.py               ← Gold reference copy
│
├── airflow-env/                            ← Python 3.11 virtual environment
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Databricks Workspace Structure

Notebooks are stored and run inside Databricks Workspace at:

```
/Workspace/education-analytics-platform/
├── eda        ← Notebook 1: Exploratory Data Analysis
├── bronze     ← Notebook 2: Raw ingestion to Delta table
├── silver     ← Notebook 3: Cleaning & transformation
└── gold       ← Notebook 4: KPI aggregations + SQL previews
```

---

## Dataset

| Property       | Detail                                      |
|---------------|---------------------------------------------|
| File           | `enrollment_data_raw.csv`                   |
| Total Rows     | 10,738 records                              |
| Schools        | 25 schools across 5 regions                 |
| Academic Years | 2018 – 2024 (7 years)                       |
| Grades         | Grade 1 – Grade 12                          |
| Subjects       | Mathematics, Science, English, Social Studies, Arts |

### 12 Data Quality Issues (Fixed in Silver)

| # | Type | Description |
|---|------|-------------|
| 1–4 | Missing Values | ~509 missing scores, ~397 dropout rates, ~292 budgets, ~294 pass rates |
| 5–6 | Inconsistent Naming | Mixed case school names, grade/subject abbreviations (G1, Maths, SS) |
| 7–9 | Outliers | Scores >100 or <0, enrollment ≤0, dropout rates >1.0 |
| 10 | Duplicates | ~102 exact duplicate rows |
| 11 | Wrong Types | academic_year as 'FY2023', counts as 'N/A', all strings in Bronze |
| 12 | Inconsistent Totals | total_enrollment ≠ male_count + female_count |

---

## Medallion Architecture — Delta Tables

### Bronze Layer
| Table | Rows | Description |
|-------|------|-------------|
| `education_db.bronze_enrollment` | 10,738 | Raw data as-is, all issues preserved, metadata columns added |

### Silver Layer
| Table | Rows | Description |
|-------|------|-------------|
| `education_db.silver_enrollment` | ~9,500 | All 12 issues fixed, derived columns added |

**Derived columns added in Silver:**
- `gender_ratio_female` — female_count / total_enrollment
- `performance_band` — Excellent / Good / Satisfactory / Needs Improvement
- `dropout_risk_flag` — HIGH / MEDIUM / LOW

### Gold Layer
| Table | Rows | Business Question |
|-------|------|------------------|
| `education_db.gold_yoy_trend` | 7 | How is enrollment changing year over year? |
| `education_db.gold_gender_dist` | ~175 | What is the gender balance per school? |
| `education_db.gold_dropout_risk` | ~300 | Which school-grade combos have highest dropout? |
| `education_db.gold_school_comparison` | ~25 | How does each school compare to its region? |
| `education_db.gold_grade_funnel` | 12 | Does enrollment drop in higher grades? |
| `education_db.gold_kpi_summary` | 1 | What are the headline platform KPIs? |

---

## Setup Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/education-analytics-platform.git
cd education-analytics-platform
```

### 2. Create Virtual Environment (Python 3.11)
```bash
python3.11 -m venv airflow-env
source airflow-env/bin/activate
pip install -r requirements.txt
```

### 3. Set Airflow Home
Add to `~/.zshrc`:
```bash
export AIRFLOW_HOME=~/education-analytics-platform/airflow
source ~/education-analytics-platform/airflow-env/bin/activate
```
Apply:
```bash
source ~/.zshrc
```

### 4. Initialise Airflow
```bash
airflow db init
airflow users create \
    --username admin \
    --password admin123 \
    --role Admin \
    --firstname Admin \
    --lastname User \
    --email admin@pipeline.com
```

### 5. Start Airflow Standalone
```bash
airflow standalone
```
Open: **http://localhost:8080** | Login: `admin` / `admin123`

---

## Running the Databricks Notebooks

### Step 1 — Upload Dataset to Databricks
1. Log in to **community.cloud.databricks.com**
2. Go to **Data > Add Data > Upload File**
3. Upload `data/raw/enrollment_data_raw.csv`
4. Note the DBFS path: `dbfs:/FileStore/tables/enrollment_data_raw.csv`

### Step 2 — Import Notebooks into Databricks Workspace
1. Go to **Workspace** in Databricks left sidebar
2. Navigate to your folder
3. Click dropdown → **Import**
4. Import each `.py` file from the `notebooks/` folder
5. Rename them to: `eda`, `bronze`, `silver`, `gold`

### Step 3 — Attach Cluster and Run in Order

| Order | Notebook | Runtime | Purpose |
|-------|----------|---------|---------|
| 1st | `eda` | ~2 min | Explore raw data, identify all issues |
| 2nd | `bronze` | ~3 min | Create Bronze Delta table |
| 3rd | `silver` | ~5 min | Fix all 12 issues, create Silver Delta table |
| 4th | `gold` | ~5 min | Build 6 Gold KPI tables, run SQL previews |

> For each notebook: click **Connect** (top right) → select your cluster → click **Run All**

---

## Airflow DAG

**File:** `airflow/dags/education_analytics_dag.py`
**DAG ID:** `education_medallion_pipeline`
**Schedule:** Daily at 01:00 AM (`0 1 * * *`)

### Task Flow
```
pipeline_start
    → ingest       (load & validate raw CSV)
    → transform    (clean all 12 issues with Pandas)
    → validate     (run 6 data quality checks)
    → analytics    (compute Gold KPI CSVs)
    → pipeline_complete
```

### Task Summary

| Task | What it does | Output |
|------|-------------|--------|
| `pipeline_start` | Logs run ID and date | — |
| `ingest` | Loads raw CSV, validates schema, saves bronze_raw.csv | `data/processed/bronze_raw.csv` |
| `transform` | Fixes all 12 issues using Pandas, adds derived columns | `data/processed/silver_clean.csv` |
| `validate` | Runs 6 quality tests on silver data | Pass/Fail log |
| `analytics` | Computes YoY trend, dropout risk, KPI summary | `data/gold/*.csv` |
| `pipeline_complete` | Logs final row counts via XCom | — |

### Trigger Manually
```bash
# From terminal
airflow dags trigger education_medallion_pipeline

# Check status
airflow dags list-runs -d education_medallion_pipeline
```

---

## Databricks SQL Dashboard

**Dashboard Name:** `Education Analytics Platform`

| Widget | Type | Source Table |
|--------|------|-------------|
| Total Students Enrolled | Counter | `gold_kpi_summary` |
| Avg Performance Score | Counter | `gold_kpi_summary` |
| Overall Dropout % | Counter | `gold_kpi_summary` |
| High Risk Count | Counter | `gold_kpi_summary` |
| Enrollment Trend | Line Chart | `gold_yoy_trend` |
| Gender Distribution | Grouped Bar | `gold_gender_dist` |
| Dropout Risk | Table + Colour | `gold_dropout_risk` |
| School vs Region | Bar Chart | `gold_school_comparison` |
| Grade Funnel | Bar Chart | `gold_grade_funnel` |

---

## Data Quality Tests

**File:** `tests/test_pipeline.py`

10 automated tests run against the Silver table inside Databricks:

| # | Test | Checks |
|---|------|--------|
| 1 | No null school names | school_name IS NOT NULL |
| 2 | No null regions | region IS NOT NULL |
| 3 | Valid academic years | academic_year BETWEEN 2015 AND 2025 |
| 4 | Score in range | avg_performance_score BETWEEN 0 AND 100 |
| 5 | Dropout rate in range | dropout_rate BETWEEN 0 AND 1 |
| 6 | Positive enrollment | total_enrollment > 0 |
| 7 | Enrollment math correct | ABS(total - (male + female)) <= 1 |
| 8 | No duplicate business keys | school+year+grade+subject unique |
| 9 | Valid risk flags | dropout_risk_flag IN (HIGH, MEDIUM, LOW) |
| 10 | Gender ratio valid | gender_ratio_female BETWEEN 0 AND 1 |

---

## Requirements

```
pandas>=2.0.0
numpy>=1.24.0
pyarrow>=12.0.0
requests>=2.31.0
python-dotenv>=1.0.0
apache-airflow==2.9.0
```

Install:
```bash
pip install -r requirements.txt
```

---

## Git Commands

```bash
# First time setup
git init
git remote add origin https://github.com/YOUR_USERNAME/education-analytics-platform.git

# Stage and commit
git add .
git commit -m "Initial commit: Education Analytics Medallion Pipeline"
git push -u origin main
```

---

*Built as part of a Data Engineering Capstone Project*
*Stack: Python | PySpark | Databricks | Delta Lake | Apache Airflow*