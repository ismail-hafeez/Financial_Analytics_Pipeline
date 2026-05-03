# NYC Urban Incident Intelligence Platform

## Project Overview
Build an end-to-end pipeline that combines NYC 311 service requests, NYPD complaint data, and borough population data to answer questions like:
- Where are service problems increasing?
- Which boroughs are most burdened per capita?
- Which complaint types are slowest to resolve?
- Do public-safety patterns correlate with service-demand spikes?

**Data Sources:**
- **NYC Open Data:** 311 dataset (2020–present, daily updates) and NYPD complaint dataset (2006–end of last year).
- **Census/ACS API:** Borough population data for per-capita analysis.

---

## Directory Structure
```text
nyc-data-pipeline/
├── airflow/
│   ├── dags/
│   │   └── nyc_pipeline_dag.py
│   ├── plugins/
│   └── requirements.txt
├── data/
│   ├── raw/
│   └── processed/
├── scripts/
│   ├── extract/
│   │   ├── fetch_311.py
│   │   ├── fetch_crime.py
│   │   └── fetch_population.py
│   ├── transform/
│   │   ├── clean_311.py
│   │   ├── clean_crime.py
│   │   └── normalize.py
│   ├── load/
│   │   └── load_to_postgres.py
├── sql/
│   ├── ddl/
│   │   └── schema.sql
│   ├── transformations/
│   └── analytics/
├── dashboards/
│   └── powerbi.pbix
├── config/
│   └── config.yaml
└── README.md
```

---

## Why This is a Strong CV Project
It is not a simple "weather to dashboard" project. It demonstrates:
- **Multi-source Data Integration:** Combining public datasets with API data.
- **Production-grade ETL:** Using Airflow for orchestration and S3 for staging.
- **Advanced SQL:** Joins, CTEs, window functions, rolling averages, and cohort analysis.
- **Business Intelligence:** Delivering actionable insights via Power BI.

---

## 🗄️ Database Schema (Star Schema)

### 🔹 Fact Tables
**fact_311_requests**
```sql
CREATE TABLE fact_311_requests (
    request_id TEXT PRIMARY KEY,
    created_date TIMESTAMP,
    closed_date TIMESTAMP,
    borough TEXT,
    complaint_type TEXT,
    status TEXT,
    resolution_time INT
);
```

**fact_crime_incidents**
```sql
CREATE TABLE fact_crime_incidents (
    complaint_id TEXT PRIMARY KEY,
    occurrence_date DATE,
    borough TEXT,
    crime_type TEXT
);
```

### 🔹 Dimension Tables
**dim_borough**
```sql
CREATE TABLE dim_borough (
    borough_id SERIAL PRIMARY KEY,
    borough_name TEXT
);
```

**dim_date**
```sql
CREATE TABLE dim_date (
    date DATE PRIMARY KEY,
    year INT,
    month INT,
    day INT
);
```

**dim_population**
```sql
CREATE TABLE dim_population (
    borough TEXT PRIMARY KEY,
    population INT
);
```

---

## Infrastructure: AWS Free Tier Stack
- **Amazon EC2:** Runs Airflow on a small instance (t3.micro/t4g.micro).
- **Amazon S3:** Raw and cleaned data storage (5 GB free).
- **Amazon RDS (PostgreSQL):** Data warehouse (750 hrs/month, 20 GB storage).
- **Cloud Management:** Set AWS Free Tier usage alerts to stay within limits.

---

## Data Pipeline Architecture

### Orchestration with Airflow
Airflow workflows are defined as DAGs. The scheduler triggers tasks as dependencies are satisfied:
`Download Data` ──► `Validate` ──► `Clean` ──► `Load` ──► `Build Marts` ──► `Refresh Dashboards`

### High-Level Design
```text
NYC Open Data APIs ──► Airflow (EC2)
                         │
                         ▼
                    S3 (Raw Layer)
                         │
                         ▼
                S3 (Cleaned/Processed)
                         │
                         ▼
                PostgreSQL (RDS)
                         │
                         ▼
                Power BI Dashboard
```

### ETL Workflow
1. **Extract:** Fetch 311, NYPD, and population data.
2. **Stage:** Store raw JSON/CSV in S3.
3. **Transform:** Validate schema, remove duplicates, standardize names, parse timestamps.
4. **Load:** Move clean data into PostgreSQL.
5. **Model:** Build SQL marts for daily trends, borough summaries, and response times.
6. **Visualize:** Refresh Power BI visuals from analytics marts.

---

## Analytics & Insights

### Key SQL Queries & Dashboard Questions
- What are the top complaint types by borough?
- Which borough has the highest 311 requests per 100,000 residents?
- What is the 7-day moving average of complaints?
- Which agencies have the slowest average resolution times?
- What is the SLA breach rate by complaint type?
- Do complaint spikes and crime spikes correlate monthly?

### Dashboard Structure (Power BI)
1. **Executive Overview:** Total requests, crimes, open-case rates, and avg resolution times.
2. **Borough Comparison:** Per-capita burden analysis.
3. **Service Performance:** Slow agencies and complaint types.
4. **Hotspots & Trends:** Georeferenced problem areas and monthly trend lines.

---

## Professional Branding (CV Descriptions)

**Option 1 (Detailed):**
> "Built an end-to-end AWS data engineering pipeline on free-tier services using Airflow, S3, and PostgreSQL to ingest and transform NYC 311, NYPD complaint, and Census data; modeled a star schema for analytics; and delivered Power BI dashboards for borough-level demand, response-time, and safety trend analysis."

**Option 2 (Concise):**
> "Designed and orchestrated an AWS-based ETL pipeline with Airflow, S3, and PostgreSQL to analyze NYC 311 and crime data; built SQL marts and Power BI dashboards for operational and public-safety insights."