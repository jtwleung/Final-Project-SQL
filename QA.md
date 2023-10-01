# What are your risk areas? Identify and describe them.

1. There is no "data dictionary" provided with this data and no "business" to interview to confirm context of the data.  Therefore, there is significant reliance upon Exploratory Data Analysis (EDA) and SQL queries with COUNT, DISTINCT and IN keywords, to confirm how these tables could be related to each other, what PK/FK relationships exist, and how the tables can be joined together to form insights.  For example, similarity in column names across tables (products.sku, allsessions.productsku, salesbysku.productsku) or (allsessions.fullvisitorid, analytics.visitid) or (allsessions.productquantity/productprice/productrevenue, salesbysku.total_ordered, analytics.revenue/unit_price) doesn't necessarily mean the data is consistent across all tables and thus can be joined to form realistic insights.

2. The significant amount of missing data/rows in some columns may be a problem 


# QA Process:
Describe your QA process and include the SQL queries used to execute it.

## My QA Process was in two parts:

### A. EDA-related QA to confirm relationships between tables and keys, to be confident in joining tables.

1. Confirming that products.sku and salesbysku.productsku are indeed the same keys.

Here is a sample of some SQL used to determine that at least 454 product.skus overlap/are listed in the 462 distinct rows of salesbysku, so the product.sku and salesbysku.productsku are likely the same key.  Please see my file "Prelim_Exploration_ecommerce.sql" in my GitHub project for more questions answered from SQL.
```SQL
--Q: How many of the 1092 products.sku are in the 462 salesbysku.productsku?  A: 454
--Q: Does 454 + 638 (products never sold) = 1092?  A: YES
SELECT count(*)
FROM products
WHERE sku IN (SELECT DISTINCT productsku FROM salesbysku)

--Q: Are there any skus in products that aren't in sales?  A: YES - 638 products (and distinct products) that have never been sold (barring checking the allsessions table)
SELECT *
FROM products p
WHERE p.sku NOT IN (SELECT DISTINCT productsku FROM salesbysku)

SELECT DISTINCT sku FROM products p
WHERE p.sku NOT IN (SELECT DISTINCT productsku FROM salesbysku)

```

2. Confirming that it would be fair to treat 'sku' as a PK in the products table, and 'productsku' as the PK in the salesbysku table:
```SQL
/*---For salesbysku table---*/
--how many distinct productsku rows?  462
select productsku, count(*) from salesbysku
group by productsku

--how many rows total?  Should be 462 if there are no NULL productskus.  YES, 462 total rows
select count(*) from salesbysku

--how many distinct productskus in salesbysku?  462 from 462 rows
select count(distinct productsku)
from salesbysku

--462 rows and 462 distinct productskus, which means each productsku is only represented once in this table
select distinct productsku from salesbysku

--any null productskus?  NO
select * from salesbysku
where productsku IS NULL or productsku = ''

/*---For products table---*/
--how many rows?  1092
select count(*) from products

--how many distinct skus in products table?  1092 - that means each row is unique
select distinct sku from products
select count(distinct sku) from products

--how many skus?  1092. This means no skus are null
select count(sku) from products

--any null skus?  NO
select * from products
where sku IS NULL or sku = ''

```

3. Confirming that there is missing data in products for 8 rows in salesbysku, so these rows may be a candidate for cleaning since they are missing any information about the product:
```SQL
--are there any productskus in salesbysku that aren't in products?  YES - 8 productskus of 462 distinct
select * from salesbysku s
where s.productsku NOT IN (select distinct sku from products)
```

### B. QA-related after data-cleaning, to ensure data-cleaning did not inadvertently destroy or distort data.

#### Related to creating allsessions_clean:

