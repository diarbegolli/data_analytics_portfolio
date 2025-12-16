# 📘 03 – Detecting & Handling Duplicates (health.user_logs)  
**Author:** Diar Begolli  
**Course:** Serious SQL  
**Dataset:** `health.user_logs`

---

## 🟦 Overview

In this chapter we:

- Explore a messy real-world health dataset  
- Count rows and distinct users  
- Inspect distributions of `measure` and weird values like `measure_value = 0`  
- Detect whether duplicates exist  
- Use **subqueries**, **CTEs**, and **temporary tables** to handle deduplication  
- Identify exactly which rows are duplicated and how often  

---

---

# 🔹 1. Initial Exploration of `health.user_logs`

---

## 1.1 Peek at First Rows

```sql
SELECT *
FROM health.user_logs
LIMIT 10;
```

---

## 1.2 Count Total Rows

```sql
SELECT COUNT(*)
FROM health.user_logs;
```

---

## 1.3 Count Distinct Users

```sql
SELECT COUNT(DISTINCT id)
FROM health.user_logs;
```

---

## 1.4 Measure Distribution (Frequency & Percentage)

```sql
SELECT
  measure,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (), 
    2
  ) AS percentage
FROM health.user_logs
GROUP BY measure
ORDER BY percentage DESC;
```

This gives you how common each `measure` type is (e.g. `blood_glucose`, `blood_pressure`, `weight`).

---

## 1.5 Records Where `measure_value = 0` (by Measure)

```sql
SELECT 
  measure,
  COUNT(*)
FROM health.user_logs
WHERE measure_value = 0
GROUP BY measure;
```

Useful for seeing how often “0” is used (sometimes incorrectly) as a measurement value.

---

## 1.6 Inspect Suspicious Rows: `measure_value = 0` and `blood_pressure`

```sql
SELECT *
FROM health.user_logs
WHERE measure_value = 0 
  AND measure = 'blood_pressure'
LIMIT 10;
```

This helps understand how blood pressure is stored (sometimes in `systolic/diastolic`, sometimes in `measure_value`).

---

---

# 🔹 2. Detecting If Duplicates Exist

We compare:

- Total row count vs  
- Count of **distinct** rows  

If they differ → duplicates exist.

---

## 2.1 Count Distinct Rows Using a Subquery

```sql
SELECT COUNT(*)
FROM (
  SELECT DISTINCT *
  FROM health.user_logs
) AS subquery;
```

---

## 2.2 Same Logic Using a CTE

```sql
WITH deduped_logs AS (
  SELECT DISTINCT *
  FROM health.user_logs
)
SELECT COUNT(*)
FROM deduped_logs;
```

If this count is **less** than the original `COUNT(*)`, your table has duplicates.

---

---

# 🔹 3. Using a Temporary Table for the Deduplicated Dataset

This is useful if you want to run many queries on the “clean” data.

---

## 3.1 Drop Old Temp Table If It Exists

```sql
DROP TABLE IF EXISTS deduplicated_user_logs;
```

---

## 3.2 Create Temp Table With Unique Records

```sql
CREATE TEMP TABLE deduplicated_user_logs AS
SELECT DISTINCT *
FROM health.user_logs;
```

---

## 3.3 Inspect the Deduplicated Table

```sql
SELECT *
FROM deduplicated_user_logs
LIMIT 10;
```

---

## 3.4 Count Deduplicated Rows

```sql
SELECT COUNT(*)
FROM deduplicated_user_logs;
```

Compare this with the original `health.user_logs` row count to see how many duplicates were removed.

---

---

# 🔹 4. Identifying Exact Duplicate Rows

Now we want to know **which exact combinations of columns are duplicated**.

We do this by grouping on **all columns** and counting.

---

## 4.1 Save Unique Duplicate Records in a Temp Table

```sql
DROP TABLE IF EXISTS unique_duplicate_records;

CREATE TEMPORARY TABLE unique_duplicate_records AS
SELECT *
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1;
```

---

## 4.2 Inspect Duplicate Records

```sql
SELECT *
FROM unique_duplicate_records
LIMIT 10;
```

Every row in this temp table represents a **combination that appears more than once** in `health.user_logs`.

---

---

# 🔹 5. Keeping Duplicate Counts With a CTE

Sometimes we also want to know **how many times** each duplicated record appears.

---

## 5.1 Top Duplicate Patterns by Frequency

```sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT *
FROM groupby_counts
WHERE frequency > 1
ORDER BY frequency DESC
LIMIT 10;
```

This shows the **most frequently duplicated combinations** of all columns.

---

## 5.2 Which IDs Have the Most Duplicate Rows?

```sql
WITH groupby_counts AS (
  SELECT
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic,
    COUNT(*) AS frequency
  FROM health.user_logs
  GROUP BY
    id,
    log_date,
    measure,
    measure_value,
    systolic,
    diastolic
)
SELECT
  id,
  SUM(frequency) AS total_duplicate_rows
FROM groupby_counts
WHERE frequency > 1
GROUP BY id
ORDER BY total_duplicate_rows DESC
LIMIT 10;
```

This aggregates duplicate counts **per user `id`**, showing which users have the most duplicated records.

---

---

## 🧠 Key Concepts From This Chapter

- Use `COUNT(*)` vs `COUNT(DISTINCT…)` or `SELECT DISTINCT` to detect duplicates  
- Use **subqueries** and **CTEs** to create deduplicated views of data  
- Use **TEMP TABLES** when you’ll reuse the cleaned data multiple times  
- Use `GROUP BY (all columns)` + `HAVING COUNT(*) > 1` to find duplicate rows  
- Use CTEs with `frequency` to analyze which users / dates / patterns are most duplicated  
- Always think about **business meaning**: some “duplicates” may be valid repeated events

---