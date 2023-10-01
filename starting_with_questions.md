Answer the following questions and provide the SQL queries used to find the answer.

    
**Question 1: Which cities and countries have the highest level of transaction revenues on the site?**

Assumptions:
1. The allsessions_clean table has information about all sessions begun on our ecommerce site, regardless of whether or not the consumer made a purchase (perhaps they were just browsing), and this is the only table with city and country information.
2. The analytics_clean table is the only table that has a recording of how many units were sold in a given session (signified by visitid).
3. The analytics_clean.unit_price is the unit price of what was sold.  Therefore the analytics_clean.unitssold * analytics_clean.unit_price is the total revenue for that particular visitid.
4. Then we can LEFT JOIN the analytics_clean table with the allsessions_clean table USING visitid.  This will give us City and Country information from allsessions_clean where there is a matching visitid, otherwise there will be NULL columns from allsessions_clean, but we still preserve the sales revenue from analytics_clean table and can treat that as an "Unknown" city/country.
5. We can use the resulting LEFT JOIN to do some aggregations to see revenue per city/country, keeping in mind that not all cities are going to be filled in from allsessions_clean.

SQL Queries:
```SQL
WITH analytics_rows_with_sold AS (
	SELECT *
	FROM analytics_clean
	WHERE unitssold IS NOT NULL AND unitssold > 0
),

joined_sold_revenue AS (
	SELECT *,
	(unit_price * unitssold) AS visit_revenue
	FROM analytics_rows_with_sold arws
	LEFT JOIN allsessions_clean allsc
	ON arws.visitid = allsc.visitid
),

city_country_revenue AS (
	SELECT city, country, visit_revenue
	FROM joined_sold_revenue
),

location_revenue AS (
	SELECT
	CASE
		WHEN city IS NULL AND country IS NULL THEN 'Unknown, Unknown'
		WHEN city = 'not available in demo dataset' OR city = '(not set)' THEN 'Anywhere, ' || country
		ELSE city || ', ' || country
	END AS location,
	visit_revenue AS revenue
	FROM city_country_revenue
),

location_grouped_revenue AS (
	SELECT location,
	SUM(revenue) AS location_total_revenue
	FROM location_revenue
	GROUP BY location
)

SELECT location,
ROUND(location_total_revenue::numeric, 2) as location_total_revenue,
RANK() OVER(ORDER BY location_total_revenue DESC) AS rank_highest_to_lowest_revenue
FROM location_grouped_revenue
```


Answer:
Output from pgAdmin:

| location                     | location_total_revenue | rank_highest_to_lowest_revenue |
|------------------------------|------------------------|--------------------------------|
| Unknown, Unknown             | 38125267.5             | 1                              |
| New York, United States      | 11378.77               | 2                              |
| Anywhere, United States      | 9330.75                | 3                              |
| Sunnyvale, United States     | 6787.81                | 4                              |
| Mountain View, United States | 4690.77                | 5                              |
| Chicago, United States       | 2941.73                | 6                              |
| Seattle, United States       | 1959.99                | 7                              |
| San Francisco, United States | 1672.68                | 8                              |
| Palo Alto, United States     | 1107.98                | 9                              |
| San Jose, United States      | 765                    | 10                             |
| Anywhere, Sweden             | 654.99                 | 11                             |
| Pittsburgh, United States    | 539.64                 | 12                             |
| Anywhere, Mexico             | 494.99                 | 13                             |
| Anywhere, Canada             | 323.45                 | 14                             |
| Dublin, Ireland              | 311.96                 | 15                             |
| London, United Kingdom       | 281.97                 | 16                             |
| Anywhere, Japan              | 230.17                 | 17                             |
| San Bruno, United States     | 217.47                 | 18                             |
| Santiago, Chile              | 199.98                 | 19                             |
| Anywhere, Maldives           | 199.98                 | 19                             |
| Anywhere, Hong Kong          | 179.98                 | 21                             |
| Toronto, Canada              | 107.93                 | 22                             |
| Anywhere, Netherlands        | 103.95                 | 23                             |
| Anywhere, Indonesia          | 99.99                  | 24                             |
| Tel Aviv-Yafo, Israel        | 93.6                   | 25                             |
| Atlanta, United States       | 89.98                  | 26                             |
| Anywhere, Egypt              | 84.42                  | 27                             |
| Bangkok, Thailand            | 74.97                  | 28                             |
| Zurich, Switzerland          | 68.95                  | 29                             |
| Ann Arbor, United States     | 54.97                  | 30                             |
| Anywhere, Belgium            | 49.98                  | 31                             |
| Washington, United States    | 37.99                  | 32                             |
| Bogota, Colombia             | 33.98                  | 33                             |
| Anywhere, Vietnam            | 33.98                  | 33                             |
| Anywhere, India              | 33.96                  | 35                             |
| Kirkland, United States      | 30.99                  | 36                             |
| Anywhere, Taiwan             | 29.98                  | 37                             |
| Austin, United States        | 28.58                  | 38                             |
| Anywhere, Bulgaria           | 26.95                  | 39                             |
| Anywhere, Denmark            | 24.99                  | 40                             |
| Dallas, United States        | 20.97                  | 41                             |
| Hong Kong, Hong Kong         | 18.99                  | 42                             |
| Paris, France                | 17.99                  | 43                             |
| Houston, United States       | 17.5                   | 44                             |
| Anywhere, France             | 17.49                  | 45                             |
| Anywhere, Switzerland        | 16.99                  | 46                             |
| Anywhere, Austria            | 12.99                  | 47                             |
| Berlin, Germany              | 12.99                  | 47                             |
| Munich, Germany              | 10.99                  | 49                             |
| Detroit, United States       | 8.97                   | 50                             |
| Singapore, Singapore         | 7.6                    | 51                             |
| Hyderabad, India             | 6.5                    | 52                             |
| Anywhere, Thailand           | 6                      | 53                             |
| Anywhere, Romania            | 5.98                   | 54                             |
| Anywhere, Australia          | 4.49                   | 55                             |
| Anywhere, South Korea        | 2.99                   | 56                             |



**Question 2: What is the average number of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?**


SQL Queries:



Answer:





**Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?**


SQL Queries:



Answer:





**Question 5: Can we summarize the impact of revenue generated from each city/country?**

SQL Queries:



Answer:







