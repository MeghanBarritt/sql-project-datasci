Answer the following questions and provide the SQL queries used to find the answer.

**Question 1: Which cities and countries have the highest level of transaction revenues on the site?** 
Query 1: overall by both city and country
Query 2: highest total for city excluding 'unknown'

SQL Queries:
```sql
SELECT country, city, 
	SUM(totalTransactionRevenue) AS total_revenue
FROM all_sessions
WHERE totalTransactionRevenue IS NOT NULL
GROUP BY country, city;

SELECT city,
	SUM(totalTransactionRevenue) AS total_revenue
FROM all_sessions
WHERE totalTransactionRevenue IS NOT NULL AND city <> 'unknown'
GROUP BY city
ORDER BY total_revenue DESC
LIMIT 1;
```
Answer:
"Australia"	"Sydney"	358
"Canada"	"Toronto"	82.16
"Israel"	"Tel Aviv-Yafo"	602
"Switzerland"	"Zurich"	16.99
"United States"	"Atlanta"	854.44
"United States"	"Austin"	157.78
"United States"	"Chicago"	449.52
"United States"	"Columbus"	21.99
"United States"	"Houston"	38.98
"United States"	"Los Angeles"	479.48
"United States"	"Mountain View"	483.36
"United States"	"Nashville"	157
"United States"	"New York"	598.35
"United States"	"Palo Alto"	608
"United States"	"San Bruno"	103.77
"United States"	"San Francisco"	1564.32
"United States"	"San Jose"	262.38
"United States"	"Seattle"	358
"United States"	"Sunnyvale"	992.23
"United States"	"unknown"	6092.56

"San Francisco"	1564.32


The United States has by far the largest transaction totals of any country. The city value that returns the highest revenue total is the 
'unknown' category, but of the known cities, the one with the highest revenue is San Francisco.
('New York, Canada'. That's not correct. Off to go fix that. It won't change the outcome here though. Given the vast majority of the transactions
are American I understand this to have been meant as New York, USA)


**Question 2: What is the average number of products ordered from visitors in each city and country?**
Calculating the quantity of products ordered by each city and country is not reasonable, because there is so much missing data in the 
productquantity column; each completed transaction must have involved the purchase of at least one item, but that is all that can be said for 
sure. It is possible to set all NULL values to one, but there is no way to know if that is an accurate representation of the actual purchase 
behavior, and it artificially forces the count and therefore the average down. You would be trading one kind of bad data for another.

Calculating the number of items as either all SKUs or unique SKUs also seems to present some issues, as it makes the 'average' into a 
simple count. 

Query 1: count of distinct SKUS by country
Query 2: attempt to average distinct SKU count

SQL Queries:
```sql
SELECT COUNT(DISTINCT(productSKU)), country
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY country;

WITH counted AS(
SELECT COUNT(DISTINCT(productSKU)) AS counted, country
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY country
)
SELECT AVG(counted)::numeric(8,3), country
FROM counted
GROUP BY country;
```
Answer:
"city"	"avg"
"Atlanta"	2.500
"Austin"	1.000
"Chicago"	1.000
"Columbus"	1.000
"Houston"	1.000
"Nashville"	1.000
"Palo Alto"	1.000
"San Bruno"	1.000
"San Jose"	1.000
"Seattle"	1.000
"Sunnyvale"	1.000
"Sydney"	1.000
"Tel Aviv-Yafo"	1.000
"Toronto"	1.000
"United States"	1.000
"unknown"	5.520
"Zurich"	1.000

"country"	"avg"
"Australia"	1.000
"Canada"	1.000
"Israel"	1.000
"Switzerland"	1.000
"United States"	2.506


"count"	"country"
1	"Australia"
1	"Canada"
1	"Israel"
1	"Switzerland"
58	"United States"

1	"Australia"
1	"Canada"
1	"Israel"
1	"Switzerland"
58	"United States" 

With how few sales there actually are, and the fact that most of the product quantities are missing, there is no good
way to answer this question. If the overall product order numbers seen on the sales_report table was somehow useable, 
that would be helpful.

**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**
Query 1 - Create temp table with simplified category names for use in aggregation
Query 2 - non-USA orders
Query 3 - USA orders only, by city and categoy, to see which cities ordered the most of each category, filtered by totals more than 1 for
	space and because there are a lot of 1s.
Query 4 - Unsorted category totals for most popular overall

