What issues will you address by cleaning the data?

When preliminarily examining the data to get it ready for import, I found a number of issues:

1. allsessions.country has a 24 entries of '(not set)' over 15134 rows.  If requiring country information in analysis, this data would be incomplete.  However, such a small percentage (24/15134 = 0.15% occurence) is negligible and these rows could be dropped.  Unfortunately, it is not possible to interpolate the country value because the city value for all of these rows are either '(not set)' or 'not available in demo dataset'.

2. allsessions.city has '(not set)' (354 of 15134 rows = 2.69% occurence) and 'not available in demo dataset' values (8302 of 15134 rows = 63.11% occurence).  Given that ~66% of the dataset does not have city information, it may be worth dropping this column or otherwise not using this column in most of our analysis.
```SQL
select city, count(city), CAST((count(city)::real/13154::real)*100 AS numeric) as pctge from allsessions group by city order by city
```



# ISSUES IDENTIFIED DURING DATA INSPECTION

When doing preliminary examination of the data to prepare for import, and analysis using SQL on the tables after import, I found the following potential issues:

| Item # | Tablename | Column Name | Issue and Comments |
| ------ | --------- | ----------- | ------------------ |
| 1 |allsessions | country | - 24 entries of '(not set)' = 0.15% occurence<br>- small percentage is negligible, so rows could be dropped if country distribution specifically is being examined, otherwise keep this column<br>- not possible to interpolate country value from city value due to missing city value data in those rows |
| 2 | | city | - '(not set)' value occuring 2.69% percent of time<br>- 'not available in demo dataset' occuring 63.11% of the time<br>- 66% of the dataset does not have city information, suggest <b><font color="red">caution</font></b> if using this column in analysis as it may not provide comprehensive view |
| 3 | | transactions | - NULL 99.46% of the rows, and value of 1 in 0.54% of the rows<br>- suggests this column is not useful and should be <b><font color="green">removed</font></b> |
| 4 | | productrefundamount | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 5 | | productquantity | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table | 
| 6 | | itemquantity | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 7 | | itemrevenue | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 8 | | searchkeyword | - entire column empty, suggest <b><font color="green">remove column</font></b> in cleaned table |
| 9 | | productvariant | - 99.74% of rows are '(not set)', so there is mostly not much information in this column<br> - 0.26% have some information which may be useful information<br>- suggest keeping this column with <b><font color="red">caution</font></b> that data may not provide a comprehensive view |
| 10 | | productprice | - very large numbers (largest is 249,000,000, smallest is 0 and second smallest is 790,000)<br>- this data could probably use a division by 1,000,000 to <b><font color="yellow">scale it down</font></b> to more reasonable and relatable numbers |
| 11 | | productrevenue | - the largest (non-null) number is 176,400,000 and smallest (non-null) is 58,656,666<br>- probably should be <b><font color="yellow">scaled down</font></b> with division by 1,000,000 to make it relative to productprice |
| 12 | | transactionrevenue | -largest (non-null) number is 1,015,480,000 and smallest (non-null) is 169,970,000<br>- 99.97% are NULL, and only 4 unique numeric entries exist so use in large-scale analysis is limited<br>- this data should be <b><font color="yellow">scaled down</font></b> with division by 1,000,000 to make it relative to productprice and productrevenue |
| 13 | | pagetitle | - there is one entry for this column which appears to be an outlier which begins with "weixin://private/setresult/SCENE_FETCHQUEUE&"<br>- suggest <b><font color="yellow">setting this value to "NULL"</font></b> in a clean data set |

When doing preliminary EDA to understand the 4 tables I imported, and attempting to confirm the relationships between the tables and their columns, I found a number of additional issues:





Queries:
Below, provide the SQL queries you used to clean your data.

Did not action all the items, but here are the queries used to address the items from table above which were deemed an issue:

Item #1:
```SQL
CREATE TEMP TABLE allsessions_clean AS
SELECT *
FROM allsessions
WHERE country != '(not set)'
```

Item #3, #4, #5, #6, #7:
```SQL
-- Create allsessions_clean from selecting all columns EXCEPT:  [transactions, productrefundamount, productquantity, itemquantity, itemrevenue]
-- This may be unnecessary because we could just NOT use the above columns in any of our analysis SQL instead of creating a whole new temp table without these columns.

CREATE TEMP TABLE allsessions_clean AS (
    SELECT fullvisitorid, channelgrouping, time, country, city, totaltransactionrevenue, timeonsite, pageviews, sessionqualitydim, date, visitid, type, productprice, productrevenue, productsku, v2productname, v2productcategory, productvariant, currencycode, transactionrevenue, transactionid, pagetitle, searchkeyword, pagepathlevel1, ecommerceactiontype, ecommerceactionstep, ecommercectionoption
    FROM allsessions
)
```

Item #10, #11, #12:
```SQL
CREATE TEMP TABLE allsessions_clean AS (
	SELECT *,
	(productprice::real / 1000000) as productprice_clean,
	(productrevenue::real / 1000000) as productrevenue_clean
    (transactionrevenue::real / 1000000) as transactionrevenue_clean
	FROM allsessions
)
```

Item #13:
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
