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
SELECT
  COUNT(*) AS num_column
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
  MIN(value) AS minimum, MAX(value) AS maximum, ROUND(AVG(value),2) AS average,
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
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/e1387360-c651-43de-96b7-e13dde9f4220)
1. The range from the minimum to the maximum rating (1.8 to 4.9) indicates **variability** in the ratings, with a wide spread of values.
2. The average rating (3.51) is close to the median (3.5), suggesting a relatively **symmetric distribution** of ratings without a strong skewness. Also, the average rating of 3.51 indicates a moderate overall satisfaction level among the restaurants.
3. The maximum rating is 4.9 (close to the maximum possible rating of 5), which suggests that there are **highly rated restaurants** in the dataset.
4. The interquartile range (IQR) between Q1 and Q3 (3.2 to 3.8) indicates a **moderately compact middle 50% of the data**.
***

### 7. Numerical analysis of Num_of_ratings column
```sql
SELECT 
  MIN(value) AS minimum, MAX(value) AS maximum, ROUND(AVG(value),2) AS average,
  MAX(CASE WHEN percentile BETWEEN 0 AND 25 THEN value END) AS '25th Percentile',
  MAX(CASE WHEN percentile BETWEEN 25 AND 50 THEN value END) AS 'Median',
  MAX(CASE WHEN percentile BETWEEN 50 AND 75 THEN value END) AS '75th Percentile'
FROM
(
  SELECT 
    num_of_ratings AS value,
    PERCENT_RANK() OVER (ORDER BY num_of_ratings) * 100 AS percentile
  FROM sales
) AS percentile_data;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/a4935905-b4c2-4cbd-b47c-038db0c17a5e)

1. **Wide Range**: The number of ratings varies significantly, ranging from a minimum of 1 to a maximum of 16,345, indicating diverse distribution and potentially different levels of popularity among restaurants.
2. **Right-skewed distribution**: The average (188.93) is higher than the median (40), indicating a positive skew distribution.  This means most restaurants have relatively few ratings, while a smaller number have a very high number of ratings.
3. The **median (40)** represents that half of the restaurants have 40 or fewer ratings.
4. **Outlier potential**: The high maximum value (16,345) indicates that outliers might be present, which we need to investigate.
***

### 8. Outliers for Num_of_ratings column 
- From the previous query, we know that **Q1** (25th percentile) is 16 and **Q3** (75th percentile) is 128. So Interquartile Range (**IQR**) which is Q3 - Q1 = 128 - 16 = 112. Therefore, Upper bound which is Q3 + 1.5\*IQR = 128 + 1.5 * 112 = 296 and Lower bound which is Q1 - 1.5\*IQR = 16 - 1.5*112 = -152. So, any value that falls outside **(152,296)** will be considered an **outlier**.
```sql
WITH percentile_data AS (
  SELECT 
    num_of_ratings AS value,
    PERCENT_RANK() OVER (ORDER BY num_of_ratings) * 100 AS percentile
  FROM sales
),

quartiles AS (  
  SELECT
    MAX(CASE WHEN percentile BETWEEN 0 AND 25 THEN value END) AS Q1, 
    MAX(CASE WHEN percentile BETWEEN 50 AND 75 THEN value END) AS Q3
  FROM percentile_data
)

SELECT COUNT(*) AS num_outliers
FROM sales
WHERE num_of_ratings < (SELECT Q1 FROM quartiles) - 1.5 * (SELECT Q3 - Q1 FROM quartiles)
   OR num_of_ratings > (SELECT Q3 FROM quartiles) + 1.5 * (SELECT Q3 - Q1 FROM quartiles);
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/2fc4bf15-bf45-4ac1-85e3-7afc652699e9)
- There are 941 outliers (13.25% of the total) that fall outside the upper and lower bounds, which is a significant number.
***

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