SQL Queries:
```sql
SELECT 
	CASE  
	WHEN v2productcategory LIKE '%Accessories%' THEN 'Accessories'
	WHEN v2productcategory LIKE '%Apparel%' THEN 'Apparel'
	WHEN v2productcategory LIKE '%Bags%' THEN 'Bags'
	WHEN v2productcategory LIKE '%Drinkware%' THEN 'Drinkware'
	WHEN v2productcategory LIKE '%Nest%' THEN 'Nest'
	WHEN v2productcategory LIKE '%Office%' THEN 'Office'
	WHEN v2productcategory LIKE '%Shop by Brand%' THEN 'Branded'
	WHEN v2productcategory LIKE '%Electronics%' THEN 'Electronics'
	WHEN v2productcategory LIKE '%Accessories%' THEN 'Accessories'
	WHEN v2productcategory LIKE '%Housewares%' THEN 'Housewares'
		ELSE v2productcategory 
	END AS category, country, city
INTO TEMP TABLE cat_simple
FROM all_sessions
WHERE transactions IS NOT NULL;

SELECT DISTINCT(category), COUNT(category) AS orders_from, city, country
FROM cat_simple
WHERE country NOT LIKE 'United States'
GROUP BY category, country, city;

SELECT DISTINCT(category), COUNT(category) AS orders_from, city
FROM cat_simple
WHERE country LIKE 'United States'
GROUP BY category, country, city
HAVING COUNT(category) > 1
ORDER BY category, orders_from DESC;

SELECT DISTINCT(category), COUNT(category) AS orders_from, city, country
FROM cat_simple
WHERE country LIKE 'United States'
GROUP BY category, country, city
HAVING COUNT(category) > 1
ORDER BY category, orders_from DESC;

SELECT DISTINCT(category), COUNT(category) AS orders_from
FROM cat_simple
GROUP BY category
ORDER BY orders_from DESC
LIMIT 5;
```
Answer:
"category"	"orders_from"	"city"	"country"
"Apparel"	1	"Zurich"	"Switzerland"
"Nest"	1	"Sydney"	"Australia"
"Nest"	1	"Tel Aviv-Yafo"	"Israel"
"Apparel"	1	"Toronto"	"Canada"

"Accessories"	2	"San Francisco"	"United States"
"Apparel"	7	"unknown"	"United States"
"Apparel"	5	"New York"	"United States"
"Apparel"	4	"Mountain View"	"United States"
"Apparel"	2	"San Francisco"	"United States"
"Bags"	2	"unknown"	"United States"
"Electronics"	2	"unknown"	"United States"
"Nest"	8	"unknown"	"United States"
"Nest"	4	"San Francisco"	"United States"
"Nest"	3	"Palo Alto"	"United States"
"Nest"	2	"Mountain View"	"United States"
"Nest"	2	"Sunnyvale"	"United States"

In general, items from the 'Nest' and 'Apparel' categories were purchased the most. 

"category"	"orders_from"
"Nest"	27
"Apparel"	26
"Accessories"	5
"Bags"	4
"Branded"	4


**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**
For result readability, filtering results where a city/country ordered more than one of an item.
Query 1 - products ordered by city and country
Query 2 - products ordered by country only
Query 3 - product names/categories on prior query

SQL Queries:
```sql
SELECT COUNT(productsku) AS orders, productSKU, country, city
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY productsku, country, city
ORDER BY country, city;

SELECT COUNT(productsku) AS orders, productSKU, country
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY productsku, country
HAVING COUNT(productsku) > 1
ORDER BY country;

SELECT COUNT(productsku) AS orders, productSKU, v2ProductName, 
	CASE WHEN v2ProductCategory LIKE '%Nest%' THEN 'Nest'
		ELSE v2ProductCategory
	END AS 
category, country
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY productsku, v2ProductName, 
	(CASE WHEN v2ProductCategory LIKE '%Nest%' THEN 'Nest'
	ELSE v2ProductCategory
	END), country
HAVING COUNT(productsku) > 1;
```
Answer:
"orders"	"productsku"	"country"	"city"
2	"GGOENEBJ079499"	"United States"	"unknown"
2	"GGOENEBQ079199"	"United States"	"unknown"

"orders"	"productsku"	"country"
2	"GGOEGAAX0106"	"United States"
3	"GGOEGAAX0794"	"United States"
2	"GGOEGBJR018199"	"United States"
3	"GGOENEBB078899"	"United States"
7	"GGOENEBJ079499"	"United States"
5	"GGOENEBQ078999"	"United States"
2	"GGOENEBQ079099"	"United States"
2	"GGOENEBQ079199"	"United States"
2	"GGOENEBQ084699"	"United States"


