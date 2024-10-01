What are your risk areas? Identify and describe them.

- duplicated rows; need to be removed to simplify queries and for speed
	- verify rows used for PKs are unique 
- mismatch between transactions marker and recording of totaltransactionrevenue could exist; causing needed rows to be missed if 
	transactions marker is used for filering
- duplicate city values noticed during analysis; mismatches between city and country skew analysis of regional data be assigning activity to
	the wrong country

QA Process:
Describe your QA process and include the SQL queries used to execute it.

Checked each table for duplicated enteries by getting a row count both with and without DISTINCT function
```
SELECT * FROM <table>;
SELECT DISTINCT* FROM <table>;
```
4/5 tables had no change in rows returned;
	all_sessions: 15134
	products: 1092
	sales_by_sku: 462
	sales_report: 454
Table 'analysis' returned numerous duplicates; going from 4301122 to 1739308
DISTINCT query was 5-10 sec faster; going to create a new, cleaned table for the sake of processing efficiency. Also because 
trying to JOIN crashed the program.

```
SELECT * FROM analytics_distinct;
```
returns desired 1739308 rows
___

Veryify productsku row in products, sales_by_sku and sales_report is unique in each row

```
SELECT DISTINCT(productsku), productsku
FROM products --sales_by_sku / sales_report;
```
Correct outputs; 1092, 462, 454. Good to use as PKs. 
No single column in analytics_distinct or all_sessions passed this check.
___

Check that number of rows in transactions and in totaltransactionrevenue are both the same and in the same rows so that filtering by either
returns the same rows.
```
SELECT 'transactions', COUNT(*)
FROM all_sessions
WHERE transactions IS NOT NULL;
```
"transactions"	81
```
SELECT 'totaltransactionrevenue', COUNT(*)
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL;
```
"totaltransactionrevenue"	81
```
SELECT 'test', COUNT(*)
FROM all_sessions
WHERE totaltransactionrevenue IS NOT NULL 
	OR transactions IS NOT NULL;
```
"test"	81

No change in row count when given the OR option, so there is a 1:1 match and it won't matter if completed transactions are filtered using 
totaltransactionrevenue or transactions.
___

Some cities appeared multiple times; such as San Francisco, Mountain View and Vancouver. San Francisco was an obvious error; 
```
SELECT county, COUNT(city)
FROM all_sessions
WHERE city = 'San Francisco'
GROUP BY country;
```
"France"	1
"United States"	462
"Japan"		1

Others were slightly less obvious; Vancouver, Washington is a real place, as is Mountain View, Australia, but there is no Mountain View, Japan. 
```
SELECT county, COUNT(city)
FROM all_sessions
WHERE city = 'Vancouver'
GROUP BY country;

SELECT county, COUNT(city)
FROM all_sessions
WHERE city = 'Mountain View'
GROUP BY country;
```
"Canada"	6
"United States"	1

"United States" 1175
"Australia"	1
"Japan" 	1

Given the way all of these have a single count outside the main category, just like the San Fran case, I suspect they are all cases of errors, 
even though Vancouver, USA and Mountain View, AUS technically exist, and I will be recoding accordingly. 
```
SELECT country, COUNT(city)
FROM all_sessions
WHERE city = 'Toronto'
GROUP BY country;
```
"Canada"	133
"United States"	1
___





