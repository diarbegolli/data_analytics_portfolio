# Serious SQL — Summary Statistics, Outliers, and Percentile Distributions (Weight Data)

## Objective
Explore, clean, and analyze weight measurements from the `health.user_logs` dataset by:
- Removing unrealistic outliers
- Calculating robust summary statistics
- Building percentile-based distributions using window functions
- Demonstrating analytical thinking used in real data analysis

---

## Context & Data Issues
Initial summary statistics on the raw `weight` data revealed severe problems:
- Extremely large values (hundreds of thousands to millions)
- Mean and standard deviation completely distorted
- Median and mode suggesting a very different “typical” weight

These issues indicate **outliers and data quality problems**, which must be handled before any meaningful analysis.

---

## Step 1: Create a Cleaned Temporary Table
We remove:
- Zero values (invalid measurements)
- Unrealistically large values (> 200kg)

```sql
DROP TABLE IF EXISTS clean_weight_logs;

CREATE TEMP TABLE clean_weight_logs AS
SELECT *
FROM health.user_logs
WHERE
  measure = 'weight'
  AND measure_value > 0
  AND measure_value < 201;
```

---

## Step 2: Summary Statistics on Cleaned Data
This query calculates central tendency and spread metrics after outlier removal.

```sql
SELECT
  COUNT(*) AS total_records,
  ROUND(MIN(measure_value), 2) AS min_value,
  ROUND(MAX(measure_value), 2) AS max_value,
  ROUND(AVG(measure_value), 2) AS mean_value,
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
    2
  ) AS median_value,
  ROUND(MODE() WITHIN GROUP (ORDER BY measure_value), 2) AS mode_value,
  ROUND(STDDEV(measure_value), 2) AS standard_deviation,
  ROUND(VARIANCE(measure_value), 2) AS variance_value
FROM clean_weight_logs;
```

### Interpretation
- **Mean**: Sensitive to extreme values, useful only after cleaning
- **Median**: Best indicator of a “typical” weight in skewed data
- **Mode**: Most frequently logged value (often rounded or repeated)
- **Standard deviation / variance**: Describe spread around the mean

---

## Step 3: Percentile Distribution Using NTILE(100)
We divide the cleaned dataset into 100 equal-sized buckets (percentiles) to inspect the full distribution and detect tail behavior.

```sql
WITH percentile_values AS (
  SELECT
    measure_value,
    NTILE(100) OVER (ORDER BY measure_value) AS percentile
  FROM clean_weight_logs
)
SELECT
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM percentile_values
GROUP BY percentile
ORDER BY percentile;
```

### Why NTILE?
- Produces clean, integer percentile buckets (1–100)
- Easy to aggregate and visualize
- Ideal for identifying outliers and distribution skew

---

## Key Concept Explanations (Interview-Ready)

### ROW_NUMBER vs RANK vs DENSE_RANK
- **ROW_NUMBER**: Assigns a unique number to each row (no ties)
- **RANK**: Ties share the same rank, but numbers are skipped
- **DENSE_RANK**: Ties share the same rank, no gaps in ranking

### Why Use Median Instead of Mean?
The **median** is resistant to outliers.  
In skewed datasets (like weights with bad data), the mean becomes misleading, while the median still represents the central value accurately.

### NTILE vs CUME_DIST
- **NTILE(n)**: Splits data into `n` equal-sized buckets (great for grouped analysis)
- **CUME_DIST()**: Returns a cumulative probability (0–1) per row, harder to aggregate

### How I Approach Exploratory Data Analysis (EDA)
1. Inspect raw data (`SELECT *`, `LIMIT`)
2. Check row counts and uniqueness
3. Analyze distributions and frequencies
4. Identify missing values and outliers
5. Clean data based on logic + context
6. Recalculate statistics
7. Validate results with percentiles and visual intuition

---

## Final Notes
- Outlier handling is always context-dependent
- Cleaning decisions must be explainable
- Percentile analysis provides visibility that averages cannot
- This workflow reflects real-world analytical thinking, not just SQL syntax