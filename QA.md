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

Checking Item #13 from cleaning_data.md:
```SQL
--Expect to see "[null]" in the first column, and the original long outlier pagetitle in the second column from the below after data-cleaning:
WITH tempresultset as (
	SELECT *,
	CASE
		WHEN pagetitle LIKE '%weixin://private/setresult/SCENE_FETCHQUEUE&eyJmdW5jIjoibG9nIiwicGFyYW1zIjp7Im1zZyI6Il9ydW5PbjNyZEFwaUxpc3QgOiBtZW51OnNoYXJlOnRpbWVsaW5lLG1lbnU6c2hhcmU6YXBwbWVzc2FnZSxvblZvaWNlUmVjb3JkRW5kLG9uVm9pY2VQbGF5QmVnaW4sb25Wb2ljZVBsYXlFbmQsb25Mb2NhbEltYWdlVXBsb2FkUHJvZ3Jlc3Msb25JbWFnZURvd25sb2FkUHJvZ3Jlc3Msb25Wb2ljZVVwbG9hZFByb2dyZXNzLG9uVm9pY2VEb3dubG9hZFByb2dyZXNzLG1lbnU6c2V0Zm9udCxtZW51OnNoYXJlOndlaWJvLG1lbnU6c2hhcmU6ZW1haWwsd3hkb3dubG9hZDpzdGF0ZV9jaGFuZ2UsaGRPbkRldmljZVN0YXRlQ2hhbmdlZCxhY3Rpdml0eTpzdGF0ZV9jaGFuZ2UifSwiX19tc2dfdHlwZSI6ImNhbGwiLCJfX2NhbGxiYWNrX2lkIjoiMTAwMCJ9%'
	 THEN NULL
		ELSE pagetitle
	END AS pagetitle_clean
	FROM allsessions
)
SELECT pagetitle_clean, pagetitle
FROM tempresultset
WHERE pagetitle LIKE '%weixin%'
```

POSSIBLE ADDS:
1. Confirm revenue_cleaned * 1000000 = revenue for every row for the 2 tables/4 columns that had scaling down done
2. Confirm dropped columns on 2 tables using information_schema.columns names against a hard-coded list
3. Confirm row count for salesbysku after dropping, is 8 skus less