Ran the following SQL to create a new view "allsessions_clean" incorporating my cleaning steps for allsessions:
```SQL
CREATE OR REPLACE VIEW allsessions_clean AS (
	SELECT
	fullvisitorid,
	channelgrouping,
	time,
	country,
	city,
	totaltransactionrevenue,
	transactions,
	timeonsite,
	pageviews,
	sessionqualitydim,
	date,
	visitid,
	type,
	productrefundamount,
	productquantity,
	(productprice::real / 1000000) AS productprice,
	productprice AS productprice_original,  -- NEW COLUMN
	(productrevenue::real / 1000000) as productrevenue,
	productrevenue as productrevenue_original,  -- NEW COLUMN
	productsku,
	v2productname,
	v2productcategory,
	productvariant,
	currencycode,
	itemquantity,
	itemrevenue,
    (transactionrevenue::real / 1000000) as transactionrevenue,
	transactionrevenue as transactionrevenue_original,  -- NEW COLUMN
	transactionid,
	CASE
		WHEN pagetitle LIKE '%weixin://private/setresult/SCENE_FETCHQUEUE&eyJmdW5jIjoibG9nIiwicGFyYW1zIjp7Im1zZyI6Il9ydW5PbjNyZEFwaUxpc3QgOiBtZW51OnNoYXJlOnRpbWVsaW5lLG1lbnU6c2hhcmU6YXBwbWVzc2FnZSxvblZvaWNlUmVjb3JkRW5kLG9uVm9pY2VQbGF5QmVnaW4sb25Wb2ljZVBsYXlFbmQsb25Mb2NhbEltYWdlVXBsb2FkUHJvZ3Jlc3Msb25JbWFnZURvd25sb2FkUHJvZ3Jlc3Msb25Wb2ljZVVwbG9hZFByb2dyZXNzLG9uVm9pY2VEb3dubG9hZFByb2dyZXNzLG1lbnU6c2V0Zm9udCxtZW51OnNoYXJlOndlaWJvLG1lbnU6c2hhcmU6ZW1haWwsd3hkb3dubG9hZDpzdGF0ZV9jaGFuZ2UsaGRPbkRldmljZVN0YXRlQ2hhbmdlZCxhY3Rpdml0eTpzdGF0ZV9jaGFuZ2UifSwiX19tc2dfdHlwZSI6ImNhbGwiLCJfX2NhbGxiYWNrX2lkIjoiMTAwMCJ9%'
	 THEN NULL
		ELSE pagetitle
	END AS pagetitle,
	pagetitle AS pagetitle_original,  -- NEW COLUMN
	searchkeyword,
	pagepathlevel1,
	ecommerceactiontype,
	ecommerceactionstep,
	ecommerceactionoption
	FROM allsessions
)
```

Ran the following SQL for QA on the new allsessions_clean view:
```SQL
-- 1. Confirm that the productprice is just productprice_original divided by 1,000,000 for all rows that productprice_original isn't 0 (in which case they should be equal, and both 0)
-- The below should give NULL SET:
SELECT productprice, productprice_original
FROM allsessions_clean
WHERE (productprice_original != 0 AND productprice::real != (productprice_original::real/1000000)::real) OR
(productprice_original = 0 AND productprice != productprice_original AND productprice = 0)

--2. Confirm that the producerevenue is just productrevenue_original divided by 1,000,000 for all rows that productrevenue_original IS NOT NULL (in which case, they should be equal, and both NULL)
-- The below should give NULL SET:
SELECT productrevenue, productrevenue_original
FROM allsessions_clean
WHERE (productrevenue_original IS NOT NULL AND productrevenue::real != (productrevenue_original::real/1000000)::real) OR
(productrevenue_original IS NULL AND productrevenue IS NOT NULL)

--3. Confirm that pagetitle_original is always equal to pagetitle except when pagetitle_original contains 'weixin', pagetitle is NULL:
-- The below should give NULL SET:
SELECT pagetitle, pagetitle_original
FROM allsessions_clean
WHERE (pagetitle_original LIKE '%weixin%' AND pagetitle IS NOT NULL) OR
(pagetitle != pagetitle_original)

--4. Confirm same number/name of columns in allsessions_clean as allsessions
-- The following should return ONLY the 4 "new" columns in the allsessions_clean VIEW:  productprice_original, productrevenue_original, transactionrevenue_original, pagetitle_original
WITH original_columns AS (
	SELECT column_name
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'allsessions'
),

new_columns AS (
	SELECT column_name
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'allsessions_clean'
)

SELECT *
FROM original_columns oc
FULL OUTER JOIN new_columns nc
ON oc.column_name = nc.column_name
WHERE oc.column_name IS NULL OR nc.column_name IS NULL


--5. Confirm the same order of columns from allsessions and allsessions_clean
-- The below should return NULL SET
WITH original_columns AS (
	SELECT ordinal_position, column_name,
	ROW_NUMBER() OVER (ORDER BY ordinal_position) AS row_num
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'allsessions'
),

new_columns AS (
	SELECT ordinal_position, column_name,
	ROW_NUMBER() OVER (ORDER BY ordinal_position) AS row_num
	FROM information_schema.columns
	WHERE table_schema = 'public' and column_name NOT IN ('productprice_original', 'productrevenue_original', 'transactionrevenue_original', 'pagetitle_original')
	AND table_name = 'allsessions_clean'
)

SELECT *
FROM original_columns oc
FULL OUTER JOIN new_columns nc
ON oc.column_name = nc.column_name
WHERE oc.row_num != nc.row_num
```


#### Related to creating analytics_clean:

Ran the following SQL for QA on the new analytics_clean view:
```SQL
CREATE OR REPLACE VIEW analytics_clean AS (
	SELECT
	visitnumber,
	visitid,
	visitstarttime,
	date,
	fullvisitorid,
	userid,
	channelgrouping,
	socialengagementtype,
	unitssold,
	pageviews,
	timeonsite,
	bounces,
	(revenue::real / 1000000) AS revenue,
	revenue AS revenue_original, --NEW COLUMN
	(unit_price::real / 1000000) as unit_price,
	unit_price AS unit_price_original --NEW COLUMN
	FROM analytics
)
```

