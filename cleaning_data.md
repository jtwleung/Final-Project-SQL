# What issues will you address by cleaning the data?

## ISSUES IDENTIFIED DURING DATA INSPECTION

### When doing preliminary examination of the data to prepare for import, and analysis using SQL on the tables after import, I found the following potential issues:

(Note to reader: Please render this .md file in Visual Studio Code preview in order to see the color coding within the table.  GitHub does not support rendering 'font color="red"' tags.)

| Item # | Table Name | Column Name | Issue and Comments |
| ------ | ---------- | ----------- | ------------------ |
| 1 |allsessions | country | - 24 entries of '(not set)' = 0.15% occurence<br>- small percentage is negligible, so rows could be dropped if country distribution specifically is being examined, otherwise keep this column<br>- not possible to interpolate country value from city value due to missing city value data in those rows |
| 2 | | city | - '(not set)' value occuring 2.69% percent of time<br>- 'not available in demo dataset' occuring 63.11% of the time<br>- 66% of the dataset does not have city information, suggest <b><font color="red">caution</font></b> if using this column in analysis as it may not provide comprehensive view<br>- please see my cautionary notes within the starting_with_questions Q1 for some reasons/rationale why I did not clean this data |
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
| 14 | | v2productcategory | - depending on analysis scenario, may be useful to parse the '/'-delimited categories and subcategories into additional related tables instead of a long string like "Home/Apparentl/Women's/Women's T-Shirts/" |
| 15 | | v2productname | - there appear to be several social media brand names like "Google", "YouTube", "Waze" and "Android" which have corrupted this name and should be removed using a REGEXP_REPLACE command
| 15a | | pagepathlevel1 | - there are duplicate categories in these values that should be collapsed into a singular<br>- example 1: "/asearch.html" and "/asearch.html/" (set all to "/asearch.html")<br>- example 2: "/store.html" and "/store.html/ (set all to "/store.html")<br>- I also question whether the prevalent entry "/google+redesign/" (14318 rows) is corrupt data. This value could be set to NULL, but I will leave it as is
| 17 | analytics | userid | - all rows are "NULL", suggest <b><font color="green">remove column</font></b> in cleaned table |
| 18 | | unitssold | - 97.8% of these entries are NULL, suggest <b><font color="red">caution</font></b> if using this column in analysis as it may not provide comprehensive view |
| 19 | | revenue | - very large numbers (largest is 6,252,750,000, smallest is 1,123,333)<br>- data should be <b><font color="yellow">standardized/scaled down</font></b> with division by 1,000,000 |
| 20 | | unit_price | - very large numbers (largest is 995,000,000, second smallest is 790,000)<br>- data should be <b><font color="yellow">standardized/scaled down</font></b> with division by 1,000,000 |
| 21 | products | name | - <b><font color="yellow">remove leading and trailing whitespace</font></b> from name, for consistency in data presentation and matches if joining products.name with allsessions.v2productname |


### When doing preliminary EDA to understand the 4 tables I imported, and attempting to confirm the relationships between the tables and their columns, I found these additional issues:

| Item # | Table Name | Column Name | Issue and Comments |
| ------ | ---------- | ----------- | ------------------ |
| 22 | salesbysku | productsku | - there are 8 productsku values in this table for whom no information exists in the products table.  There is a data integrity issue, which should be rectified by speaking to the business or data source to obtain product information for the missing productsku values.<br>- the rows associated to these missing productsku values, in salesbysku table, could be removed; however, an inner join as is typically used in analysis between salesbysku and products tables will automatically remove these rows, so it may not be necessary to physically remove these rows in a cleaned table |


# Queries:
# Below, provide the SQL queries you used to clean your data.

## QUERIES USED TO ACTION ITEMS IDENTIFIED ABOVE

Note:  I did not action all the items identified above (as indicated in the table above, some were highlighted simply as areas to exercise "<b><font color="red">caution</font></b>" over when running analyses).  However, below are the queries used to address the items from table above which were deemed issues to be addressed by data cleaning.

\*** **Note to reader**:  Please note that the complete SQL used to create views for allsessions_clean, analytics_clean and products_clean, are **listed at the beginning of the QA.md file**.  Ultimately, I decided to approach the creation of the clean tables slightly differently than written below, after my initial analysis written in this file on a column by column basis.

