Question 1: **Are the visitor IDs on the analytics table the same as the ones on all_sessions, and if not,**
			**what is the total number of visitor IDs?**

Repurposed from what I understood the question on the assignment.md to mean. 
Queries 1 and 2 - counts of distinct IDs in all_sessions and analytics 
Query 3 - looking for IDs that are missing from either of the tables with the assumption that, based on how many unique
	IDS are in the analytics table, there will be a lot
Query 4 - count of unique IDs between the two tables
SQL Queries:
```sql
SELECT COUNT(DISTINCT(fullvisitorid)), 'all_sessions'
FROM all_sessions;


SELECT COUNT(DISTINCT(fullvisitorid)), 'analytics'
FROM analytics_distinct;


SELECT a.fullvisitorid, s.fullvisitorid
FROM analytics_distinct a
FULL OUTER JOIN all_sessions s USING (fullvisitorid)
WHERE a.fullvisitorid IS NULL
	OR s.fullvisitorid IS NULL;


WITH joined AS (
SELECT COALESCE(a.fullvisitorid, s.fullvisitorid) AS fullvisitorid
FROM analytics_distinct a
FULL OUTER JOIN all_sessions s USING (fullvisitorid)
)
SELECT COUNT(DISTINCT(fullvisitorid)), 'both tables'
FROM joined
```
Answer: 
14223	"all_sessions"

120018	"analytics"

Selection of 1,667,134 returns from query 3:

"fullvisitorid"	"fullvisitorid-2"
6161810276922547829	
5268505332547388426	
	3608475193341679870
	3608475193341679870


130345	"both tables"


Question 2: **What products, by category, were ordered the most, using total_ordered from the sales_report? Is there a** 
			**category that is particularly popular?**
Query 1 - includes standardized categories I intended to use to make counts possible
Query 2 - the raw categories to see if there was any sense in there

SQL Queries:

```sql 
SELECT DISTINCT(productSKU), total_ordered, name, 
	CASE WHEN v2ProductCategory LIKE '%Accessories%' THEN 'Accessories'
	WHEN v2ProductCategory LIKE '%Electronics%' THEN 'Electronics'
	WHEN v2ProductCategory LIKE '%Apparel%' THEN 'Apparel'
	WHEN v2ProductCategory LIKE '%Shop by Brand%' THEN 'Branded'
	WHEN v2ProductCategory LIKE '%Brands%' THEN 'Branded'
	WHEN v2ProductCategory LIKE '%Drinkware%' THEN 'Drinkware'
	WHEN v2ProductCategory LIKE '%Headgear%' THEN 'Headgear'
	WHEN v2ProductCategory LIKE '%Bottles%' THEN 'Drinkware'
	WHEN v2ProductCategory LIKE '%Bags%' THEN 'Bags'
	WHEN v2ProductCategory LIKE '%Lifestyle%' THEN 'Lifestyle'
	WHEN v2ProductCategory LIKE '%Office%' THEN 'Office'
	WHEN v2ProductCategory LIKE '%Nest%' THEN 'Nest'
	WHEN v2ProductCategory LIKE '%Kids%' THEN 'Kids'
	WHEN v2ProductCategory LIKE '%Wearables%' THEN 'Apparel'
	ELSE 'Misc'
	END AS category	
FROM sales_report
JOIN all_sessions USING (productSKU)
ORDER BY productSKU;


SELECT DISTINCT(productSKU), total_ordered, name, v2ProductCategory
FROM sales_report
JOIN all_sessions USING (productSKU)
ORDER BY productSKU;
``` 

Answer:
(subset of answers for one SKU)
"productsku"	"total_ordered"	"name"	"category"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"Branded"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"Misc"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"Office"

"productsku"	"total_ordered"	"name"	"v2productcategory"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"(not set)"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"Home/Office/"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"Home/Office/Notebooks & Journals/"
"GGOEA0CH077599"	15	"Android Hard Cover Journal"	"Home/Shop by Brand/Android/"

Despite all being for the same SKU, there are multiple categories showing up. Even with the category streamlined, there
are apparently multiple sight routes to an item and missing data getting in the way of doing this kind of analysis 
cleanly.


Question 3: Which items were viewed the most times by visitorid, with and without incorporating page views?
Using the names from sales_report for standardization.
Query 1 - set up temp table with names from sales_report
Query 2 - top 10 most viewed based only on count of how many times the sku appears 
Query 3 - top 10 most viewed including pageview count; assumes all views are on the same SKU
Query 4 - compares if the overall lists are 1:1

