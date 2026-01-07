-- ============================================
-- SQL JOINS PRACTICE
-- Demonstrating INNER, LEFT, FULL, CROSS,
-- SEMI (EXISTS), and ANTI (NOT EXISTS) joins
-- ============================================

-- ----------------------------
-- Create base tables
-- ----------------------------

DROP TABLE IF EXISTS names;
CREATE TEMP TABLE names AS
WITH input_data (iid, first_name, title) AS (
  VALUES
    (1, 'Kate',  'Datacated Visualizer'),
    (2, 'Eric',  'Captain SQL'),
    (3, 'Danny', 'Data Wizard Of Oz'),
    (4, 'Ben',   'Mad Scientist'),
    (5, 'Dave',  'Analytics Heretic'),
    (6, 'Ken',   'The YouTuber')
)
SELECT * FROM input_data;

DROP TABLE IF EXISTS jobs;
CREATE TEMP TABLE jobs AS
WITH input_data (iid, occupation, salary) AS (
  VALUES
    (1, 'Cleaner', 'High'),
    (2, 'Janitor', 'Medium'),
    (3, 'Monkey',  'Low'),
    (6, 'Plumber', 'Ultra'),
    (7, 'Hero',    'Plus Ultra')
)
SELECT * FROM input_data;

-- ----------------------------
-- INNER JOIN
-- Returns only matching rows
-- ----------------------------

SELECT
  names.iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM names
INNER JOIN jobs
  ON names.iid = jobs.iid;

-- ----------------------------
-- LEFT JOIN
-- Keeps all rows from left table
-- ----------------------------

SELECT
  names.iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM names
LEFT JOIN jobs
  ON names.iid = jobs.iid;

-- ----------------------------
-- FULL OUTER JOIN
-- Keeps all rows from both tables
-- ----------------------------

SELECT 
  names.iid AS name_id,
  jobs.iid  AS job_id,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM names
FULL JOIN jobs
  ON names.iid = jobs.iid;

-- ----------------------------
-- CROSS JOIN
-- Cartesian product of both tables
-- ----------------------------

SELECT
  names.iid AS name_iid,
  jobs.iid  AS job_iid,
  names.first_name,
  names.title,
  jobs.occupation,
  jobs.salary
FROM names
CROSS JOIN jobs;

-- ----------------------------
-- Create table with duplicates
-- ----------------------------

DROP TABLE IF EXISTS new_jobs;
CREATE TEMP TABLE new_jobs AS
WITH input_table (iid, occupation, salary) AS (
  VALUES
    (1, 'Cleaner', 'High'),
    (1, 'Cleaner', 'Very High'),
    (2, 'Janitor', 'Medium'),
    (3, 'Monkey',  'Low'),
    (3, 'Monkey',  'Very Low'),
    (6, 'Plumber', 'Ultra'),
    (7, 'Hero',    'Plus Ultra')
)
SELECT * FROM input_table;

-- ----------------------------
-- LEFT SEMI JOIN (EXISTS)
-- Keeps rows from names that
-- exist in new_jobs
-- ----------------------------

SELECT
  names.iid,
  names.first_name
FROM names
WHERE EXISTS (
  SELECT 1
  FROM new_jobs
  WHERE names.iid = new_jobs.iid
);

-- ----------------------------
-- ANTI JOIN (NOT EXISTS)
-- Keeps rows from names that
-- do NOT exist in new_jobs
-- ----------------------------

SELECT
  names.iid,
  names.first_name
FROM names
WHERE NOT EXISTS (
  SELECT 1
  FROM new_jobs
  WHERE names.iid = new_jobs.iid
);