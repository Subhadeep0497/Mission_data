# FMCG Data Lakehouse with PySpark & Delta Lake

A medallion-architecture (Bronze → Silver → Gold) data lakehouse built with **PySpark** and **Delta Lake**, using the Instacart Market Basket Analysis dataset to simulate an FMCG / retail analytics platform.

## Overview

The pipeline ingests raw CSV files (~37M rows in total), cleans and denormalizes them into analytics-ready fact and dimension tables, and produces business-level marts for BI dashboards.

```
Raw CSV  ──►  Bronze (raw + audit)  ──►  Silver (clean + modeled)  ──►  Gold (BI marts)
```

## Dataset

[Instacart Market Basket Analysis](https://www.kaggle.com/datasets/psparks/instacart-market-basket-analysis) (Kaggle)

| File | Rows |
|------|------|
| `departments.csv` | 21 |
| `aisles.csv` | 134 |
| `products.csv` | 49,688 |
| `orders.csv` | 3,421,083 |
| `order_products__prior.csv` | 32,434,489 |
| `order_products__train.csv` | 1,384,617 |

## Architecture

### 🥉 Bronze: Raw Ingestion
- Reads each CSV with an **explicit schema** (no schema inference).
- Adds audit columns: `_ingested_at` (timestamp) and `_source_file`.
- Handles Kaggle's `.csv.zip` format by unzipping automatically when needed.
- Writes each table as a Delta table.

### 🥈 Silver: Cleansing & Modeling
| Table | Description |
|-------|-------------|
| `dim_products` | Products denormalized with department and aisle names |
| `fact_orders` | Orders with null `days_since_prior_order` filled to `0.0`, an `is_first_order` flag, **partitioned by `order_dow`** |
| `fact_order_items` | `prior` and `train` order items unioned into a single table |

### 🥇 Gold: Business Marts
| Table | Description |
|-------|-------------|
| `department_metrics` | Total items sold, reordered items, reorder rate %, average cart-add position per department |
| `customer_profiles` | Lifetime orders, average days between orders, preferred order hour per customer |

## Sample Output

**Top departments by items sold**

| Department | Items Sold | Reorder Rate % | Avg Cart Position |
|------------|-----------|----------------|-------------------|
| produce | 9,888,378 | 65.05 | 8.04 |
| dairy eggs | 5,631,067 | 67.02 | 7.51 |
| snacks | 3,006,412 | 57.45 | 9.20 |
| beverages | 2,804,175 | 65.37 | 6.98 |
| frozen | 2,336,858 | 54.26 | 9.02 |

Produce and dairy eggs dominate volume, and dairy eggs has the highest reorder rate among the top five.

## Tech Stack

- Python 3.12
- Apache Spark / PySpark (Adaptive Query Execution enabled)
- Delta Lake (`delta-spark`)
- Kaggle Notebooks

## Getting Started

### Prerequisites
- Python 3.9+
- Java 8/11/17
- The Instacart dataset CSV files
- At least ~12 GB of RAM available for the Spark driver (configured as `spark.driver.memory = 12g`; reduce for smaller machines)

### Install

```bash
pip install pyspark delta-spark jupyter
```

### Run

1. Download the dataset from Kaggle.
2. Update the path variables at the top of the notebook:

   ```python
   BASE_RAW_DIR = "path/to/instacart-market-basket-analysis"
   WORKING_DATA_DIR = "path/to/data"
   LAKEHOUSE_DIR = "path/to/lakehouse"
   ```
3. Launch and run all cells:

   ```bash
   jupyter notebook fmcgdatalakehouse.ipynb
   ```

On Kaggle, add the dataset via **Add Data** and run the notebook as is.

## Output Structure

```
lakehouse/
├── bronze/
│   ├── departments/
│   ├── aisles/
│   ├── products/
│   ├── orders/
│   ├── order_products_prior/
│   └── order_products_train/
├── silver/
│   ├── dim_products/
│   ├── fact_orders/          # partitioned by order_dow
│   └── fact_order_items/
└── gold/
    ├── department_metrics/
    └── customer_profiles/
```

## Project Structure

```
.
├── fmcgdatalakehouse.ipynb   # Main pipeline notebook
└── README.md
```

## Key Concepts Demonstrated

- Medallion architecture on Delta Lake
- Explicit schema enforcement at ingestion
- Audit metadata for lineage (`_ingested_at`, `_source_file`)
- Dimensional modeling (facts and dimensions)
- Partitioned writes for faster queries
- Aggregated marts ready for BI tools

## Possible Extensions

- Add data quality checks between layers
- Incremental loads using Delta `MERGE`
- Time travel and `OPTIMIZE` / `ZORDER` examples
- Market basket analysis (frequently bought together)
- Connect the Gold layer to a BI tool such as Power BI or Tableau

## License

MIT, or choose a license that suits your project.
