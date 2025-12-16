# 📗 02 – Counts, Distinct Values & Distributions  
**Author:** Diar Begolli  
**Course:** Serious SQL  
**Database:** dvd_rentals (DuckDB)

---

## 🟦 Overview

In this section we explore how to:

- Count total rows in a table  
- Count distinct values  
- Calculate frequency distributions  
- Use window functions to compute percentages  
- Combine `GROUP BY`, aggregates, and window functions  

All queries here use the `dvd_rentals` schema.

---

---

# 🔹 1. Basic Row & Distinct Counts

---

## 1.1 Inspect Film List Data

```sql
SELECT * 
FROM dvd_rentals.film_list;
```

---

## 1.2 Count Total Rows in `film_list`

```sql
SELECT
  COUNT(*) AS row_count
FROM dvd_rentals.film_list;
```

---

## 1.3 Distinct Ratings in `film_list`

```sql
SELECT DISTINCT 
  rating
FROM dvd_rentals.film_list;
```

---

## 1.4 Count Distinct Categories

```sql
SELECT 
  COUNT(DISTINCT category) AS unique_category_count
FROM dvd_rentals.film_list;
```

---

---

# 🔹 2. Frequency Distributions

---

## 2.1 Rating Frequency & Percentage

```sql
SELECT
  rating,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*)::NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY rating
ORDER BY frequency DESC;
```

- `COUNT(*)` → how many films per rating  
- `SUM(COUNT(*)) OVER ()` → total films (as a window)  
- Percentage = `count_for_rating / total_count * 100`  

---

## 2.2 Rating + Category Frequency

```sql
SELECT 
  rating,
  category,
  COUNT(*) AS frequency
FROM dvd_rentals.film_list
GROUP BY rating, category
ORDER BY frequency DESC
LIMIT 5;
```

- Shows the **top 5 (rating, category)** combinations by frequency.  

---

---

# 🔹 3. Distinct Counts Per Actor

---

## 3.1 Top Actors by Number of Unique Films

```sql
SELECT
  actor_id,
  COUNT(DISTINCT film_id) AS unique_film_id
FROM dvd_rentals.film_actor
GROUP BY actor_id
ORDER BY unique_film_id DESC
LIMIT 5;
```

- `COUNT(DISTINCT film_id)` tells us how many different films each actor appears in.  
- We then sort and show the top 5.

---

---

# 🔹 4. Percentage Contributions by Category

---

## 4.1 Category Sales as Percentage of Total Sales

```sql
SELECT
  category,
  ROUND(
    100 * total_sales::NUMERIC / SUM(total_sales) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.sales_by_film_category;
```

- Uses a window `SUM(total_sales) OVER ()` to compute total sales.  
- Each row shows **what % of total sales** that category contributes.

---

## 4.2 Category Share of Unique Films

```sql
SELECT 
  category,
  ROUND(
    100 * COUNT(DISTINCT fid)::NUMERIC / SUM(COUNT(DISTINCT fid)) OVER (),
    2
  ) AS percentage
FROM dvd_rentals.film_list
GROUP BY category
ORDER BY category;
```

- `COUNT(DISTINCT fid)` = how many unique films per category  
- Window `SUM(COUNT(DISTINCT fid)) OVER ()` = total distinct films  
- We compute each category’s percentage of total unique films.

---

---

## 🧠 Key Patterns to Remember

- `COUNT(*)` → total rows  
- `COUNT(DISTINCT column)` → unique values  
- `GROUP BY col` + `COUNT(*)` → frequency table  
- `SUM(aggregate) OVER ()` → total across all rows (no GROUP BY needed)  
- `ROUND(100 * part / total, 2)` → percentage calculations  

---