### 1. Number of rows
```sql
SELECT 
  COUNT(*) AS num_rows
FROM sales;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/2c0d9f02-0594-4509-9ce9-71233708b7fc)

- There are 7105 rows.
***

### 2. Number of columns
```sql
SELECT COUNT(*) AS num_column
FROM information_schema.columns
WHERE table_schema = 'zomato' AND table_name = 'sales'; 
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/fa9f7a5b-dbd1-4fdf-a9ac-841aa65b5644)

- There are 11 columns.
***

### 3. Duplicate rows
```sql
SELECT 
  COUNT(*) - COUNT(DISTINCT restaurant_name, area) AS duplicate_rows
FROM sales;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/3d360bb2-954b-4ee6-9584-bcd8db931e04)
- There are 5 duplicate rows.
***

### 4. Removing duplicate rows 
```sql
DELETE FROM sales
WHERE id IN (
    SELECT id
    FROM (
        SELECT 
            *,
            ROW_NUMBER() OVER(PARTITION BY restaurant_name, area ORDER BY restaurant_name) AS duplicate_row
        FROM sales
    ) t
    WHERE duplicate_row > 1
);
```
- Now there are 7100 rows.
***

### 5. Missing Values
```sql 
SELECT 
  SUM(restaurant_name IS NULL) AS col1_nulls,
  SUM(restaurant_type IS NULL) AS col2_nulls,
  SUM(rating IS NULL) AS col3_nulls,
  SUM(num_of_ratings IS NULL) AS col4_nulls,
  SUM(avg_cost_two_people IS NULL) AS col5_nulls,
  SUM(online_order IS NULL) AS col6_nulls,
  SUM(table_booking IS NULL) AS col7_nulls,
  SUM(cuisines_type IS NULL) AS col8_nulls,
  SUM(area IS NULL) AS col9_null,
  SUM(local_address IS NULL) AS col10_nulls
FROM sales;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/149e165f-d765-42e2-aaee-f91ba5e97045)

- Only 68 rows have missing values for rating and 57 for avg_cost_two_people, which is less than 1% of the total rows.
***

### 6. Numerical analysis of Rating column 
```sql
SELECT 
  MIN(value) AS minimum, MAX(value) AS maximum, ROUND(AVG(value),2) AS average, COUNT(DISTINCT value) AS distinct_ratings,
  MAX(CASE WHEN percentile BETWEEN 0 AND 25 THEN value END) AS '25th Percentile',
  MAX(CASE WHEN percentile BETWEEN 25 AND 50 THEN value END) AS '50th Percentile',
  MAX(CASE WHEN percentile BETWEEN 50 AND 75 THEN value END) AS '75th Percentile'
FROM
(
  SELECT 
    rating AS value,
    PERCENT_RANK() OVER (ORDER BY rating) * 100 AS percentile
  FROM sales
) AS percentile_data; 
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/e2263d4b-1b36-4e42-9702-6007ce5e4f5e)
***

### Num_of_ratings column analysis
```sql
SELECT 
  MIN(value),MAX(value),AVG(value),
  MAX(CASE WHEN percentile BETWEEN 0 AND 25 THEN value END) AS '25th Percentile',
  MAX(CASE WHEN percentile BETWEEN 25 AND 50 THEN value END) AS '50th Percentile',
  MAX(CASE WHEN percentile BETWEEN 50 AND 75 THEN value END) AS '75th Percentile'
FROM
(
  SELECT 
    num_of_ratings AS value,
    PERCENT_RANK() OVER (ORDER BY num_of_ratings) * 100 AS percentile
  FROM sales
) AS percentile_data;
```

### Outliers for num_of_ratings column 
```sql
SELECT COUNT(*)
FROM sales
WHERE num_of_ratings <
  (SELECT AVG(num_of_ratings) - 1.5 * STDDEV(num_of_ratings) FROM sales)
OR num_of_ratings >
  (SELECT AVG(num_of_ratings) + 1.5 * STDDEV(num_of_ratings) FROM sales)
ORDER BY num_of_ratings; 
```

### avg_cost_two_people column analysis
```sql
SELECT 
  MIN(value),MAX(value),AVG(value), COUNT(DISTINCT value),
  MAX(CASE WHEN percentile BETWEEN 0 AND 25 THEN value END) AS '25th Percentile',
  MAX(CASE WHEN percentile BETWEEN 25 AND 50 THEN value END) AS '50th Percentile',
  MAX(CASE WHEN percentile BETWEEN 50 AND 75 THEN value END) AS '75th Percentile'
FROM
(
  SELECT 
    avg_cost_two_people AS value,
    PERCENT_RANK() OVER (ORDER BY avg_cost_two_people) * 100 AS percentile
  FROM sales
) AS percentile_data; 
```
 
### Outliers for avg_cost_two_people column 
```sql
SELECT COUNT(*)
FROM sales
WHERE avg_cost_two_people <
  (SELECT AVG(avg_cost_two_people) - 1.5 * STDDEV(avg_cost_two_people) FROM sales)
OR avg_cost_two_people >
  (SELECT AVG(avg_cost_two_people) + 1.5 * STDDEV(avg_cost_two_people) FROM sales)
ORDER BY avg_cost_two_people;
```

### Distinct values in Categorical columns
```sql
SELECT 
	COUNT(DISTINCT restaurant_name),COUNT(DISTINCT TRIM(j1.restaurant_type)),COUNT(DISTINCT online_order),
    COUNT(DISTINCT table_booking),COUNT(DISTINCT TRIM(j2.cuisines_type)),
    COUNT(DISTINCT area),COUNT(DISTINCT local_address)
FROM sales t
JOIN json_table(replace(json_array(t.restaurant_type), ',', '","'),'$[*]' COLUMNS (restaurant_type VARCHAR(30) PATH '$')) j1
JOIN json_table(replace(json_array(t.cuisines_type), ',', '","'),'$[*]' COLUMNS (cuisines_type VARCHAR(30) PATH '$')) j2;
```
