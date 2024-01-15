### Most Popular Restaurants (based on the number of ratings):
```sql
SELECT *
FROM sales
ORDER BY num_of_ratings DESC
LIMIT 5;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/38afbe88-8236-4e57-a4a7-2e09b586bd51)
***

### Top-rated Restaurants (based on ratings):
I have taken **409** ( 90 percentile of num_of_ratings) as a **threshold** to eliminate new restaurants because they have potentially unreliable ratings. So, only the top 10% of restaurants will be included, ensuring high quality and credibility.
```sql
SELECT *
FROM sales
WHERE rating >= 4.5 AND num_of_ratings >= 409
ORDER BY rating DESC, num_of_ratings DESC
LIMIT 5;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/321381fc-911d-4776-b526-a48c408a75ba)
***

### Cheapest restaurants 
```sql
SELECT *
FROM sales
WHERE avg_cost_two_people IS NOT NULL
ORDER BY avg_cost_two_people
LIMIT 5;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/ad035cba-aeb8-4973-acb6-b2f8e90e9b85)
- Restaurants are mostly new and have Quick Bites as common restaurant types.
***

### Expensive restaurants 
```sql
SELECT *
FROM sales
ORDER BY avg_cost_two_people DESC
LIMIT 5;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/a97cf7e0-8c2e-4968-b6f5-1f7847d2e6e1)
- Restaurants have ratings of 4+ and have Fine Dining as a common restaurant type.
***

### Does having an Online Delivery option affect Ratings or the Average cost of two people?
```sql
SELECT
  online_order,
  COUNT(*) AS num_restaurants,
  CONCAT(ROUND(COUNT(*)*100/(SELECT COUNT(*) FROM sales),2),'%') AS percent,
  ROUND(AVG(rating),2) AS avg_rating,
  ROUND(AVG(avg_cost_two_people),2) AS avg_cost_two_people
FROM sales
GROUP BY 1;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/b2ba9724-6e3e-4a8a-9979-dcffd1c41ac4)
- Restaurants with online delivery have a **slightly higher average rating** (3.55) compared to those without (3.47).
- Restaurants with online delivery have a **lower average cost** for two people (487.75) than those without (599.00), which is surprising.
***

1. Why does an online delivery option have a lower average cost?
```sql
SELECT 
  online_order,
  MIN(avg_cost_two_people) AS minimum,
  MAX(avg_cost_two_people) AS maximum,
  ROUND(AVG(avg_cost_two_people), 2) AS average,
  MAX(CASE WHEN percentile BETWEEN 0 AND 25 THEN avg_cost_two_people END) AS '25th Percentile',
  MAX(CASE WHEN percentile BETWEEN 25 AND 50 THEN avg_cost_two_people END) AS 'Median',
  MAX(CASE WHEN percentile BETWEEN 50 AND 75 THEN avg_cost_two_people END) AS '75th Percentile',
  MAX(CASE WHEN percentile BETWEEN 50 AND 90 THEN avg_cost_two_people END) AS '90th Percentile'
FROM (
  SELECT 
    online_order,
    avg_cost_two_people,
    PERCENT_RANK() OVER (PARTITION BY online_order ORDER BY avg_cost_two_people) * 100 AS percentile
  FROM sales
) AS percentile_data
GROUP BY online_order;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/22aa1f9e-212b-42a5-b09d-3374c67321b4)
- The minimum, maximum, and values after the median are less for having an online delivery option compared with not having one.
- Higher values pull up the average cost of not having an online delivery option. 
***

### Does having Table Booking option affect Ratings or the Average cost of two people?
```sql
SELECT 
  table_booking,
  COUNT(*) AS num_restaurants,
  CONCAT(ROUND(COUNT(*)*100/(SELECT COUNT(*) FROM sales),2),'%') AS percent,
  ROUND(AVG(rating),2) AS avg_rating,
  ROUND(AVG(avg_cost_two_people),2) AS avg_cost_two_people
FROM sales
GROUP BY 1;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/911ca5eb-70c4-4342-b533-cc604d539e2f)

***