The only overlaps that exist at the city level are within the 'unknown' category, which makes them meaningless; they could easily be in 
different places and we wouldn't know. On a country level, only the USA has any repeated sellers.  

"orders"	"productsku"	"v2productname"	"category"	"country"
3	"GGOEGAAX0794"	"Nest® Learning Thermostat 3rd Gen-USA"	"Nest"	"United States"
2	"GGOEGBJR018199"	"Reusable Shopping Bag"	"Bags"	"United States"
3	"GGOENEBB078899"	"Nest® Cam Indoor Security Camera - USA"	"Nest"	"United States"
6	"GGOENEBJ079499"	"Nest® Learning Thermostat 3rd Gen-USA - Stainless Steel"	"Nest"	"United States"
5	"GGOENEBQ078999"	"Nest® Cam Outdoor Security Camera - USA"	"Nest"	"United States"
2	"GGOENEBQ079099"	"Nest® Protect Smoke + CO White Battery Alarm-USA"	"Nest"	"United States"
2	"GGOENEBQ079199"	"Nest® Protect Smoke + CO White Wired Alarm-USA"	"Nest"	"United States"
2	"GGOENEBQ084699"	"Nest® Learning Thermostat 3rd Gen-USA - White"	"Nest"	"United States"

Most of the best sellers across the USA were items from the Nest category, with the outlier being reusable shopping bags.


**Question 5: Can we summarize the impact of revenue generated from each city/country?**
Query 1 - calculate total revenue, total revenue by city, and then from there the percentage each city's total contributes to the overall
	total
Query 2 - same as the 1st but only filtering by country

SQL Queries:
```sq;
WITH all_revenue AS (
SELECT SUM(totaltransactionrevenue) OVER () AS all_revenue, city
FROM all_sessions
WHERE transactions IS NOT NULL
),
region_revenue AS (
SELECT city, country, SUM(totaltransactionrevenue) AS region_revenue
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY country, city
)
SELECT DISTINCT(city), country, 
	(SELECT (region_revenue / all_revenue * 100.0)::numeric(8,3)) 
		AS percent_of, all_revenue 
FROM region_revenue
JOIN all_revenue USING (city)
ORDER BY country, city;

WITH all_revenue AS (
SELECT SUM(totaltransactionrevenue) OVER () AS all_revenue, country
FROM all_sessions
WHERE transactions IS NOT NULL
),
region_revenue AS (
SELECT country, SUM(totaltransactionrevenue) AS region_revenue
FROM all_sessions
WHERE transactions IS NOT NULL
GROUP BY country
)
SELECT DISTINCT(country), 
	(SELECT (region_revenue / all_revenue * 100.0)::numeric(8,3)) 
		AS percent_of, all_revenue 
FROM region_revenue
JOIN all_revenue USING (country)
ORDER BY country;
```
Answer:
"city" 		"country"	"percent_of" "all_revenue"
"Sydney"	"Australia"	2.507	14281.31
"Toronto"	"Canada"	0.575	14281.31
"Tel Aviv-Yafo"	"Israel"	4.215	14281.31
"Zurich"	"Switzerland"	0.119	14281.31
"Atlanta"	"United States"	5.983	14281.31
"Austin"	"United States"	1.105	14281.31
"Chicago"	"United States"	3.148	14281.31
"Columbus"	"United States"	0.154	14281.31
"Houston"	"United States"	0.273	14281.31
"Los Angeles"	"United States"	3.357	14281.31
"Mountain View"	"United States"	3.385	14281.31
"Nashville"	"United States"	1.099	14281.31
"New York"	"United States"	4.190	14281.31
"Palo Alto"	"United States"	4.257	14281.31
"San Bruno"	"United States"	0.727	14281.31
"San Francisco"	"United States"	10.954	14281.31
"San Jose"	"United States"	1.837	14281.31
"Seattle"	"United States"	2.507	14281.31
"Sunnyvale"	"United States"	6.948	14281.31
"unknown"	"United States"	42.661	14281.31


"country"	"percent_of"	"all_revenue"
"Australia"	2.507	14281.31
"Canada"	0.575	14281.31
"Israel"	4.215	14281.31
"Switzerland"	0.119	14281.31
"United States"	92.584	14281.31

Of the known city categories, San Fracisco made the most contributions to total revenue, and the USA had the vast majority of the revenue



