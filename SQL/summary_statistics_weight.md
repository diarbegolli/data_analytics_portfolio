## Summary Statistics – Weight Measurements

This query calculates key summary statistics for the `weight` measure from the
`health.user_logs` table.

Before computing statistics, the data was inspected for outliers.
Extremely large values (in the millions) were identified as invalid entries.
To preserve meaningful and realistic results, the analysis is limited to
weights between **0 and 202**, which represents a reasonable human weight range.

### SQL Query

```sql
SELECT 
  MIN(measure_value) AS min_value,
  MAX(measure_value) AS max_value,
  MAX(measure_value) - MIN(measure_value) AS range_value,
  PERCENTILE_CONT(0.5) 
    WITHIN GROUP (ORDER BY measure_value) AS median_value,
  MODE() 
    WITHIN GROUP (ORDER BY measure_value) AS mode_value,
  AVG(measure_value) AS mean_value,
  STDDEV(measure_value) AS stddev_value,
  VARIANCE(measure_value) AS variance_value
FROM health.user_logs
WHERE measure = 'weight'
  AND measure_value BETWEEN 0 AND 202;