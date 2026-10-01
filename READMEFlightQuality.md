# Flight Data Quality & Partitioning with PySpark

A PySpark demo that validates a ~6 million row flight dataset with native distributed data quality checks, then benchmarks **unpartitioned vs. partitioned Parquet** to show the real-world impact of partition pruning.

## Overview

This project walks through a common data engineering workflow:

1. **Ingest** the US DOT Flight Delays dataset (`flights.csv`, ~6M rows).
2. **Validate** data quality in a single distributed pass using PySpark.
3. **Write** the data twice as Parquet: unpartitioned and partitioned by `YEAR` / `MONTH`.
4. **Benchmark** the same BI-style query against both layouts.

## Dataset

[2015 Flight Delays and Cancellations](https://www.kaggle.com/datasets/usdot/flight-delays) (US Department of Transportation), available on Kaggle.

## Data Quality Rules

All rules are evaluated in one `select` pass over the data (no extra scans):

| # | Rule | Check |
|---|------|-------|
| 1 | `TAIL_NUMBER` null ratio | Must be ≤ 1% |
| 2 | `SCHEDULED_DEPARTURE` | Must be valid military time (0–2359) |
| 3 | `CANCELLED` | Must be a boolean flag (0 or 1) |

If any rule fails, the notebook prints a warning listing each issue; otherwise it reports success.

## Partitioning Benchmark

**Query:** count all Delta Air Lines (`DL`) flights in July 2015 with a departure delay over 30 minutes.

```python
df.filter((col("YEAR") == 2015) & (col("MONTH") == 7) &
          (col("AIRLINE") == "DL") & (col("DEPARTURE_DELAY") > 30)).count()
```

- **Unpartitioned:** Spark performs a full table scan.
- **Partitioned (`YEAR/MONTH`):** Spark prunes irrelevant folders and reads only the `YEAR=2015/MONTH=7/` directory.

### Sample Results

| Layout | Query Time | Strategy |
|--------|-----------|----------|
| Unpartitioned | 0.978 s | Full table scan |
| Partitioned | 0.400 s | Partition pruning |

**Result:** 7,463 matching flights, with the partitioned query about **2.4x faster**.

> Timings vary by hardware and caching. The gap grows significantly with larger datasets and more partitions.

## Tech Stack

- Python 3.12
- Apache Spark / PySpark (with Adaptive Query Execution enabled)
- Parquet
- Kaggle Notebooks

## Getting Started

### Prerequisites

- Python 3.9+
- Java 8/11/17 (required by Spark)
- The `flights.csv` file from the Kaggle dataset above

### Install

```bash
pip install pyspark jupyter
```

### Run

1. Download `flights.csv` from Kaggle.
2. Update the paths at the top of the notebook:

   ```python
   RAW_PATH = "path/to/flights.csv"
   OUTPUT_UNPARTITIONED = "path/to/flights_unpartitioned.parquet"
   OUTPUT_PARTITIONED = "path/to/flights_partitioned.parquet"
   ```
3. Launch and run all cells:

   ```bash
   jupyter notebook flightdataqualitypartitioning.ipynb
   ```

On Kaggle, simply add the dataset to the notebook and run it as is.

## Project Structure

```
.
├── flightdataqualitypartitioning.ipynb   # Main notebook
└── README.md
```

## Key Takeaways

- Native PySpark can run multiple data quality checks in a **single pass**, avoiding repeated scans.
- Partitioning on columns that are commonly filtered (here `YEAR`, `MONTH`) lets Spark skip irrelevant data entirely.
- Choose partition columns with low-to-moderate cardinality to avoid creating too many small files.

## Possible Extensions

- Fail the pipeline (raise an exception) instead of only warning on DQ failures.
- Quarantine bad records into a separate table.
- Compare with Delta Lake or Z-ordering.
- Add more rules (e.g., delay ranges, airport code validity).

## License

MIT, or choose a license that suits your project.
