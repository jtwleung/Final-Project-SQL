# What issues will you address by cleaning the data?

## ISSUES IDENTIFIED DURING DATA INSPECTION

### When doing preliminary examination of the data to prepare for import, and analysis using SQL on the tables after import, I found the following potential issues:

| Item # | Table Name | Column Name | Issue and Comments |
| ------ | ---------- | ----------- | ------------------ |
| 1 |allsessions | country | - 24 entries of '(not set)' = 0.15% occurence<br>- small percentage is negligible, so rows could be dropped if country distribution specifically is being examined, otherwise keep this column<br>- not possible to interpolate country value from city value due to missing city value data in those rows |
| 2 | | city | - '(not set)' value occuring 2.69% percent of time<br>- 'not available in demo dataset' occuring 63.11% of the time<br>- 66% of the dataset does not have city information, suggest <b><font color="red">caution</font></b> if using this column in analysis as it may not provide comprehensive view |
| 3 | | transactions | - NULL 99.46% of the rows, and value of 1 in 0.54% of the rows<br>- suggests this column is not useful and should be <b><font color="green">removed</font></b> |
| 4 | | productrefundamount | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 5 | | productquantity | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table | 
| 6 | | itemquantity | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 7 | | itemrevenue | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 8 | | searchkeyword | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 9 | | productvariant | - 99.74% of rows are '(not set)', so there is mostly not much information in this column<br> - 0.26% have some information which may be useful information<br>- suggest keeping this column with <b><font color="red">caution</font></b> that data may not provide a comprehensive view |
| 10 | | productprice | - very large numbers (largest is 249,000,000, smallest is 0 and second smallest is 790,000)<br>- this data could probably use a division by 1,000,000 to <b><font color="yellow">standardize/scale it down</font></b> to more reasonable and relatable numbers |
| 11 | | productrevenue | - largest (non-null) number is 176,400,000 and smallest (non-null) is 58,656,666<br>- probably should be <b><font color="yellow">standardized/scaled down</font></b> with division by 1,000,000 to make it relative to productprice |
| 12 | | transactionrevenue | - largest (non-null) number is 1,015,480,000 and smallest (non-null) is 169,970,000<br>- 99.97% are NULL, and only 4 unique numeric entries exist so use in large-scale analysis is limited<br>- this data should be <b><font color="yellow">standardized/scaled down</font></b> with division by 1,000,000 to make it relative to productprice and productrevenue |
| 13 | | pagetitle | - there is one entry for this column which appears to be an outlier which begins with "weixin://private/setresult/SCENE_FETCHQUEUE&"<br>- suggest <b><font color="yellow">setting this value to "NULL" as it is irrelevant data</font></b> in a clean data set<br>- if  |
| 13a | | v2productcategory | - depending on analysis scenario, may be useful to parse the '/'-delimited categories and subcategories into additional related tables instead of a long string like "Home/Apparentl/Women's/Women's T-Shirts/"
| 15 | analytics | userid | - all rows are "NULL", suggest <b><font color="green">remove column</font></b> in cleaned table |
| 16 | | unitssold | - 97.8% of these entries are NULL, suggest <b><font color="red">caution</font></b> if using this column in analysis as it may not provide comprehensive view |
| 17 | | revenue | - very large numbers (largest is 6,252,750,000, smallest is 1,123,333)<br>- data should be <b><font color="yellow">standardized/scaled down</font></b> with division by 1,000,000 |
| 18 | | unit_price | - very large numbers (largest is 995,000,000, second smallest is 790,000)<br>- data should be <b><font color="yellow">standardized/scaled down</font></b> with division by 1,000,000 |
| 19 | products | name | - <b><font color="yellow">remove leading and trailing whitespace</font></b> from name, for consistency in data presentation and matches if joining products.name with allsessions.v2productname |


### When doing preliminary EDA to understand the 4 tables I imported, and attempting to confirm the relationships between the tables and their columns, I found these additional issues:

| Item # | Table Name | Column Name | Issue and Comments |
| ------ | ---------- | ----------- | ------------------ |
| 20 | salesbysku | productsku | - there are 8 productsku values in this table for whom no information exists in the products table.  There is a data integrity issue, which should be rectified by speaking to the business or data source to obtain product information for the missing productsku values.<br>- the rows associated to these missing productsku values, in salesbysku table, could be removed; however, an inner join as is typically used in analysis between salesbysku and products tables will automatically remove these rows, so it may not be necessary to physically remove these rows in a cleaned table |


# Queries:
# Below, provide the SQL queries you used to clean your data.

