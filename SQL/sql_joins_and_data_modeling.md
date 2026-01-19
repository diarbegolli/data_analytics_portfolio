/* =========================================================
   Serious SQL — Joins & Data Exploration
   ========================================================= */


/* ---------------------------------------------------------
   1. Distribution check:
   How many rental records exist per inventory_id?
   --------------------------------------------------------- */

WITH counts_base AS (
  SELECT
    inventory_id AS target_column_values,
    COUNT(*) AS row_counts
  FROM dvd_rentals.rental
  GROUP BY target_column_values
)
SELECT
  row_counts,
  COUNT(target_column_values) AS count_of_target_values
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;


/* ---------------------------------------------------------
   2. Join Parts 1 and 2
   rental → inventory → film
   --------------------------------------------------------- */

DROP TABLE IF EXISTS join_parts_1_and_2;

CREATE TEMP TABLE join_parts_1_and_2 AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id;

SELECT *
FROM join_parts_1_and_2
LIMIT 10;


/* ---------------------------------------------------------
   3. Complete joined dataset
   rental → inventory → film → film_category → category
   --------------------------------------------------------- */

DROP TABLE IF EXISTS complete_joint_dataset;

CREATE TEMP TABLE complete_joint_dataset AS
SELECT
  rental.customer_id,
  inventory.film_id,
  film.title,
  film_category.category_id,
  category.name AS category_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;

SELECT *
FROM complete_joint_dataset
LIMIT 10;