Ran the following SQL for QA on the new analytics_clean view:

```SQL
--1. Confirm that the revenue is just the revenue_original divided by 1,000,000, but where revenue_original was NULL, revenue should be NULL, and where revenue_original was 0, revenue should be 0
-- The below should give NULL SET:
SELECT revenue, revenue_original
FROM analytics_clean
WHERE (revenue IS NOT NULL AND revenue != 0 AND revenue::real != (revenue_original::real/1000000)::real) OR
(revenue_original = 0 AND revenue != revenue_original AND revenue = 0)

--2. Confirm that the unit_price is just the unit_price_original divided by 1,000,000, but where unit_price_original was NULL, unit_price should be NULL, and where unit_price_original was 0, unit_price should be 0
-- The below should give NULL SET:
SELECT unit_price, unit_price_original
FROM analytics_clean
WHERE (unit_price IS NOT NULL AND unit_price != 0 AND unit_price::real != (unit_price_original::real/1000000)::real) OR
(unit_price_original = 0 AND unit_price != unit_price_original AND unit_price = 0)

--3. Confirm same number/name of columns in analytics_clean as analytics
-- The following should return ONLY the 2 "new" columns in the analytics_clean VIEW:  revenue_original, unit_price_original
WITH original_columns AS (
	SELECT column_name
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'analytics'
),

new_columns AS (
	SELECT column_name
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'analytics_clean'
)

SELECT *
FROM original_columns oc
FULL OUTER JOIN new_columns nc
ON oc.column_name = nc.column_name
WHERE oc.column_name IS NULL OR nc.column_name IS NULL

--4. Confirm the same order of columns from analytics and analytics_clean
-- The below should return NULL SET
WITH original_columns AS (
	SELECT ordinal_position, column_name,
	ROW_NUMBER() OVER (ORDER BY ordinal_position) AS row_num
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'analytics'
),

new_columns AS (
	SELECT ordinal_position, column_name,
	ROW_NUMBER() OVER (ORDER BY ordinal_position) AS row_num
	FROM information_schema.columns
	WHERE table_schema = 'public' and column_name NOT IN ('revenue_original', 'unit_price_original')
	AND table_name = 'analytics_clean'
)

SELECT *
FROM original_columns oc
FULL OUTER JOIN new_columns nc
ON oc.column_name = nc.column_name
WHERE oc.row_num != nc.row_num
```

#### Related to creating products_clean:

Ran the following SQL for QA on the new products_clean view:
```SQL
CREATE OR REPLACE VIEW products_clean AS (
	SELECT
	sku,
    TRIM(BOTH ' ' FROM name) AS name,
	name AS name_original, -- NEW COLUMN
	orderedquantity,
	stocklevel,
	restockingleadtime,
	sentimentscore,
	sentimentmagnitude
    FROM products
)
```

Ran the following SQL for QA on the new products_clean view:

```SQL
--1. Confirm that the name is just the TRIM (leading and trailing whitespace) of name_original
-- THe below should give NULL SET:
SELECT name, name_original
FROM products_clean
WHERE (name != TRIM(BOTH ' ' FROM name_original))

--2. Confirm same number/name of columns as products
-- The following should return ONLY the 1 "new" column in the products_clean VIEW:  name_original
WITH original_columns AS (
	SELECT column_name
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'products'
),

new_columns AS (
	SELECT column_name
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'products_clean'
)

SELECT *
FROM original_columns oc
FULL OUTER JOIN new_columns nc
ON oc.column_name = nc.column_name
WHERE oc.column_name IS NULL OR nc.column_name IS NULL

--3. Confirm the same order of columns from products and productss_clean
-- The below should return NULL SET
WITH original_columns AS (
	SELECT ordinal_position, column_name,
	ROW_NUMBER() OVER (ORDER BY ordinal_position) AS row_num
	FROM information_schema.columns
	WHERE table_schema = 'public'
	AND table_name = 'products'
),

new_columns AS (
	SELECT ordinal_position, column_name,
	ROW_NUMBER() OVER (ORDER BY ordinal_position) AS row_num
	FROM information_schema.columns
	WHERE table_schema = 'public' and column_name NOT IN ('name_original')
	AND table_name = 'products_clean'
)

SELECT *
FROM original_columns oc
FULL OUTER JOIN new_columns nc
ON oc.column_name = nc.column_name
WHERE oc.row_num != nc.row_num
```




#### No changes were needed on the salesbysku table.







POSSIBLE ADDS:
3. Confirm row count for salesbysku after dropping, is 8 skus less
4. Reverse the order of the top EDA section, put it at the bottom after the QA on the data-cleaning