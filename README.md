# E-Commerce Data Engineering Project — Databricks Medallion Pipeline

An end-to-end e-commerce data pipeline built on **Databricks** using the **Medallion Architecture** (Bronze → Silver → Gold), implemented in PySpark notebooks. Raw e-commerce data is ingested, cleaned and conformed, then modeled into a **star schema** (dimensions + fact) ready for analytics and BI.

## Architecture

```
Raw CSV / source data
        │
        ▼
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│    BRONZE    │──▶│    SILVER    │──▶│     GOLD     │
│  brz_*       │   │  slv_*       │   │  gld_*       │
│ raw, as-is   │   │ cleaned,     │   │ star schema: │
│ with metadata│   │ deduped,     │   │ dimensions + │
│              │   │ typed        │   │ fact tables  │
└──────────────┘   └──────────────┘   └──────────────┘
```

All tables live in the **`ecommerce`** Unity Catalog catalog, in the schemas `bronze`, `silver`, and `gold`.

## Pipeline Notebooks

| # | Notebook | Layer | What it does |
|---|----------|-------|--------------|
| 1 | `1_dim_bronze.ipynb` | Bronze | Ingests raw dimension source data (products, brands, category, calendar) into `ecommerce.bronze.brz_*` tables with ingestion metadata |
| 2 | `1_dim_silver.ipynb` | Silver | Cleans and conforms dimension data into `ecommerce.silver.slv_brands`, `slv_category`, `slv_products`, `slv_calender` |
| 3 | `1_fact_bronze.ipynb` | Bronze | Ingests raw transactional data (orders / order items, customers) into `ecommerce.bronze.brz_order_items`, `brz_customers` |
| 4 | `2_fact_silver.ipynb` | Silver | Cleans and types transactional data into `ecommerce.silver.slv_order_items`, `slv_customers` |
| 5 | `3_dim_gold.ipynb` | Gold | Builds dimension tables `gld_dim_products`, `gld_dim_customers`, `gld_dim_date` |
| 6 | `3_fact_gold.ipynb` | Gold | Builds the fact table `gld_fact_order_items`, joined to dimension keys (star schema) |

## Gold Layer — Star Schema

- **`gold.gld_dim_products`** — product dimension (product, brand, category attributes)
- **`gold.gld_dim_customers`** — customer dimension
- **`gold.gld_dim_date`** — calendar/date dimension for time-based analysis
- **`gold.gld_fact_order_items`** — order line-item facts, keyed to the dimensions above

This supports typical analytics: revenue and units by product/category/brand, customer purchase behavior, and time-series trends.

## Repository Structure

```
ecommerce_databricks_project/
├── 1_dim_bronze.ipynb
├── 1_dim_silver.ipynb
├── 1_fact_bronze.ipynb
├── 2_fact_silver.ipynb
├── 3_dim_gold.ipynb
└── 3_fact_gold.ipynb
```

Notebooks are ordered by execution: bronze dimension load → silver dimensions → bronze fact load → silver facts → gold dimensions → gold fact.

## Getting Started

### Prerequisites

- A Databricks workspace with **Unity Catalog**
- A catalog named `ecommerce` with schemas `bronze`, `silver`, `gold` (create if needed)
- Source data available (CSV files or volume paths referenced inside the bronze notebooks)

### Running the Pipeline

1. Import the notebooks into your Databricks workspace (or clone this repo into a Databricks Git folder).
2. Run the notebooks in order: `1_dim_bronze` → `1_dim_silver` → `1_fact_bronze` → `2_fact_silver` → `3_dim_gold` → `3_fact_gold`.
3. Query the gold tables, e.g.:

```sql
SELECT p.brand, SUM(f.quantity) AS total_units
FROM ecommerce.gold.gld_fact_order_items f
JOIN ecommerce.gold.gld_dim_products p
  ON f.product_key = p.product_key
GROUP BY p.brand
ORDER BY total_units DESC;
```

*(Adjust key column names to match your final schema.)*

## Tech Stack

- **Databricks** (notebooks, Unity Catalog)
- **PySpark** / Spark SQL
- **Delta Lake** tables
- **Medallion Architecture** (Bronze → Silver → Gold)

---

Feel free to open issues or suggestions!
