### Handling Missing Values: 
- Replaces empty strings ('') in the rating and avg_cost_two_people columns with NULL, ensuring consistency in handling missing data.

```sql
UPDATE sales
SET rating = NULLIF(rating, ''), 
    avg_cost_two_people = NULLIF(avg_cost_two_people, '');
 ```

### Column Type Modification:
- Changes the rating column's data type to FLOAT(2,1) and the avg_cost_two_people column data type to INT.
```sql
ALTER TABLE sales
MODIFY COLUMN rating FLOAT(2,1),
MODIFY COLUMN avg_cost_two_people INT;
 ```

### Area Name Standardization:
- Replacing "Byresandra,Tavarekere,Madiwala" with "BTM" in the area column to ensure consistency.
```sql
UPDATE sales 
SET area = REPLACE(area, 'Byresandra,Tavarekere,Madiwala', 'BTM');
 ```
### Restaurant Name Cleaning and Wrangling:
- This query cleans and wrangles the "restaurant_name" column by removing special characters, replacing substrings, and handling specific patterns identified by LIKE conditions. This enhances the cleanliness and consistency of restaurant names.
```sql
UPDATE sales
SET restaurant_name = REPLACE(REPLACE(REPLACE(REGEXP_REPLACE(restaurant_name, '[^a-zA-Z0-9-& \' - °]', ''),'Caf','Cafe'), '?', '\''), '\'', '')
WHERE restaurant_name LIKE '%ƒ%' OR restaurant_name LIKE '%Caf%' OR restaurant_name LIKE '%?s%' OR restaurant_name LIKE '\'%' OR restaurant_name LIKE '%~%';
 ```

After this, we are ready to perform data exploration and data analysis. Click here for data exploration.
