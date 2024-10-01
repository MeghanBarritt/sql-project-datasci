What issues will you address by cleaning the data?

Requesting DISTINCT(all) from analytics cuts rows from 4,301,122 down to 
1,739,308. Manually verified multiple identical rows within visitid 1493622167.
Runs about 10 seconds faster with DISTINCT applied. 
I am going to make a cleaned version of this because trying to merge crashed the 
entire program.
__

Added unique primary key column to tables 'all_sessions' and 'analytics_distinct' 
__

Renamed column 'products.sku' to 'products.productsku' to be more obviously paired with other tables (used the GUI)
__

'unit price' to be /1 000 000; unit_price is in analytics/analytics_distinct; also apply to all_sessions.productprice? 
Otherwise the values won't make sense.
Other money related columns: (all_sessions) totalTransactionRevenue, transactionRevenue, productRevenue, 	
							(analytics) revenue
have the same amount skewing
Updated all large price values to maintain internal consistancy

all_sessions table 'city' column has value 'not available in demo dataset'; Updating to 'Unknown' for ease of reading and entry.
__

Found this anomaly working on p3q1:
"country"	"city"	"total_revenue"
"Canada"	"New York"	67.99
Changing that country value to United States.
More mismatches between cities and countries found and corrected.


Queries:
Below, provide the SQL queries you used to clean your data.
```sql
SELECT DISTINCT* 
INTO TABLE analytics_distinct
FROM analytics;


ALTER TABLE all_sessions
ADD keyid SERIAL PRIMARY KEY;

ALTER TABLE analytics_distinct
ADD keyid SERIAL PRIMARY KEY;


UPDATE all_sessions
SET productprice = productprice / 1000000::float;

UPDATE all_sessions
SET totalTransactionRevenue = totalTransactionRevenue / 1000000::float;

UPDATE all_sessions
SET transactionRevenue = transactionRevenue / 1000000::float;

UPDATE all_sessions
SET productRevenue = productRevenue / 1000000::float;

UPDATE analytics_distinct
SET unit_price = unit_price / 1000000::float;

UPDATE analytics_distinct
SET revenue = revenue / 1000000::float;


UPDATE all_sessions
SET city = 'unknown'
WHERE city = 'not available in demo dataset';


UPDATE all_sessions
SET country = CASE
	WHEN city = 'San Francisco' THEN 'United States'
	WHEN city = 'Dublin' THEN 'Ireland'
	WHEN city = 'Mountain View' THEN 'United States'
	WHEN city = 'New York' THEN 'United States'
	WHEN city = 'Mexico City' THEN 'Mexico'
	WHEN city = 'Hong Kong' THEN 'Hong Kong'
	WHEN city = 'Singapore' THEN 'Singapore'
	WHEN city = 'Bangkok' THEN 'Thailand'
	WHEN city = 'Los Angeles' THEN 'United States'
	WHEN city = 'Yokohama' THEN 'Japan'
	WHEN city = 'Istanbul' THEN 'Turkey'
	WHEN city = 'Toronto' THEN 'Canada'
	WHEN city = 'Vancouver' THEN 'Canada'
        ELSE country
    END;
```