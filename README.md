# Delta Lake MERGE Implementation

**Author:** Vabhravi Pandey

Two hands-on data-engineering exercises in one project: classic Pandas cleaning, and incremental
loads with Delta Lake (SCD Type 1 & Type 2).

## Project Structure

```
data-processing-assignment/
│
├── data/
│   ├── Superstore_raw.csv           # raw Superstore dataset (Task 1 input)
│   ├── Superstore_cleaned.csv       # cleaned output (Task 1 deliverable)
│   ├── customer_master.csv          # synthetic customer master table (Task 2 input)
│   └── customer_incremental.csv     # simulated incremental batch (Task 2 input)
│
├── notebooks/
│   ├── 01_pandas_data_cleaning.ipynb    # Task 1 — full executed notebook
│   ├── 02_delta_lake_incremental.ipynb  # Task 2 — full executed notebook
│   └── delta_tables/                    # real Delta tables produced by Task 2 (transaction log + Parquet)
│
├── screenshots/
│   ├── data_loading/
│   ├── data_cleaning/
│   ├── scd1/
│   ├── scd2/
│   ├── validation/
│   └── final_output/
│
├── report/
│   └── assignment_summary.pdf
│
└── README.md
```

## Task 1 — Pandas Data Exploration & Cleaning

**Dataset:** [Superstore Dataset Final](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final) (Kaggle)

Steps covered: load → explore (head/tail, shape, dtypes) → identify & handle missing values →
filter rows / select columns → remove duplicates → derive `total_amount` → save cleaned CSV.

**Key finding:** the raw file's last 806 rows weren't real orders — they were leftover rows from
other tabs (People/Returns) in the original spreadsheet, accidentally concatenated in. These were
identified and dropped rather than "filled in," since they were structurally invalid, not just
sparse. See `notebooks/01_pandas_data_cleaning.ipynb` for the full walkthrough and reasoning.

| Metric | Value |
|---|---|
| Raw rows / columns | 10,800 / 21 |
| Corrupted rows dropped | 806 |
| Missing values filled | 11 (Postal Code) |
| Duplicates removed | 0 |
| Final cleaned rows / columns | 9,994 / 23 |

## Task 2 — Incremental Processing with Delta Lake

**Engine note:** this notebook uses [`deltalake`](https://delta-io.github.io/delta-rs/) (delta-rs,
installed from PyPI) rather than PySpark + `io.delta:delta-spark`. It's a much lighter setup — no
JVM or Spark cluster needed — and still produces **real Delta tables**: genuine `_delta_log`
transaction history, Parquet files, ACID `MERGE`. Equivalent PySpark SQL for every `MERGE` is
included as markdown in the notebook, since that's the more commonly seen syntax in Databricks-style
tutorials, and the logic translates directly if you'd rather run this on Spark.

Steps covered: load into a Delta table → clean (handle nulls, drop duplicates) → simulate an
incremental batch → `MERGE` (both SCD1 and SCD2 patterns) → validate → display final results.

| Metric | SCD1 (overwrite) | SCD2 (versioned history) |
|---|---|---|
| Total rows after merge | 30 | 35 |
| Current rows | 30 | 30 |
| Historical rows | 0 | 5 |

**SCD1** keeps one row per customer, always reflecting the latest state — simplest, but destroys
history on overwrite. **SCD2** uses the classic two-step Delta pattern (expire the old "current" row,
then insert a fresh one) so you can still answer "what was true as of a given date" — see
`notebooks/02_delta_lake_incremental.ipynb` for the full MERGE logic and validation.

## Screenshots

The `screenshots/` folder captures each pipeline stage — data loading, cleaning, SCD1/SCD2 merges,
validation, and final output — organized to match the folder structure above.

## Running it yourself

```bash
pip install pandas deltalake pyarrow jupyter nbformat nbclient
jupyter notebook notebooks/01_pandas_data_cleaning.ipynb
jupyter notebook notebooks/02_delta_lake_incremental.ipynb
```

If you'd rather run Task 2 on PySpark + Delta Lake directly (e.g. in Databricks or Google Colab,
where Maven Central is reachable), swap in:

```bash
pip install pyspark==3.5.1 delta-spark==3.2.0
```

and use the `DeltaTable`/`SparkSession` APIs — the Spark SQL equivalent for every merge is already
documented in the notebook's markdown cells.
