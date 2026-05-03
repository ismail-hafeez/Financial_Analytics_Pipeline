# Automated Financial Market & Portfolio Analytics Pipeline

## The Concept
You will build an automated daily pipeline that extracts stock market and cryptocurrency data, processes it to calculate financial indicators (like moving averages and volatility), stores it in a cloud data warehouse, and powers a Power BI dashboard.

**Why this stands out:** It handles time-series data (which is notoriously tricky), uses a modern tech stack, and produces business-facing financial insights rather than just raw data dumps.

---

## Architecture & Tech Stack (All Free Tier)

- **Data Source:** [yfinance](https://github.com/ranaroussi/yfinance) (Yahoo Finance Python Library). Completely free, reliable, and provides massive amounts of historical and daily ticker data.
- **Orchestration:** **Apache Airflow**. 
  > [!TIP]
  > Run Airflow locally via Docker. Airflow is resource-heavy and will quickly crash a free-tier AWS EC2 t2.micro instance. Keep the orchestration local, but make it interact with the cloud.
- **Data Lake (Raw & Processed Storage):** **AWS S3** (5GB free tier).
- **Compute/Transformation:** **Python (Pandas)** handled by Airflow tasks.
- **Data Warehouse:** **AWS RDS for PostgreSQL** (750 hours/month free tier on a `db.t3.micro` or `db.t4g.micro` instance).
- **Visualization:** **Power BI Desktop** (Free).

---

## Step-by-Step Implementation

### 1. Extraction (The E in ETL)
- **Schedule:** Create an Airflow DAG scheduled to run daily after market close.
- **Script:** Write a Python script to fetch the daily OHLCV (Open, High, Low, Close, Volume) data for 20 major tech stocks (e.g., AAPL, MSFT) and 5 major cryptocurrencies (e.g., BTC, ETH) using `yfinance`.
- **Storage:** Save this raw data as JSON or Parquet files and use the `boto3` library to upload it to an S3 bucket named `market-data-raw-lake`.

### 2. Transformation (The T in ETL)
- **Ingest:** The next task in your DAG should pull the latest file from S3.
- **Clean:** Use Pandas to clean the data (handle missing values for days the market was closed).
- **Enrich:** 
  - **Add Business Value:** Create new columns calculating the 7-day and 30-day Simple Moving Averages (SMA).
  - **Metrics:** Calculate Daily Return Percentage and a "Volatility Index" (High minus Low).
- **Output:** Save this cleaned, enriched data as a CSV/Parquet and push it to a second S3 bucket named `market-data-processed`.

### 3. Loading (The L in ETL)
- **Warehouse Setup:** Set up a free-tier PostgreSQL instance on AWS RDS.
- **Schema Design:** 
  - Create an **assets dimension table** (`Ticker`, `Company Name`, `Sector`, `Asset Class`).
  - Create a **daily_prices fact table**.
- **Load:** Use Airflow (via `PostgresHook`) to load the processed data from S3 into your RDS tables.