## QUERIES USED TO ACTION ITEMS IDENTIFIED ABOVE

Note:  I did not action all the items identified above (as indicated in the table above, some were highlighted simply as areas to exercise "<b><font color="red">caution</font></b>" over when running analyses).  However, below are the queries used to address the items from table above which were deemed issues to be addressed by data cleaning:

**Item #1**:
```SQL
CREATE TEMP TABLE allsessions_clean AS
SELECT *
FROM allsessions
WHERE country != '(not set)'
```

**Item #3, #4, #5, #6, #7**:
```SQL
-- Create allsessions_clean from selecting all columns EXCEPT:  [transactions, productrefundamount, productquantity, itemquantity, itemrevenue]
-- This may be unnecessary because we could just NOT use the above columns in any of our analysis SQL instead of creating a whole new temp table without these columns.

CREATE TEMP TABLE allsessions_clean AS (
    SELECT fullvisitorid, channelgrouping, time, country, city, totaltransactionrevenue, timeonsite, pageviews, sessionqualitydim, date, visitid, type, productprice, productrevenue, productsku, v2productname, v2productcategory, productvariant, currencycode, transactionrevenue, transactionid, pagetitle, searchkeyword, pagepathlevel1, ecommerceactiontype, ecommerceactionstep, ecommercectionoption
    FROM allsessions
)
```

**Item #10, #11, #12**:
```SQL
CREATE TEMP TABLE allsessions_clean AS (
	SELECT *,
	(productprice::real / 1000000) as productprice_clean,
	(productrevenue::real / 1000000) as productrevenue_clean,
    (transactionrevenue::real / 1000000) as transactionrevenue_clean
	FROM allsessions
)
```

**Item #13**:
```SQL
CREATE TEMP TABLE allsessions_clean AS (
	SELECT *,
	CASE
		WHEN pagetitle LIKE '%weixin://private/setresult/SCENE_FETCHQUEUE&eyJmdW5jIjoibG9nIiwicGFyYW1zIjp7Im1zZyI6Il9ydW5PbjNyZEFwaUxpc3QgOiBtZW51OnNoYXJlOnRpbWVsaW5lLG1lbnU6c2hhcmU6YXBwbWVzc2FnZSxvblZvaWNlUmVjb3JkRW5kLG9uVm9pY2VQbGF5QmVnaW4sb25Wb2ljZVBsYXlFbmQsb25Mb2NhbEltYWdlVXBsb2FkUHJvZ3Jlc3Msb25JbWFnZURvd25sb2FkUHJvZ3Jlc3Msb25Wb2ljZVVwbG9hZFByb2dyZXNzLG9uVm9pY2VEb3dubG9hZFByb2dyZXNzLG1lbnU6c2V0Zm9udCxtZW51OnNoYXJlOndlaWJvLG1lbnU6c2hhcmU6ZW1haWwsd3hkb3dubG9hZDpzdGF0ZV9jaGFuZ2UsaGRPbkRldmljZVN0YXRlQ2hhbmdlZCxhY3Rpdml0eTpzdGF0ZV9jaGFuZ2UifSwiX19tc2dfdHlwZSI6ImNhbGwiLCJfX2NhbGxiYWNrX2lkIjoiMTAwMCJ9%'
	 THEN NULL
		ELSE pagetitle
	END AS pagetitle_clean
	FROM allsessions
)
```

**Item #15**:
```SQL
-- Create analytics_clean from selecting all columns EXCEPT:  [userid]
-- This may be unnecessary because we could just NOT use the above columns in any of our analysis SQL instead of creating a whole new temp table without these columns.

CREATE TEMP TABLE analytics_clean AS (
    SELECT visitnumber, visitid, visitstarttime, date, fullvisitorid, channelgrouping, socialengagementtype, unitssold, pageviews, timeonsite, bounces, revenue, unit_price
    FROM analytics
)
```

**Item #17, #18**:
```SQL
CREATE TEMP TABLE analytics_clean AS (
	SELECT *,
	(revenue::real / 1000000) as revenue_clean,
	(unit_price::real / 1000000) as unit_price_clean
	FROM analytics
)
```

**Item #19**:
```SQL
CREATE TEMP TABLE products_clean AS (
	SELECT *,
    TRIM(BOTH ' ' FROM name) AS name_clean
    FROM products
)
```

**Item #20**:
```SQL
-- Discuss with the business to obtain information for all columns in the products table, for the following salesbysku.productsku values which do not have a matching entry in the products table:
SELECT DISTINCT productsku
FROM salesbysku s
WHERE s.productsku NOT IN (SELECT DISTINCT sku FROM products)
```