For example, I created views instead of temp tables, and put the clean data back in the original column name, and the former data into a new column name with the "_original" appended.  (e.g. cleaned productquantity goes into column "productquantity" and the former value goes into column "productquantity_original").  This is a slightly different approach than I indicated in the individual steps listed below because once I began working with the data for analysis, I realized it was easier to run my analytical SQL queries with the original column names.  I left the original individual steps in this cleaning_data.md file as written below because breaking everything into individual steps was suggested by Brian Lynch, and in order to show that I know there are different approaches that can be taken to organizing cleaned data.  The below is simply individualized SQL to address each Item highlighted in the table above so the reader can follow along easily with my commentary in the table above, and one way to formulate the SQL to perform the cleaning, below.


**Item #1**:
```SQL
-- Create a new clean temporary table that does not have any rows where the country value is '(not set)'.

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
-- Create allsessions_clean temp table with all columns 'as is'.
-- 3 new columns are created for each of: productprice, productrevenue, and transactionrevenue.
-- The transformation is that all 3 columns will be divided by 1,000,000 and the result stored as a newly added columns productprice_clean, producerevenue_clean, transactionrevenue_clean, respectively

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
-- Create allsessions_clean temp table with all columns, with the exception of the 'pagetitle' column, 'as is'.
-- For the pagetitle column, for the 1 value with what appears to a data corruption of data that does not belong, replace that with "NULL".

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
-- Create allsessions_clean temp table with all columns 'as is'.
-- Add an additional column, v2productname_clean, which is a cleaned version of v2productname where any occurence of the words 'Google', 'YouTube', 'Waze' or 'Android' are removed.

CREATE TEMP TABLE allsessions_clean AS (
    SELECT *,
    REGEXP_REPLACE(v2productname, '(Google|YouTube|Waze|Android)( |$)', '', 'g') AS v2productname_clean
    FROM allsessions        
)
```

**Item #16**:
```SQL
-- Create allsessions_clean temp table with all columns 'as is'.
-- Add an additional column, pagepathlevel1_clean, which is a cleaned version of pagepathlevel1 where any occurence of the value '/asearch.html/' is replaced with '/asearch.html' and any occurence of the value '/store.html/' is replaced with '/store.html'.  This creates consistency in the format of the values and also de-duplicates values.


CREATE TEMP TABLE allsessions_clean AS (
	SELECT *,
	CASE
		WHEN pagepathlevel1 = '/asearch.html/' THEN '/asearch.html'
		WHEN pagepathlevel1 = '/store.html/' THEN '/store.html'
		ELSE pagepathlevel1
	END AS pagepathlevel1_clean
FROM allsessions
)
```

**Item #17**:
```SQL
-- Create analytics_clean from selecting all columns EXCEPT:  [userid]
-- This may be unnecessary because we could just NOT use the above columns in any of our analysis SQL instead of creating a whole new temp table without these columns.

CREATE TEMP TABLE analytics_clean AS (
    SELECT visitnumber, visitid, visitstarttime, date, fullvisitorid, channelgrouping, socialengagementtype, unitssold, pageviews, timeonsite, bounces, revenue, unit_price
    FROM analytics
)
```

**Item #19, #20**:
```SQL
-- Create analytics_clean temp table with all columns 'as is'.
-- 2 new columns are created for each of: revenue, unit_price.
-- The transformation is that both columns will be divided by 1,000,000 and the result stored as a newly added columns revenue_clean, unit_price_clean, respectively.

CREATE TEMP TABLE analytics_clean AS (
	SELECT *,
	(revenue::real / 1000000) as revenue_clean,
	(unit_price::real / 1000000) as unit_price_clean
	FROM analytics
)
```

**Item #21**:
```SQL
-- Create products_clean temp table with all columns 'as is'.
-- 1 new column called name_clean is created, in which leading and trailing whitespace is removed from the product name.

CREATE TEMP TABLE products_clean AS (
	SELECT *,
    TRIM(BOTH ' ' FROM name) AS name_clean
    FROM products
)
```

**Item #22**:
```SQL
-- Discuss with the business to obtain information for all columns in the products table, for the following salesbysku.productsku values which do not have a matching entry in the products table:

SELECT DISTINCT productsku
FROM salesbysku s
WHERE s.productsku NOT IN (SELECT DISTINCT sku FROM products)
```