# Global E-Commerce Sales Intelligence Pipeline

## Project Overview
Build a fully automated data pipeline that ingests real-world e-commerce transaction data, transforms it through multiple layers, orchestrates it with Airflow, stores it on AWS, and serves insights through Power BI.

**Dataset:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
This dataset contains 100k+ real orders, customers, products, sellers, payments, and reviews across multiple tables—perfect for a real-world data engineering project.

---

## Architecture Overview
```text
Kaggle Dataset (Source)
        │
        ▼
  Apache Airflow          ← Orchestration (local or EC2 free tier)
  (ETL Pipeline)
        │
     Extract
        │
        ▼
  AWS S3 (Raw Layer)      ← Landing zone for raw CSVs
        │
     Transform
  (Python/Pandas)
        │
        ▼
  AWS S3 (Clean Layer)    ← Cleaned, transformed parquet files
        │
       Load
        │
        ▼
  AWS RDS / Redshift      ← Data Warehouse (free tier)
        │
        ▼
  Power BI Desktop        ← Dashboard & Visualizations (free)
```

---

## AWS Free Tier Services Used

| Service | What For | Free Tier Limit |
| :--- | :--- | :--- |
| **S3** | Raw + cleaned data storage | 5GB free |
| **RDS (PostgreSQL)** | Relational data warehouse | 750hrs/month free |
| **EC2 (t2.micro)** | Host Airflow | 750hrs/month free |
| **IAM** | Access management | Always free |
| **CloudWatch** | Monitor pipeline | 10 metrics free |

---

## Project Layers (Step-by-Step)

### Layer 1 — Raw Landing Zone (S3)
Download the Kaggle dataset and upload raw CSVs to S3 as-is. This is your **Bronze Layer** (untouched source data).

```text
s3://your-bucket/raw/
    ├── orders.csv
    ├── customers.csv
    ├── order_items.csv
    ├── products.csv
    ├── sellers.csv
    ├── payments.csv
    └── reviews.csv
```

### Layer 2 — Transformation (Python + Pandas)
Clean and transform raw data into an analytics-ready format. This is your **Silver Layer**.
**Key Transformations:**
- Handle NULLs and missing values.
- Parse and standardize date columns.
- Calculate derived columns (delivery days, revenue, etc.).
- Translate Portuguese product categories to English.
- Deduplicate records.
- Convert CSVs to Parquet (columnar format for faster querying).

```text
s3://your-bucket/clean/
    ├── dim_customers.parquet
    ├── dim_products.parquet
    ├── dim_sellers.parquet
    ├── fact_orders.parquet
    └── fact_payments.parquet
```

### Layer 3 — Data Warehouse (AWS RDS PostgreSQL)
Load clean data into a proper **Star Schema**.

#### Dimension Tables
- `dim_customers`: (customer_id, city, state, region)
- `dim_products`: (product_id, category, name, weight, dimensions)
- `dim_sellers`: (seller_id, city, state)
- `dim_date`: (date_id, date, day, month, quarter, year, weekday)

#### Fact Tables
- `fact_orders`: (order_id, customer_id, seller_id, product_id, date_id, quantity, price, freight, total_revenue, delivery_days, review_score, payment_type)

### Layer 4 — Orchestration (Apache Airflow)
Install Airflow on EC2 (t2.micro) and build a DAG to automate the entire pipeline.

**DAG Flow:**
`extract_from_s3_raw` ──► `transform_and_clean` ──► `load_to_rds` ──► `data_quality_checks`

### Layer 5 — Data Quality Checks
Add a quality check task at the end of your DAG to ensure data integrity.

```python
def run_quality_checks():
    checks = [
        "SELECT COUNT(*) FROM fact_orders WHERE order_id IS NULL",
        "SELECT COUNT(*) FROM fact_orders WHERE total_revenue < 0",
        "SELECT COUNT(*) FROM dim_customers WHERE customer_id IS NULL",
    ]
    for check in checks:
        result = run_query(check)
        if result[0][0] > 0:
            raise ValueError(f"Data quality check failed: {check}")
```

---

## Analytics & Insights (SQL Questions)

### Sales Performance
1. What is the monthly revenue trend over 2 years?
2. Which product categories generate the most revenue?
3. Which states have the highest number of orders?

### Customer Analytics
4. What is the average order value per customer segment?
5. What percentage of customers are repeat buyers?
6. What is the customer distribution across Brazilian states?

### Delivery & Operations
7. What is the average delivery time by seller state?
8. How many orders were delivered late vs on time?
9. Which sellers have the worst average delivery times?

### Payments & Reviews
10. What payment methods are most commonly used?
11. Is there a correlation between delivery delay and low review scores?
12. Which product categories have the lowest average review scores?

---

## Power BI Dashboard Structure
- **Page 1: Executive Summary** — Total revenue, total orders, avg order value, avg delivery days (KPI cards) + monthly revenue trend.
- **Page 2: Sales by Geography** — Brazil map showing orders and revenue by state.
- **Page 3: Product Intelligence** — Top 10 categories by revenue, review scores by category.
- **Page 4: Delivery Performance** — On-time vs late delivery pie chart, delay vs review score scatter plot.
- **Page 5: Payment Analysis** — Payment method breakdown, revenue by payment installments.

---

## What Makes This CV-Worthy

| Skill Demonstrated | How It's Applied |
| :--- | :--- |
| **ETL Pipeline** | Extract → Transform → Load across 3 distinct layers. |
| **Cloud (AWS)** | Usage of S3, RDS, EC2, IAM, and CloudWatch. |
| **Orchestration** | Airflow DAG with scheduling, retries, and dependencies. |
| **Data Modeling** | Star schema design and implementation in PostgreSQL. |
| **Data Quality** | Automated SQL checks integrated into the pipeline. |
| **SQL** | Complex analytical queries for business intelligence. |
| **Visualization** | Multi-page Power BI dashboard with actionable KPIs. |
| **Python** | Pandas for transformation and custom Airflow operators. |
| **File Optimization** | CSV ingestion to Parquet conversion for performance. |

---

## GitHub Repository Structure
```text
ecommerce-data-pipeline/
├── dags/
│   └── ecommerce_etl_dag.py
├── etl/
│   ├── extract.py
│   ├── transform.py
│   └── load.py
├── sql/
│   ├── create_schema.sql
│   ├── create_tables.sql
│   └── analytical_queries.sql
├── quality_checks/
│   └── checks.py
├── dashboard/
│   └── ecommerce_dashboard.pbix
├── requirements.txt
├── README.md
└── architecture_diagram.png
```
