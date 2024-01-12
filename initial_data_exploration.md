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
	online_order, COUNT(*) AS num_restaurants, CONCAT(ROUND(COUNT(*)*100/(SELECT COUNT(*) FROM sales),2),'%') AS percent, ROUND(AVG(rating),2) AS avg_rating, ROUND(AVG(avg_cost_two_people),2) AS avg_cost_two_people
FROM sales
GROUP BY 1;
```
![image](https://github.com/Pratham955/Zomato_Banglore_Data_Analysis/assets/75075887/b2ba9724-6e3e-4a8a-9979-dcffd1c41ac4)
- Restaurants with online delivery have a slightly higher average rating (3.55) compared to those without (3.47).
- Restaurants with online delivery have a lower average cost for two people (487.75) than those without (599.00).
***
