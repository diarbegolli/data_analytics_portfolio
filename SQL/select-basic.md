# 📘 01 – SQL Fundamentals: SELECT, ORDER BY, LIMIT  
**Author:** Diar Begolli  
**Course:** Serious SQL  
**Database:** dvd_rentals (DuckDB)

---

## 🟦 Overview
This chapter covers the foundational SQL operations:

- Selecting data  
- Sorting data  
- Limiting results  
- Multi-column sorting  
- Practical analytical examples  

These fundamentals are the building blocks for advanced SQL work such as joins, aggregations, and window functions.

---

---

# 🔹 1. SELECT Statements  
Basic extraction of columns and rows from a table.

---

## 1.1 Select All Columns

```sql
SELECT *
FROM dvd_rentals.language;
```

---

## 1.2 Select Specific Columns

```sql
SELECT 
  language_id,
  name
FROM dvd_rentals.language;
```

---

## 1.3 Limit Output Rows

```sql
SELECT *
FROM dvd_rentals.actor
LIMIT 10;
```

---

---

# 🔹 2. Sorting Results (ORDER BY)

Sorting helps you understand ordering, patterns, and top/bottom values.

---

## 2.1 Alphabetical Order (Ascending)

```sql
SELECT country
FROM dvd_rentals.country
ORDER BY country
LIMIT 5;
```

---

## 2.2 Reverse Alphabetical Order (Descending)

```sql
SELECT country
FROM dvd_rentals.country
ORDER BY country DESC
LIMIT 5;
```

---

## 2.3 Sorting Numeric Columns (Lowest First)

```sql
SELECT total_sales
FROM dvd_rentals.sales_by_film_category
ORDER BY 1   -- column position
LIMIT 5;
```

---

## 2.4 Sorting by Multiple Columns

```sql
SELECT
  customer_id,
  inventory_id,
  rental_date
FROM dvd_rentals.rental
ORDER BY inventory_id, rental_date DESC
LIMIT 8;
```

---

---

# 🔹 3. Practical Analysis Queries  
Real-world examples combining selection, sorting, and limits.

---

## 3.1 Category With Lowest Total Sales

```sql
SELECT 
  category,
  total_sales
FROM dvd_rentals.sales_by_film_category
ORDER BY total_sales
LIMIT 1;
```

---

## 3.2 Latest Payment Date

```sql
SELECT payment_date
FROM dvd_rentals.payment
ORDER BY payment_date DESC
LIMIT 1;
```

---

## 3.3 Category With the Highest Category ID

```sql
SELECT
  name,
  category_id
FROM dvd_rentals.category
ORDER BY category_id DESC, name
LIMIT 5;
```

---

## 3.4 Longest Films With Lowest Replacement Cost

```sql
SELECT 
  title,
  replacement_cost,
  length,
  rating
FROM dvd_rentals.film
ORDER BY length DESC, replacement_cost
LIMIT 10;
```

---

---

# 🧠 Key Takeaways

- `SELECT` retrieves data; `*` selects all columns.  
- `ORDER BY` defaults to ascending unless `DESC` is added.  
- `LIMIT` restricts the number of rows returned.  
- Multi-column sorting is critical for real analysis work.  
- Clean formatting makes SQL readable and easier to debug.

---

# 📝 Next Steps

Continue with:

➡ **02-aggregations.md**  
You will learn:

- Aggregate functions (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`)  
- `GROUP BY`  
- `HAVING`  
- Analytical exercises  

---