SQL Queries:
```sql
WITH item_set AS (
SELECT fullVisitorID, pageviews, date, visitID, productSKU, sr.name
FROM all_sessions
JOIN sales_report sr USING (productSKU)
)
SELECT DISTINCT*
INTO TEMP TABLE item_views
FROM item_set;


SELECT COUNT(*), productsku, name
FROM item_views
GROUP BY productsku, name
ORDER BY count DESC
LIMIT 10;


SELECT SUM(pageviews) AS views, productsku, name
FROM item_views
GROUP BY productsku, name
ORDER BY views DESC
LIMIT 10;


WITH count_10 AS (
SELECT COUNT(*), productsku, name
FROM item_views
GROUP BY productsku, name
ORDER BY count DESC
LIMIT 10
),
sum_10 AS(
SELECT SUM(pageviews) AS views, productsku, name
FROM item_views
GROUP BY productsku, name
ORDER BY views DESC
LIMIT 10);


SELECT c.productsku AS ranked_by_visitors, 
	COALESCE(c.name, s.name), s.productsku AS ranked_by_pageview
FROM count_10 c
FULL OUTER JOIN sum_10 s USING (productsku)
ORDER BY c.productsku;
```
Answer:
"views"	"productsku"	"name"
284	"GGOEGAAX0104"	" Men's 100% Cotton Short Sleeve Hero Tee White"
247	"GGOEYDHJ056099"	"22 oz  Bottle Infuser"
221	"GGOEYHPB072210"	" Twill Cap"
216	"GGOEYFKQ020699"	" Custom Decals"
197	"GGOEGAAX0318"	" Men's Short Sleeve Hero Tee Black"
178	"GGOEGAAX0325"	" Men's Short Sleeve Hero Tee Charcoal"
170	"GGOEGAAX0106"	" Men's 100% Cotton Short Sleeve Hero Tee Navy"
170	"GGOEYOCR077799"	" Hard Cover Journal"
164	"GGOEGAAX0105"	" Men's 100% Cotton Short Sleeve Hero Tee Black"
156	"GGOEGAAX0351"	" Men's Vintage Henley"


1233	"GGOEGAAX0104"	" Men's 100% Cotton Short Sleeve Hero Tee White"
747	"GGOEYDHJ056099"	"22 oz  Bottle Infuser"
724	"GGOEGAAX0318"	" Men's Short Sleeve Hero Tee Black"
708	"GGOEGAAX0105"	" Men's 100% Cotton Short Sleeve Hero Tee Black"
707	"GGOEGAAX0106"	" Men's 100% Cotton Short Sleeve Hero Tee Navy"
620	"GGOEYHPB072210"	" Twill Cap"
608	"GGOEYFKQ020699"	" Custom Decals"
591	"GGOEGBFC018799"	"Electronics Accessory Pouch"
566	"GGOEGAAX0568"	" Men's Watershed Full Zip Hoodie Grey"
564	"GGOEGAAX0338"	" Men's Vintage Badge Tee Black"


"ranked_by_visitors"	"coalesce"	"ranked_by_pageview"
"GGOEGAAX0104"	" Men's 100% Cotton Short Sleeve Hero Tee White"	"GGOEGAAX0104"
"GGOEGAAX0105"	" Men's 100% Cotton Short Sleeve Hero Tee Black"	"GGOEGAAX0105"
"GGOEGAAX0106"	" Men's 100% Cotton Short Sleeve Hero Tee Navy"	"GGOEGAAX0106"
"GGOEGAAX0318"	" Men's Short Sleeve Hero Tee Black"	"GGOEGAAX0318"
"GGOEGAAX0325"	" Men's Short Sleeve Hero Tee Charcoal"	
"GGOEGAAX0351"	" Men's Vintage Henley"	
"GGOEYDHJ056099"	"22 oz  Bottle Infuser"	"GGOEYDHJ056099"
"GGOEYFKQ020699"	" Custom Decals"	"GGOEYFKQ020699"
"GGOEYHPB072210"	" Twill Cap"	"GGOEYHPB072210"
"GGOEYOCR077799"	" Hard Cover Journal"	
	" Men's Vintage Badge Tee Black"	"GGOEGAAX0338"
	" Men's Watershed Full Zip Hoodie Grey"	"GGOEGAAX0568"
	"Electronics Accessory Pouch"	"GGOEGBFC018799"

So whether you are counting each visitor as one or including each pageview does change which items make it into the
top ten, but only a little bit. And the top two items remained completely unchanged. 