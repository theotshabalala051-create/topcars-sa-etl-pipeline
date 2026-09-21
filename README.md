# Top Cars South Africa — ETL Pipeline

An end-to-end data engineering pipeline that ingests raw dealership sales data and transforms it into a clean, analytics-ready star schema for reporting in Google Looker Studio.

Built on **Databricks** using the **Medallion Architecture** (Bronze → Silver → Gold) with **Unity Catalog** for governance and **Delta Lake** as the storage format.

---

## 📌 Project Overview

Top Cars South Africa sells vehicles through multiple dealerships across the country. Sales data is exported as raw CSV files, in USD, with inconsistent date formats and column naming. This project builds a repeatable pipeline that:

- Ingests raw CSV exports with zero manual editing
- Cleans and standardizes the data (currency conversion, column naming, type correction)
- Models the data into a proper **star schema** for fast, reliable reporting
- Feeds a live dashboard in Google Looker Studio

---

## 🏗️ Architecture



**Flow:** `CSV Source Files` → `Bronze Layer` → `Silver Layer` → `Gold Layer (Star Schema)` → `Google Looker Studio`

| Layer | Purpose | Tables |
|---|---|---|
| **Bronze** | Raw data, loaded as-is with only structural fixes (valid column names) | `dealer_bronze`, `models_bronze`, `sales_bronze` |
| **Silver** | Cleaned and business-ready: currency converted (USD → ZAR), types validated | `dealer_silver`, `models_silver`, `sales_silver` |
| **Gold** | Star schema for analytics: one fact table, three dimensions | `fact_sales`, `dim_dealer`, `dim_model`, `dim_date` |

---

## 🗂️ Data Model (Gold Layer)

<!-- Add your star schema / ERD diagram here -->
<!-- ![Star Schema](docs/star_schema_diagram.png) -->

**`fact_sales`** — grain: one row per sale — built from `sales_silver`, with surrogate keys resolved from each dimension.

| Dimension | Description |
|---|---|
| `dim_dealer` | Dealer name, city, country |
| `dim_model` | Car brand, model, segment, engine size, fuel type |
| `dim_date` | Calendar dimension for time-based reporting |

---

## 🛠️ Tech Stack

- **Databricks** (Free Edition / Serverless SQL) — compute and orchestration
- **Unity Catalog** — catalog/schema governance (`topcars_sa.bronze/silver/gold`)
- **Delta Lake** — storage format for all tables
- **SQL** — all transformation logic
- **Google Looker Studio** — final reporting layer
- **Git / GitHub** — version control, synced via Databricks Git folders

---

## 📁 Repository Structure

```
topcars-sa-etl-pipeline/
│
├── topcars_project/
│   ├── 00_debug_scratch.ipynb          # Ad-hoc verification & debugging queries
│   ├── 01_bronze_layer_ingestion.ipynb # Raw CSV → Bronze tables
│   ├── 02_silver_layer_transform.ipynb # Bronze → Silver (cleaning, currency, types)
│   └── 03_gold_layer_star_schema.ipynb # Silver → Gold (fact + dimension tables)
│
├── docs/
│   └── (diagrams, screenshots, data dictionary)
│
└── README.md
```

---

## 🚀 Pipeline Details

### Bronze Layer
Raw CSVs are loaded into Delta tables with minimal changes — only fixing column names that Delta Lake doesn't allow (e.g. `Price (USD)` → `Price_USD`), since spaces and parentheses are invalid characters in Delta column names. No values are altered at this stage.

### Silver Layer
- Currency conversion: USD columns converted to ZAR
- Data quality checks: nulls, duplicates, type validation
- Standardization of dealer/model reference data

### Gold Layer
- `fact_sales` is built from `sales_silver` and enriched with surrogate keys looked up from each dimension table
- Dimension tables (`dim_dealer`, `dim_model`, `dim_date`) provide clean, deduplicated reference data
- Designed as a **star schema** for optimal query performance in BI tools

---

## 📊 Dashboard

<!-- Add a link or screenshot of the final Looker Studio dashboard here -->
<!-- [View live dashboard](your-looker-studio-link-here) -->

---

## ✅ Project Status

- ✅ Bronze layer — complete, verified, version-controlled
- ✅ Silver layer —  complete, verified (USD→ZAR conversion, referential integrity confirmed)
- ❎ Gold layer — not started
- ❎ Looker Studio dashboard — not started

---

## 📖 What I Learned
Schema inference isn't guesswork — Databricks correctly auto-detected date and numeric types straight from raw CSVs, but Delta Lake strictly rejects invalid characters (spaces, parentheses) in column names, requiring explicit renaming during ingestion.
Verification beats assumption — a table can report "success" while silently loading zero rows; tracing the issue from read_files() → raw file bytes → Spark's distributed reader revealed a stale file-metadata cache, fixed by re-uploading the source file.
Star schemas separate facts from lookups — fact_sales is built from transactional data (sales_silver), not from dimension tables, which only supply surrogate keys for stable, non-breaking joins.
Centralized the exchange rate as a single SQL variable instead of hardcoding it in every calculation — one place to update, consistent everywhere.
A missing comma silently dropped a column from `sales_silver` with no visible error. Learned to always verify with `DESCRIBE TABLE` after any `CREATE OR REPLACE`, not just trust the code on screen. (If this was a job, I'd be jobless now.)
Checked referential integrity (`NOT IN` subqueries) before trusting `sales_silver` as the fact table source — catches orphaned records that would silently break joins later.

---

## 👤 Author

**Theo ML Tshabalala**
www.linkedin.com/in/motlalepula-lawrence-tshabalala-67b625168
