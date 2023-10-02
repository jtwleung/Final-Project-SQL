Answer the following questions and provide the SQL queries used to find the answer.

    
# Question 1: Which cities and countries have the highest level of transaction revenues on the site?

Assumptions:
1. The allsessions_clean table has information about all sessions begun on our ecommerce site, regardless of whether or not the consumer made a purchase (perhaps they were just browsing), and this is the only table with city and country information.
2. The analytics_clean table is the only table that has a recording of how many units were sold in a given session (signified by visitid).
3. The analytics_clean.unit_price is the unit price of what was sold.  Therefore the analytics_clean.unitssold * analytics_clean.unit_price is the total revenue for that particular visitid.
4. Then we can LEFT JOIN the analytics_clean table with the allsessions_clean table USING visitid.  This will give us City and Country information from allsessions_clean where there is a matching visitid, otherwise there will be NULL columns from allsessions_clean, but we still preserve the sales revenue from analytics_clean table and can treat that as an "Unknown" city/country.
5. We can use the resulting LEFT JOIN to do some aggregations to see revenue per city/country, keeping in mind that not all cities are going to be filled in from allsessions_clean.

### SQL Queries:
```SQL
WITH analytics_rows_with_sold AS (
	SELECT *
	FROM analytics_clean
	WHERE unitssold IS NOT NULL AND unitssold > 0
),

joined_sold_revenue AS (
	SELECT  *,
	        (unit_price * unitssold) AS visit_revenue
	FROM analytics_rows_with_sold arws
	LEFT JOIN allsessions_clean allsc
	ON arws.visitid = allsc.visitid
),

city_country_revenue AS (
	SELECT
        city,
        country,
        visit_revenue
	FROM joined_sold_revenue
),

location_revenue AS (
	SELECT
	    CASE
		    WHEN city IS NULL AND country IS NULL THEN 'Unknown, Unknown'
		    WHEN city = 'not available in demo dataset' OR city = '(not set)' THEN 'Cities in ' || country
		    ELSE city || ', ' || country
	    END AS location,
	    visit_revenue AS revenue
	FROM city_country_revenue
),

location_grouped_revenue AS (
	SELECT
        location,
	    SUM(revenue) AS location_total_revenue
	FROM location_revenue
	GROUP BY location
)

SELECT
    location,
    ROUND(location_total_revenue::numeric, 2) as location_total_revenue,
    RANK() OVER(ORDER BY location_total_revenue DESC) AS rank_highest_to_lowest_revenue
FROM location_grouped_revenue
```


## Answer:

The result set for the above set of queries shows that by far, the highest ranking revenue is generated from "unknown" location, which exists when the city and country data is NULL.  This result is not reliable because numerous places from around the world could make up this result.  It is perhaps more useful to look at the result set from rank 2 onward.  New York, United States is the city/country combination that generated the highest revenue according to my assumptions about how to join the tables.  The "Cities in [country]" entries may also be unreliable to form conclusions on, because this means the city had 'not available in demo dataset' or '(not set)', which means it could have been any city in the country.  The third highest revenue generator is "Cities in United States", but this could have been comprised of numerous cities, and is probably not fair to compare to a specific city like "Sunnyvale, United States" which is 4th highest.  If the question wasn't worded to request inclusion of city/country pairs, a potentially more useful result set is one that looks at countries alone, and groups all findings within that country, together into the total_revenue value.

One other thing I will point out is that the city, country data wasn't cleaned.  It is very hard to clean accurately.  For example, there are entries for "Toronto, United States" and also "Vancouver, United States".  We may presume that "Toronto, United States" is incorrect - but do we clean that data to be "Toronto, Canada" or "Unknown city, United States"?  Furthermore, some people might assume that "Vancouver, United States" is also incorrect and needs to be cleaned -- but there is, in fact, a city called Vancouver in Washingston State, and that does belong in the United States.  Without a business person, or additional data to drill down into to know how to clean the city/country data, any attempts to clean the data could introduce even more errors; hence I have left this data and the result set including this potentially flawed data, as is.

The result set was:

| location                     | location_total_revenue | rank_highest_to_lowest_revenue |
|------------------------------|------------------------|--------------------------------|
| Unknown, Unknown             | 38125267.5             | 1                              |
| New York, United States      | 11378.77               | 2                              |
| Cities in United States      | 9330.75                | 3                              |
| Sunnyvale, United States     | 6787.81                | 4                              |
| Mountain View, United States | 4690.77                | 5                              |
| Chicago, United States       | 2941.73                | 6                              |
| Seattle, United States       | 1959.99                | 7                              |
| San Francisco, United States | 1672.68                | 8                              |
| Palo Alto, United States     | 1107.98                | 9                              |
| San Jose, United States      | 765                    | 10                             |
| Cities in Sweden             | 654.99                 | 11                             |
| Pittsburgh, United States    | 539.64                 | 12                             |
| Cities in Mexico             | 494.99                 | 13                             |
| Cities in Canada             | 323.45                 | 14                             |
| Dublin, Ireland              | 311.96                 | 15                             |
| London, United Kingdom       | 281.97                 | 16                             |
| Cities in Japan              | 230.17                 | 17                             |
| San Bruno, United States     | 217.47                 | 18                             |
| Santiago, Chile              | 199.98                 | 19                             |
| Cities in Maldives           | 199.98                 | 19                             |
| Cities in Hong Kong          | 179.98                 | 21                             |
| Toronto, Canada              | 107.93                 | 22                             |
| Cities in Netherlands        | 103.95                 | 23                             |
| Cities in Indonesia          | 99.99                  | 24                             |
| Tel Aviv-Yafo, Israel        | 93.6                   | 25                             |
| Atlanta, United States       | 89.98                  | 26                             |
| Cities in Egypt              | 84.42                  | 27                             |
| Bangkok, Thailand            | 74.97                  | 28                             |
| Zurich, Switzerland          | 68.95                  | 29                             |
| Ann Arbor, United States     | 54.97                  | 30                             |
| Cities in Belgium            | 49.98                  | 31                             |
| Washington, United States    | 37.99                  | 32                             |
| Cities in Vietnam            | 33.98                  | 33                             |
| Bogota, Colombia             | 33.98                  | 33                             |
| Cities in India              | 33.96                  | 35                             |
| Kirkland, United States      | 30.99                  | 36                             |
| Cities in Taiwan             | 29.98                  | 37                             |
| Austin, United States        | 28.58                  | 38                             |
| Cities in Bulgaria           | 26.95                  | 39                             |
| Cities in Denmark            | 24.99                  | 40                             |
| Dallas, United States        | 20.97                  | 41                             |
| Hong Kong, Hong Kong         | 18.99                  | 42                             |
| Paris, France                | 17.99                  | 43                             |
| Houston, United States       | 17.5                   | 44                             |
| Cities in France             | 17.49                  | 45                             |
| Cities in Switzerland        | 16.99                  | 46                             |
| Cities in Austria            | 12.99                  | 47                             |
| Berlin, Germany              | 12.99                  | 47                             |
| Munich, Germany              | 10.99                  | 49                             |
| Detroit, United States       | 8.97                   | 50                             |
| Singapore, Singapore         | 7.6                    | 51                             |
| Hyderabad, India             | 6.5                    | 52                             |
| Cities in Thailand           | 6                      | 53                             |
| Cities in Romania            | 5.98                   | 54                             |
| Cities in Australia          | 4.49                   | 55                             |
| Cities in South Korea        | 2.99                   | 56                             |



# Question 2: What is the average number of products ordered from visitors in each city and country?

**Solution Note**:  I am assuming we need to start with the allsessions table as our base, since the question is asking for "visitors", and also because allsessions is the only table with city and country data.  The number of products ordered is more difficult to obtain, because there is allsessions.unitssold, but filtering for rows where unitssold is > 0 OR NOT NULL cuts out more than 3M rows of usable data, and gives us mostly tiny averages of 1.xxx.  Therefore, I decided to join the allsessions data to the salesbysku via SKU, which gives an total_ordered value by SKU.  I also decided to join that end data set to the products table, which also provides a orderedquantity value for each SKU.  I made sure to do a left join in each case, so I would get the maximum number of rows (and preserve the maximum number of rows from allsessions, too).  I then went about "cleaning" data even within my set of queries, as you'll see below.  I decided that I would use salesbysku.total_ordered as the primary "total product ordered" number I would base my calculations on, and where that value is NULL or 0, I would then defer to using the products.orderedquantity.  If they were both NULL, I just used 0 as the ordered quantity (because we can't know, or guess).  I then went about doing some data cleaning right inside my queries, to deal with the missing data in city and country, before using the cleaned city/country data as my GROUP BY in the final query which should show the average products ordered in each city, country combination, to the best "data cleaning" and "assumptions" guesses I could do.

### SQL Queries:
```SQL
WITH city_country_sordered_pordered AS (
	SELECT
		city,
		country,
		alls.productsku AS sessions_sku,
		s.productsku AS sales_sku,
		s.total_ordered AS sales_ordered,
		p.sku AS products_sku,
		p.orderedquantity as products_ordered
    FROM allsessions_clean alls
    LEFT JOIN salesbysku s
    ON alls.productsku = s.productsku
    LEFT JOIN products_clean p
    ON alls.productsku = p.sku
)
,
city_country_sku_clean_ordered AS (
	SELECT
		city,
		country,
		sessions_sku,
		CASE
			WHEN sales_ordered IS NOT NULL AND sales_ordered > 0 THEN sales_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NOT NULL THEN products_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NULL THEN 0::int
			ELSE 0::int
		END AS ordered
	FROM city_country_sordered_pordered
)
,
clean_city_country_sku_clean_ordered AS (
	SELECT
		CASE
			WHEN (city IS NULL OR city = '(not set)' OR city = 'not available in demo dataset') AND (country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset') THEN 'Unknown'
			WHEN city = '(not set)' OR city = 'not available in demo dataset' THEN 'Somewhere in ' || country
			ELSE city
		END AS city,
		CASE
			WHEN country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset' THEN 'Unknown'
			ELSE country
		END AS country,
		sessions_sku,
		ordered
	FROM city_country_sku_clean_ordered
)

SELECT DISTINCT
	city,
	country,
	AVG(ordered) OVER (PARTITION BY city, country) AS average_products_ordered
FROM clean_city_country_sku_clean_ordered
ORDER BY city, country
```


## Answer:

The result set obtained by this query is as follows:

| city                              | country              | average_products_ordered |
|-----------------------------------|----------------------|--------------------------|
| Adelaide                          | Australia            | 3                        |
| Ahmedabad                         | India                | 182.9166667              |
| Akron                             | United States        | 13                       |
| Alexandria                        | Egypt                | 0                        |
| Amã                               | Jordan               | 0                        |
| Amsterdam                         | Netherlands          | 24.09090909              |
| Amsterdam                         | United States        | 0                        |
| Ann Arbor                         | United States        | 19.36                    |
| Antalya                           | Turkey               | 0                        |
| Antwerp                           | Belgium              | 1.5                      |
| Appleton                          | United States        | 66                       |
| Ashburn                           | United States        | 4                        |
| Asuncion                          | Paraguay             | 13                       |
| Athens                            | Greece               | 44.6                     |
| Atlanta                           | United States        | 45.77272727              |
| Auckland                          | New Zealand          | 0                        |
| Austin                            | United States        | 40.34259259              |
| Avon                              | United States        | 100                      |
| Bandung                           | Indonesia            | 6                        |
| Bangkok                           | Thailand             | 88.30555556              |
| Bangkok                           | United States        | 0                        |
| Barcelona                         | Spain                | 16.55555556              |
| Beijing                           | China                | 0                        |
| Belgrade                          | Serbia               | 0                        |
| Bellflower                        | United States        | 30                       |
| Bellingham                        | United States        | 62                       |
| Belo Horizonte                    | Brazil               | 43                       |
| Bengaluru                         | India                | 53.66216216              |
| Berkeley                          | United States        | 25                       |
| Berlin                            | Germany              | 87.6                     |
| Bhubaneswar                       | India                | 499                      |
| Bilbao                            | Spain                | 13.5                     |
| Bogota                            | Colombia             | 21.95                    |
| Bordeaux                          | France               | 0                        |
| Boston                            | United States        | 30.26923077              |
| Boulder                           | United States        | 4                        |
| Bratislava                        | Slovakia             | 0                        |
| Brisbane                          | Australia            | 97.16666667              |
| Brno                              | Czechia              | 172                      |
| Brussels                          | Belgium              | 4                        |
| Bucharest                         | Romania              | 8.230769231              |
| Budapest                          | Hungary              | 13.16666667              |
| Buenos Aires                      | Argentina            | 11.22222222              |
| Burnaby                           | Canada               | 10.33333333              |
| Calgary                           | Canada               | 239.8                    |
| Cambridge                         | United States        | 10.72727273              |
| Cape Town                         | South Africa         | 25                       |
| Chandigarh                        | India                | 13                       |
| Charlotte                         | United States        | 16.94444444              |
| Charlottetown                     | Canada               | 0                        |
| Chennai                           | India                | 131.4520548              |
| Chicago                           | United States        | 31.12562814              |
| Chico                             | United States        | 0.666666667              |
| Chiyoda                           | Japan                | 0                        |
| Chuo                              | Japan                | 0                        |
| Cluj-Napoca                       | Romania              | 4                        |
| Colombo                           | Sri Lanka            | 3.5                      |
| Columbia                          | United States        | 5                        |
| Columbus                          | United States        | 8.5                      |
| Copenhagen                        | Denmark              | 8                        |
| Cork                              | Ireland              | 30                       |
| Council Bluffs                    | United States        | 1.5                      |
| Courbevoie                        | France               | 7.5                      |
| Coventry                          | United Kingdom       | 0                        |
| Culiacan                          | Mexico               | 169                      |
| Cupertino                         | United States        | 12.46153846              |
| Dallas                            | United States        | 84.29411765              |
| Denver                            | United States        | 5.333333333              |
| Detroit                           | United States        | 9.25                     |
| Doha                              | Qatar                | 4                        |
| Druid Hills                       | United States        | 3                        |
| Dubai                             | United Arab Emirates | 61.11111111              |
| Dublin                            | Ireland              | 71.03703704              |
| Dublin                            | Netherlands          | 12                       |
| East Lansing                      | United States        | 7                        |
| Eau Claire                        | United States        | 4.2                      |
| Edmonton                          | Canada               | 0.666666667              |
| Egham                             | United Kingdom       | 50                       |
| El Paso                           | United States        | 2                        |
| Esbjerg                           | Denmark              | 0                        |
| Forest Park                       | United States        | 0                        |
| Fort Worth                        | United States        | 0                        |
| Fortaleza                         | Brazil               | 0                        |
| Frankfurt                         | Germany              | 7                        |
| Fremont                           | United States        | 57                       |
| Ghent                             | Belgium              | 16                       |
| Glasgow                           | United Kingdom       | 91.5                     |
| Goose Creek                       | United States        | 0                        |
| Gothenburg                        | Sweden               | 0                        |
| Greer                             | United States        | 1                        |
| Gurgaon                           | India                | 12.1                     |
| Hamburg                           | Germany              | 116                      |
| Hanoi                             | Vietnam              | 28.25                    |
| Hayward                           | United States        | 0                        |
| Helsinki                          | Finland              | 6                        |
| Ho Chi Minh City                  | Vietnam              | 11.5                     |
| Hong Kong                         | Hong Kong            | 49.0754717               |
| Hong Kong                         | United States        | 11                       |
| Houston                           | United States        | 35.10909091              |
| Hyderabad                         | India                | 52.15517241              |
| Iasi                              | Romania              | 2                        |
| Indianapolis                      | United States        | 0                        |
| Indore                            | India                | 257.2                    |
| Ipoh                              | Malaysia             | 11                       |
| Irvine                            | United States        | 18.9                     |
| Istanbul                          | Hungary              | 11                       |
| Istanbul                          | Turkey               | 68.03703704              |
| Izmir                             | Turkey               | 3.5                      |
| Jacksonville                      | United States        | 252                      |
| Jaipur                            | India                | 7.333333333              |
| Jakarta                           | Indonesia            | 53.66666667              |
| Jersey City                       | United States        | 1.6                      |
| Johnson City                      | United States        | 0                        |
| Kalamazoo                         | United States        | 52.5                     |
| Kansas City                       | United States        | 0.666666667              |
| Karachi                           | Pakistan             | 233.75                   |
| Kharagpur                         | India                | 6                        |
| Kharkiv                           | Ukraine              | 2                        |
| Kiev                              | Ukraine              | 68.2                     |
| Kirkland                          | United States        | 12.06666667              |
| Kitchener                         | Canada               | 15.66666667              |
| Kolkata                           | India                | 70.5                     |
| Kovrov                            | Russia               | 58                       |
| Krakow                            | Poland               | 0                        |
| Kuala Lumpur                      | Malaysia             | 116.8823529              |
| La Victoria                       | Peru                 | 10.5                     |
| LaFayette                         | United States        | 13                       |
| Lahore                            | Pakistan             | 1                        |
| Lake Oswego                       | United States        | 8.4                      |
| Las Vegas                         | United States        | 2                        |
| Lisbon                            | Portugal             | 189                      |
| London                            | United Kingdom       | 48.66666667              |
| London                            | United States        | 8.333333333              |
| Longtan District                  | Taiwan               | 97                       |
| Los Angeles                       | Australia            | 3                        |
| Los Angeles                       | United States        | 32.04072398              |
| Lucknow                           | India                | 0                        |
| Madison                           | United States        | 3                        |
| Madrid                            | Spain                | 152.15                   |
| Makati                            | Philippines          | 0.666666667              |
| Manchester                        | United Kingdom       | 0                        |
| Manila                            | Philippines          | 24.2                     |
| Maracaibo                         | Venezuela            | 0.75                     |
| Marlboro                          | United States        | 0                        |
| Marseille                         | France               | 2                        |
| Medellin                          | Colombia             | 7                        |
| Melbourne                         | Australia            | 71.89473684              |
| Menlo Park                        | United States        | 26.75                    |
| Mexico City                       | Mexico               | 74.65517241              |
| Mexico City                       | United States        | 0                        |
| Milan                             | Italy                | 19.71428571              |
| Milpitas                          | United States        | 5                        |
| Minato                            | Japan                | 69.19512195              |
| Minneapolis                       | United States        | 0                        |
| Mississauga                       | Canada               | 21.4                     |
| Monterrey                         | Mexico               | 393                      |
| Montevideo                        | Uruguay              | 7.5                      |
| Montreal                          | Canada               | 58.36585366              |
| Montreuil                         | France               | 23                       |
| Moscow                            | Russia               | 8.473684211              |
| Mountain View                     | Australia            | 1                        |
| Mountain View                     | Japan                | 7                        |
| Mountain View                     | United States        | 30.62978723              |
| Mumbai                            | India                | 64.44444444              |
| Munich                            | Germany              | 27.4                     |
| Nagoya                            | Japan                | 0                        |
| Nairobi                           | Kenya                | 0                        |
| Nanded                            | India                | 6                        |
| Nashville                         | United States        | 94                       |
| Neipu Township                    | Taiwan               | 58                       |
| New Delhi                         | India                | 17.1                     |
| New York                          | Canada               | 3                        |
| New York                          | United States        | 32.27399381              |
| Nice                              | France               | 0                        |
| Norfolk                           | United States        | 1.5                      |
| Oakland                           | United States        | 15.375                   |
| Odessa                            | Ukraine              | 0                        |
| Orlando                           | United States        | 6.666666667              |
| Osaka                             | Japan                | 14.16666667              |
| Oslo                              | Norway               | 1                        |
| Ottawa                            | Canada               | 25.5                     |
| Oviedo                            | United States        | 4.333333333              |
| Oxford                            | United Kingdom       | 183                      |
| Palo Alto                         | United States        | 43.90721649              |
| Panama City                       | United States        | 0                        |
| Paris                             | France               | 76.78846154              |
| Paris                             | United States        | 0                        |
| Parsippany-Troy Hills             | United States        | 18                       |
| Patna                             | India                | 1                        |
| Perth                             | Australia            | 242.4                    |
| Petaling Jaya                     | Malaysia             | 3                        |
| Philadelphia                      | United States        | 16.76923077              |
| Phoenix                           | United States        | 23.14285714              |
| Piscataway Township               | United States        | 1                        |
| Pittsburgh                        | United States        | 43.48148148              |
| Pleasanton                        | United States        | 0                        |
| Portland                          | United States        | 2.375                    |
| Poznan                            | Poland               | 29.66666667              |
| Pozuelo de Alarcon                | Spain                | 25.75                    |
| Prague                            | Czechia              | 12                       |
| Pune                              | India                | 40.43478261              |
| Quebec City                       | Canada               | 12.85714286              |
| Quezon City                       | Philippines          | 4.333333333              |
| Quimper                           | France               | 0                        |
| Raleigh                           | United States        | 0                        |
| Redmond                           | United States        | 306.25                   |
| Redwood City                      | United States        | 15.28571429              |
| Rexburg                           | United States        | 167                      |
| Richardson                        | United States        | 4                        |
| Rio de Janeiro                    | Brazil               | 39.2                     |
| Riyadh                            | Saudi Arabia         | 165.5                    |
| Rome                              | Italy                | 307.8                    |
| Rosario                           | Argentina            | 1                        |
| Rotterdam                         | Netherlands          | 0                        |
| Sacramento                        | United States        | 189                      |
| Saint Petersburg                  | Russia               | 101.25                   |
| Sakai                             | Japan                | 1148                     |
| Salem                             | United States        | 45.54901961              |
| Salford                           | United Kingdom       | 2                        |
| San Antonio                       | United States        | 50                       |
| San Bruno                         | United States        | 82.70212766              |
| San Diego                         | United States        | 21.7037037               |
| San Francisco                     | France               | 499                      |
| San Francisco                     | Japan                | 0                        |
| San Francisco                     | United States        | 24.48484848              |
| San Jose                          | United States        | 25.4291498               |
| San Mateo                         | United States        | 3.666666667              |
| San Salvador                      | El Salvador          | 0                        |
| Sandton                           | South Africa         | 183                      |
| Santa Clara                       | United States        | 28.82191781              |
| Santa Fe                          | Argentina            | 81                       |
| Santa Monica                      | United States        | 13                       |
| Santiago                          | Chile                | 11.85714286              |
| Sao Paulo                         | Brazil               | 29.17142857              |
| Sapporo                           | Japan                | 12                       |
| Seattle                           | United States        | 30.6                     |
| Seoul                             | South Korea          | 86.92                    |
| Sherbrooke                        | Canada               | 97                       |
| Shibuya                           | Japan                | 25                       |
| Shinjuku                          | Japan                | 45.625                   |
| Singapore                         | France               | 1                        |
| Singapore                         | Singapore            | 15.52173913              |
| Skopje                            | Macedonia (FYROM)    | 0                        |
| Somewhere in Albania              | Albania              | 16.66666667              |
| Somewhere in Algeria              | Algeria              | 5.25                     |
| Somewhere in Argentina            | Argentina            | 11.56666667              |
| Somewhere in Armenia              | Armenia              | 16                       |
| Somewhere in Australia            | Australia            | 71.40540541              |
| Somewhere in Austria              | Austria              | 93                       |
| Somewhere in Bahamas              | Bahamas              | 1.666666667              |
| Somewhere in Bahrain              | Bahrain              | 5.75                     |
| Somewhere in Bangladesh           | Bangladesh           | 99.28125                 |
| Somewhere in Barbados             | Barbados             | 12.5                     |
| Somewhere in Belarus              | Belarus              | 8.166666667              |
| Somewhere in Belgium              | Belgium              | 108.4761905              |
| Somewhere in Belize               | Belize               | 0                        |
| Somewhere in Bolivia              | Bolivia              | 6.5                      |
| Somewhere in Bosnia & Herzegovina | Bosnia & Herzegovina | 383.3333333              |
| Somewhere in Botswana             | Botswana             | 5                        |
| Somewhere in Brazil               | Brazil               | 23.6635514               |
| Somewhere in Brunei               | Brunei               | 2                        |
| Somewhere in Bulgaria             | Bulgaria             | 49                       |
| Somewhere in Cambodia             | Cambodia             | 5.833333333              |
| Somewhere in Canada               | Canada               | 47.04245283              |
| Somewhere in Chile                | Chile                | 112.75                   |
| Somewhere in China                | China                | 13.08                    |
| Somewhere in Colombia             | Colombia             | 54.40625                 |
| Somewhere in Costa Rica           | Costa Rica           | 13.16666667              |
| Somewhere in Côte d’Ivoire        | Côte d’Ivoire        | 15.5                     |
| Somewhere in Croatia              | Croatia              | 98.06666667              |
| Somewhere in Cyprus               | Cyprus               | 1.166666667              |
| Somewhere in Czechia              | Czechia              | 83                       |
| Somewhere in Denmark              | Denmark              | 27.37704918              |
| Somewhere in Dominican Republic   | Dominican Republic   | 104.75                   |
| Somewhere in Ecuador              | Ecuador              | 72.07142857              |
| Somewhere in Egypt                | Egypt                | 103.25                   |
| Somewhere in El Salvador          | El Salvador          | 9.111111111              |
| Somewhere in Estonia              | Estonia              | 16                       |
| Somewhere in Ethiopia             | Ethiopia             | 616.5                    |
| Somewhere in Finland              | Finland              | 14.64285714              |
| Somewhere in France               | France               | 30.56493506              |
| Somewhere in French Polynesia     | French Polynesia     | 0                        |
| Somewhere in Georgia              | Georgia              | 247.8                    |
| Somewhere in Germany              | Germany              | 64.79452055              |
| Somewhere in Ghana                | Ghana                | 5                        |
| Somewhere in Gibraltar            | Gibraltar            | 2                        |
| Somewhere in Greece               | Greece               | 60.90243902              |
| Somewhere in Guatemala            | Guatemala            | 31.44444444              |
| Somewhere in Haiti                | Haiti                | 1                        |
| Somewhere in Honduras             | Honduras             | 22.4                     |
| Somewhere in Hong Kong            | Hong Kong            | 22.04761905              |
| Somewhere in Hungary              | Hungary              | 10.48                    |
| Somewhere in Iceland              | Iceland              | 0                        |
| Somewhere in India                | India                | 65.37951807              |
| Somewhere in Indonesia            | Indonesia            | 61.12903226              |
| Somewhere in Iraq                 | Iraq                 | 3.25                     |
| Somewhere in Ireland              | Ireland              | 9.65625                  |
| Somewhere in Israel               | Israel               | 132.7058824              |
| Somewhere in Italy                | Italy                | 65.11206897              |
| Somewhere in Jamaica              | Jamaica              | 3                        |
| Somewhere in Japan                | Japan                | 54.44378698              |
| Somewhere in Jersey               | Jersey               | 11                       |
| Somewhere in Jordan               | Jordan               | 2.5                      |
| Somewhere in Kazakhstan           | Kazakhstan           | 31                       |
| Somewhere in Kenya                | Kenya                | 13                       |
| Somewhere in Kosovo               | Kosovo               | 1                        |
| Somewhere in Kuwait               | Kuwait               | 70.5                     |
| Somewhere in Kyrgyzstan           | Kyrgyzstan           | 16                       |
| Somewhere in Laos                 | Laos                 | 56.66666667              |
| Somewhere in Latvia               | Latvia               | 158.75                   |
| Somewhere in Lebanon              | Lebanon              | 935                      |
| Somewhere in Lithuania            | Lithuania            | 29.11764706              |
| Somewhere in Luxembourg           | Luxembourg           | 12                       |
| Somewhere in Macau                | Macau                | 384                      |
| Somewhere in Macedonia (FYROM)    | Macedonia (FYROM)    | 17.66666667              |
| Somewhere in Malaysia             | Malaysia             | 50.76470588              |
| Somewhere in Maldives             | Maldives             | 1                        |
| Somewhere in Mali                 | Mali                 | 30                       |
| Somewhere in Malta                | Malta                | 467.5                    |
| Somewhere in Martinique           | Martinique           | 92.5                     |
| Somewhere in Mauritius            | Mauritius            | 31                       |
| Somewhere in Mexico               | Mexico               | 55.66176471              |
| Somewhere in Moldova              | Moldova              | 15                       |
| Somewhere in Montenegro           | Montenegro           | 30                       |
| Somewhere in Morocco              | Morocco              | 18.2                     |
| Somewhere in Myanmar (Burma)      | Myanmar (Burma)      | 23.66666667              |
| Somewhere in Nepal                | Nepal                | 22                       |
| Somewhere in Netherlands          | Netherlands          | 45.75862069              |
| Somewhere in New Zealand          | New Zealand          | 93.95918367              |
| Somewhere in Nicaragua            | Nicaragua            | 2.5                      |
| Somewhere in Nigeria              | Nigeria              | 11.5                     |
| Somewhere in Norway               | Norway               | 74.82352941              |
| Somewhere in Oman                 | Oman                 | 42.5                     |
| Somewhere in Pakistan             | Pakistan             | 129.1666667              |
| Somewhere in Palestine            | Palestine            | 3                        |
| Somewhere in Panama               | Panama               | 12.46666667              |
| Somewhere in Papua New Guinea     | Papua New Guinea     | 57.5                     |
| Somewhere in Peru                 | Peru                 | 36.94736842              |
| Somewhere in Philippines          | Philippines          | 65.69620253              |
| Somewhere in Poland               | Poland               | 48.08                    |
| Somewhere in Portugal             | Portugal             | 84.375                   |
| Somewhere in Puerto Rico          | Puerto Rico          | 8.222222222              |
| Somewhere in Qatar                | Qatar                | 0                        |
| Somewhere in Réunion              | Réunion              | 32                       |
| Somewhere in Romania              | Romania              | 55.68571429              |
| Somewhere in Russia               | Russia               | 42.31081081              |
| Somewhere in Rwanda               | Rwanda               | 0                        |
| Somewhere in San Marino           | San Marino           | 25                       |
| Somewhere in Saudi Arabia         | Saudi Arabia         | 37.18181818              |
| Somewhere in Serbia               | Serbia               | 119.2857143              |
| Somewhere in Singapore            | Singapore            | 21.38888889              |
| Somewhere in Sint Maarten         | Sint Maarten         | 2                        |
| Somewhere in Slovakia             | Slovakia             | 61.23684211              |
| Somewhere in Slovenia             | Slovenia             | 142.8888889              |
| Somewhere in Somalia              | Somalia              | 50                       |
| Somewhere in South Africa         | South Africa         | 7.47826087               |
| Somewhere in South Korea          | South Korea          | 14.61538462              |
| Somewhere in Spain                | Spain                | 93.68                    |
| Somewhere in Sri Lanka            | Sri Lanka            | 8.583333333              |
| Somewhere in Sudan                | Sudan                | 467.5                    |
| Somewhere in Sweden               | Sweden               | 97.10909091              |
| Somewhere in Switzerland          | Switzerland          | 51.14754098              |
| Somewhere in Taiwan               | Taiwan               | 59.66071429              |
| Somewhere in Tanzania             | Tanzania             | 9                        |
| Somewhere in Thailand             | Thailand             | 7.363636364              |
| Somewhere in Trinidad & Tobago    | Trinidad & Tobago    | 47                       |
| Somewhere in Tunisia              | Tunisia              | 11.8                     |
| Somewhere in Turkey               | Turkey               | 71.19444444              |
| Somewhere in Uganda               | Uganda               | 6.5                      |
| Somewhere in Ukraine              | Ukraine              | 82.175                   |
| Somewhere in United Arab Emirates | United Arab Emirates | 32.8                     |
| Somewhere in United Kingdom       | United Kingdom       | 84.15434783              |
| Somewhere in United States        | United States        | 44.02040816              |
| Somewhere in Uruguay              | Uruguay              | 7.727272727              |
| Somewhere in Venezuela            | Venezuela            | 27.71428571              |
| Somewhere in Vietnam              | Vietnam              | 113.2                    |
| Somewhere in Zimbabwe             | Zimbabwe             | 12                       |
| South El Monte                    | United States        | 0                        |
| South San Francisco               | United States        | 6.75                     |
| St. John's                        | Canada               | 0                        |
| St. Louis                         | United States        | 3.5                      |
| Stanford                          | United States        | 3                        |
| Stockholm                         | Sweden               | 18.4                     |
| Stuttgart                         | Germany              | 214                      |
| Sunnyvale                         | United States        | 20.93169399              |
| Sydney                            | Australia            | 39.91891892              |
| Taguig                            | Philippines          | 141.8571429              |
| Tampa                             | United States        | 0                        |
| Tel Aviv-Yafo                     | Israel               | 26.70212766              |
| Tempe                             | United States        | 6                        |
| The Dalles                        | United States        | 6.5                      |
| Thessaloniki                      | Greece               | 107.6666667              |
| Timisoara                         | Romania              | 5.333333333              |
| Toronto                           | Canada               | 45.63157895              |
| Toronto                           | United States        | 0                        |
| University Park                   | United States        | 5                        |
| Unknown                           | Unknown              | 18.58333333              |
| Vancouver                         | Canada               | 17.5                     |
| Vancouver                         | United States        | 0                        |
| Vienna                            | Austria              | 3.8                      |
| Villeneuve-d'Ascq                 | France               | 1                        |
| Vilnius                           | Lithuania            | 1                        |
| Vladivostok                       | Russia               | 62                       |
| Warsaw                            | Poland               | 12.64285714              |
| Washington                        | United States        | 15.09756098              |
| Waterloo                          | Canada               | 0                        |
| Watford                           | United Kingdom       | 0                        |
| Wellesley                         | United States        | 3                        |
| Westlake Village                  | United States        | 0                        |
| Westville                         | South Africa         | 70                       |
| Wrexham                           | United Kingdom       | 13                       |
| Wroclaw                           | Poland               | 0                        |
| Yokohama                          | Japan                | 7.857142857              |
| Yokohama                          | United States        | 25                       |
| Zagreb                            | Croatia              | 1.5                      |
| Zhongli District                  | Taiwan               | 46                       |
| Zurich                            | Switzerland          | 29                       |


# Question 3: Is there any pattern in the types (product categories) of products ordered from visitors in each city and country?

**Solution Note**:  I used as a basis for the dataset that I did my final query on, a similar set of CTEs as in #2, but with a slight adjustment to add the v2productcategory into the result sets, from the allsessions_clean database.  My final query includes a filter to include only rows where the (cleaned) quantity ordered is NOT NULL or > 0, because that is what the question is asking.

### SQL Queries:
```SQL
WITH city_country_sordered_pordered AS (
	SELECT
		city,
		country,
		alls.productsku AS sessions_sku,
		s.productsku AS sales_sku,
	    alls.v2productcategory AS category,
		s.total_ordered AS sales_ordered,
		p.sku AS products_sku,
		p.orderedquantity as products_ordered
    FROM allsessions_clean alls
    LEFT JOIN salesbysku s
    ON alls.productsku = s.productsku
    LEFT JOIN products_clean p
    ON alls.productsku = p.sku
)
,
city_country_sku_clean_ordered AS (
	SELECT
		city,
		country,
		sessions_sku,
		category,
		CASE
			WHEN sales_ordered IS NOT NULL AND sales_ordered > 0 THEN sales_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NOT NULL THEN products_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NULL THEN 0::int
			ELSE 0::int
		END AS ordered
	FROM city_country_sordered_pordered
)
,
clean_city_country_sku_clean_ordered AS (
	SELECT
		CASE
			WHEN (city IS NULL OR city = '(not set)' OR city = 'not available in demo dataset') AND (country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset') THEN 'Unknown'
			WHEN city = '(not set)' OR city = 'not available in demo dataset' THEN 'Somewhere in ' || country
			ELSE city
		END AS city,
		CASE
			WHEN country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset' THEN 'Unknown'
			ELSE country
		END AS country,
		sessions_sku,
		category,
		ordered
	FROM city_country_sku_clean_ordered
)

SELECT DISTINCT
	city,
	country,
	category,
	SUM(ordered) OVER (PARTITION BY category) AS total_ordered_per_category
FROM clean_city_country_sku_clean_ordered
WHERE ordered > 0 AND ordered IS NOT NULL  -- we only want rows representing where something was ordered
ORDER BY city ASC, category ASC

```


## Answer:

Without doing further histogram, graphical, and statistical analysis on the result set, and only "eye-balling" the result set, it appears that there isn't an obvious, discernable pattern to the categories ordered by distinct city and country combinations.  I looked for (with my eyeballs only, no statisical or graphical analysis) patterns related to geography (high level) - like are visitors in hotter climates more likely to order lighter apparel and colder climates more likely to order heavier apparel?  Are people in certain locales more likely to purchase electronics or things in social media categories?  Etc.  I found no "obvious" discernable pattern, obvious to the eye, that is.  One of the reasons there might be, for example, no "apparel" pattern, is that as we saw above, this dataset has dates focused solely in the months of May, June, July.  Even in the "coldest" parts of the world, the weather is moderate and mild at that time of year, and people would be buying T-Shirts at that time just like people in the hottest parts of the world.  It would be interesting to put this result set through a graphical/histogram and statistical analysis to see if there are more subtle patterns not obvious to the human eye.

The result set generated by my query above is here:

| city                              | country              | category                                       | total_ordered_per_category |
|-----------------------------------|----------------------|------------------------------------------------|----------------------------|
| Adelaide                          | Australia            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Ahmedabad                         | India                | Home/Accessories/                              | 5272                       |
| Ahmedabad                         | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Ahmedabad                         | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Ahmedabad                         | India                | Home/Bags/                                     | 11458                      |
| Ahmedabad                         | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Ahmedabad                         | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Akron                             | United States        | Home/Apparel/Men's/                            | 2172                       |
| Amsterdam                         | Netherlands          | Home/Bags/                                     | 11458                      |
| Amsterdam                         | Netherlands          | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Amsterdam                         | Netherlands          | Home/Electronics/Flashlights/                  | 704                        |
| Amsterdam                         | Netherlands          | Home/Electronics/Power/                        | 628                        |
| Amsterdam                         | Netherlands          | Home/Shop by Brand/Google/                     | 17021                      |
| Amsterdam                         | Netherlands          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Ann Arbor                         | United States        | (not set)                                      | 21012                      |
| Ann Arbor                         | United States        | Home/Accessories/Fun/                          | 4753                       |
| Ann Arbor                         | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Ann Arbor                         | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Ann Arbor                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Ann Arbor                         | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Ann Arbor                         | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Ann Arbor                         | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Ann Arbor                         | United States        | Home/Bags/                                     | 11458                      |
| Ann Arbor                         | United States        | Home/Clearance Sale/                           | 1043                       |
| Ann Arbor                         | United States        | Home/Electronics/                              | 19272                      |
| Ann Arbor                         | United States        | Home/Electronics/Audio/                        | 12395                      |
| Ann Arbor                         | United States        | Home/Lifestyle/                                | 4885                       |
| Ann Arbor                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Ann Arbor                         | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Ann Arbor                         | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Antwerp                           | Belgium              | Home/Office/                                   | 52375                      |
| Appleton                          | United States        | Home/Electronics/                              | 19272                      |
| Ashburn                           | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Ashburn                           | United States        | Home/Lifestyle/Fun/                            | 271                        |
| Asuncion                          | Paraguay             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Athens                            | Greece               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Athens                            | Greece               | Home/Bags/                                     | 11458                      |
| Athens                            | Greece               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Atlanta                           | United States        | Home/Apparel/                                  | 7643                       |
| Atlanta                           | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Atlanta                           | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Atlanta                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Atlanta                           | United States        | Home/Apparel/Women's/                          | 2086                       |
| Atlanta                           | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Atlanta                           | United States        | Home/Bags/                                     | 11458                      |
| Atlanta                           | United States        | Home/Drinkware/                                | 48455                      |
| Atlanta                           | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Atlanta                           | United States        | Home/Office/                                   | 52375                      |
| Atlanta                           | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Atlanta                           | United States        | Home/Shop by Brand/                            | 5918                       |
| Atlanta                           | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Atlanta                           | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Austin                            | United States        | (not set)                                      | 21012                      |
| Austin                            | United States        | Apparel                                        | 480                        |
| Austin                            | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Austin                            | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Austin                            | United States        | Home/Apparel/                                  | 7643                       |
| Austin                            | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Austin                            | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Austin                            | United States        | Home/Apparel/Men's/                            | 2172                       |
| Austin                            | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Austin                            | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Austin                            | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Austin                            | United States        | Home/Apparel/Women's/                          | 2086                       |
| Austin                            | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Austin                            | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Austin                            | United States        | Home/Bags/Backpacks/                           | 788                        |
| Austin                            | United States        | Home/Drinkware/                                | 48455                      |
| Austin                            | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Austin                            | United States        | Home/Electronics/                              | 19272                      |
| Austin                            | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Austin                            | United States        | Home/Limited Supply/Bags/                      | 981                        |
| Austin                            | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Austin                            | United States        | Home/Office/                                   | 52375                      |
| Austin                            | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Austin                            | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Austin                            | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Austin                            | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Avon                              | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Avon                              | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Bandung                           | Indonesia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Bangkok                           | Thailand             | Home/Accessories/                              | 5272                       |
| Bangkok                           | Thailand             | Home/Accessories/Stickers/                     | 3390                       |
| Bangkok                           | Thailand             | Home/Apparel/                                  | 7643                       |
| Bangkok                           | Thailand             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Bangkok                           | Thailand             | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Bangkok                           | Thailand             | Home/Bags/                                     | 11458                      |
| Bangkok                           | Thailand             | Home/Drinkware/                                | 48455                      |
| Bangkok                           | Thailand             | Home/Office/                                   | 52375                      |
| Bangkok                           | Thailand             | Home/Office/Notebooks & Journals/              | 32307                      |
| Bangkok                           | Thailand             | Home/Shop by Brand/Android/                    | 8295                       |
| Bangkok                           | Thailand             | Home/Shop by Brand/Google/                     | 17021                      |
| Bangkok                           | Thailand             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Barcelona                         | Spain                | Home/Accessories/Stickers/                     | 3390                       |
| Barcelona                         | Spain                | Home/Apparel/                                  | 7643                       |
| Barcelona                         | Spain                | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Barcelona                         | Spain                | Home/Apparel/Men's/                            | 2172                       |
| Barcelona                         | Spain                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Barcelona                         | Spain                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Barcelona                         | Spain                | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Barcelona                         | Spain                | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Barcelona                         | Spain                | Home/Electronics/Electronics Accessories/      | 1121                       |
| Barcelona                         | Spain                | Home/Office/                                   | 52375                      |
| Barcelona                         | Spain                | Home/Office/Notebooks & Journals/              | 32307                      |
| Barcelona                         | Spain                | Home/Shop by Brand/Google/                     | 17021                      |
| Barcelona                         | Spain                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Bellflower                        | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Bellingham                        | United States        | Home/Accessories/                              | 5272                       |
| Bellingham                        | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Belo Horizonte                    | Brazil               | Home/Bags/Shopping and Totes/                  | 800                        |
| Bengaluru                         | India                | (not set)                                      | 21012                      |
| Bengaluru                         | India                | Home/Accessories/                              | 5272                       |
| Bengaluru                         | India                | Home/Accessories/Fun/                          | 4753                       |
| Bengaluru                         | India                | Home/Accessories/Pet/                          | 764                        |
| Bengaluru                         | India                | Home/Apparel/                                  | 7643                       |
| Bengaluru                         | India                | Home/Apparel/Kid's/                            | 1450                       |
| Bengaluru                         | India                | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Bengaluru                         | India                | Home/Apparel/Men's/                            | 2172                       |
| Bengaluru                         | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Bengaluru                         | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Bengaluru                         | India                | Home/Drinkware/                                | 48455                      |
| Bengaluru                         | India                | Home/Electronics/                              | 19272                      |
| Bengaluru                         | India                | Home/Electronics/Electronics Accessories/      | 1121                       |
| Bengaluru                         | India                | Home/Electronics/Flashlights/                  | 704                        |
| Bengaluru                         | India                | Home/Limited Supply/Bags/                      | 981                        |
| Bengaluru                         | India                | Home/Office/                                   | 52375                      |
| Bengaluru                         | India                | Home/Office/Notebooks & Journals/              | 32307                      |
| Bengaluru                         | India                | Home/Shop by Brand/                            | 5918                       |
| Bengaluru                         | India                | Home/Shop by Brand/Android/                    | 8295                       |
| Bengaluru                         | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Bengaluru                         | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Berkeley                          | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Berlin                            | Germany              | Home/Accessories/Stickers/                     | 3390                       |
| Berlin                            | Germany              | Home/Apparel/                                  | 7643                       |
| Berlin                            | Germany              | Home/Apparel/Headgear/                         | 2141                       |
| Berlin                            | Germany              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Berlin                            | Germany              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Berlin                            | Germany              | Home/Office/                                   | 52375                      |
| Berlin                            | Germany              | Home/Shop by Brand/Android/                    | 8295                       |
| Berlin                            | Germany              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Bhubaneswar                       | India                | Home/Accessories/Drinkware/                    | 7306                       |
| Bilbao                            | Spain                | (not set)                                      | 21012                      |
| Bogota                            | Colombia             | Home/Apparel/                                  | 7643                       |
| Bogota                            | Colombia             | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Bogota                            | Colombia             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Bogota                            | Colombia             | Home/Bags/Backpacks/                           | 788                        |
| Bogota                            | Colombia             | Home/Electronics/                              | 19272                      |
| Bogota                            | Colombia             | Home/Lifestyle/                                | 4885                       |
| Bogota                            | Colombia             | Home/Nest/Nest-USA/                            | 24443                      |
| Bogota                            | Colombia             | Home/Shop by Brand/Google/                     | 17021                      |
| Bogota                            | Colombia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Boston                            | United States        | Home/Accessories/Fun/                          | 4753                       |
| Boston                            | United States        | Home/Apparel/                                  | 7643                       |
| Boston                            | United States        | Home/Apparel/Men's/                            | 2172                       |
| Boston                            | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Boston                            | United States        | Home/Bags/                                     | 11458                      |
| Boston                            | United States        | Home/Bags/Backpacks/                           | 788                        |
| Boston                            | United States        | Home/Office/                                   | 52375                      |
| Boulder                           | United States        | Home/Accessories/                              | 5272                       |
| Boulder                           | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Boulder                           | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Boulder                           | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Boulder                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Boulder                           | United States        | Home/Drinkware/                                | 48455                      |
| Brisbane                          | Australia            | (not set)                                      | 21012                      |
| Brisbane                          | Australia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Brisbane                          | Australia            | Home/Apparel/Women's/                          | 2086                       |
| Brisbane                          | Australia            | Home/Electronics/                              | 19272                      |
| Brisbane                          | Australia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Brno                              | Czechia              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Brno                              | Czechia              | Home/Office/Notebooks & Journals/              | 32307                      |
| Brussels                          | Belgium              | (not set)                                      | 21012                      |
| Bucharest                         | Romania              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Bucharest                         | Romania              | Home/Bags/                                     | 11458                      |
| Bucharest                         | Romania              | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Bucharest                         | Romania              | Home/Limited Supply/Bags/                      | 981                        |
| Bucharest                         | Romania              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Budapest                          | Hungary              | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Budapest                          | Hungary              | Home/Electronics/                              | 19272                      |
| Budapest                          | Hungary              | Home/Shop by Brand/Android/                    | 8295                       |
| Budapest                          | Hungary              | Home/Shop by Brand/Google/                     | 17021                      |
| Buenos Aires                      | Argentina            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Buenos Aires                      | Argentina            | Home/Bags/Backpacks/                           | 788                        |
| Buenos Aires                      | Argentina            | Home/Electronics/Flashlights/                  | 704                        |
| Buenos Aires                      | Argentina            | Home/Lifestyle/                                | 4885                       |
| Burnaby                           | Canada               | Home/Apparel/Men's/                            | 2172                       |
| Burnaby                           | Canada               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Burnaby                           | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Calgary                           | Canada               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Calgary                           | Canada               | Home/Shop by Brand/Android/                    | 8295                       |
| Calgary                           | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Cambridge                         | United States        | (not set)                                      | 21012                      |
| Cambridge                         | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Cambridge                         | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Cambridge                         | United States        | Home/Apparel/Men's/                            | 2172                       |
| Cambridge                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Cambridge                         | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Cambridge                         | United States        | Home/Bags/                                     | 11458                      |
| Cambridge                         | United States        | Home/Drinkware/                                | 48455                      |
| Cambridge                         | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Cambridge                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Cambridge                         | United States        | Home/Office/Office Other/                      | 762                        |
| Cambridge                         | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Cape Town                         | South Africa         | Home/Apparel/Men's/                            | 2172                       |
| Chandigarh                        | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Charlotte                         | United States        | Home/Accessories/Pet/                          | 764                        |
| Charlotte                         | United States        | Home/Apparel/                                  | 7643                       |
| Charlotte                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Charlotte                         | United States        | Home/Apparel/Women's/                          | 2086                       |
| Charlotte                         | United States        | Home/Bags/                                     | 11458                      |
| Charlotte                         | United States        | Home/Bags/Backpacks/                           | 788                        |
| Charlotte                         | United States        | Home/Drinkware/                                | 48455                      |
| Charlotte                         | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Charlotte                         | United States        | Home/Lifestyle/                                | 4885                       |
| Charlotte                         | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Chennai                           | India                | Home/Accessories/                              | 5272                       |
| Chennai                           | India                | Home/Apparel/                                  | 7643                       |
| Chennai                           | India                | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Chennai                           | India                | Home/Apparel/Men's/                            | 2172                       |
| Chennai                           | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Chennai                           | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Chennai                           | India                | Home/Drinkware/                                | 48455                      |
| Chennai                           | India                | Home/Electronics/                              | 19272                      |
| Chennai                           | India                | Home/Office/                                   | 52375                      |
| Chennai                           | India                | Home/Office/Notebooks & Journals/              | 32307                      |
| Chennai                           | India                | Home/Office/Writing Instruments/               | 4872                       |
| Chennai                           | India                | Home/Shop by Brand/                            | 5918                       |
| Chennai                           | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Chennai                           | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Chicago                           | United States        | (not set)                                      | 21012                      |
| Chicago                           | United States        | Apparel                                        | 480                        |
| Chicago                           | United States        | Home/Accessories/                              | 5272                       |
| Chicago                           | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Chicago                           | United States        | Home/Accessories/Fun/                          | 4753                       |
| Chicago                           | United States        | Home/Accessories/Pet/                          | 764                        |
| Chicago                           | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Chicago                           | United States        | Home/Apparel/                                  | 7643                       |
| Chicago                           | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Chicago                           | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Chicago                           | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Chicago                           | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Chicago                           | United States        | Home/Apparel/Men's/                            | 2172                       |
| Chicago                           | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Chicago                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Chicago                           | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Chicago                           | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Chicago                           | United States        | Home/Bags/                                     | 11458                      |
| Chicago                           | United States        | Home/Bags/Backpacks/                           | 788                        |
| Chicago                           | United States        | Home/Bags/More Bags/                           | 2773                       |
| Chicago                           | United States        | Home/Brands/                                   | 20                         |
| Chicago                           | United States        | Home/Brands/YouTube/                           | 442                        |
| Chicago                           | United States        | Home/Drinkware/                                | 48455                      |
| Chicago                           | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Chicago                           | United States        | Home/Electronics/                              | 19272                      |
| Chicago                           | United States        | Home/Electronics/Audio/                        | 12395                      |
| Chicago                           | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Chicago                           | United States        | Home/Lifestyle/                                | 4885                       |
| Chicago                           | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Chicago                           | United States        | Home/Office/                                   | 52375                      |
| Chicago                           | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Chicago                           | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Chicago                           | United States        | Home/Shop by Brand/                            | 5918                       |
| Chicago                           | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Chicago                           | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Chicago                           | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Chicago                           | United States        | Lifestyle                                      | 40                         |
| Chicago                           | United States        | Nest-USA                                       | 1007                       |
| Chico                             | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Cluj-Napoca                       | Romania              | Home/Apparel/                                  | 7643                       |
| Colombo                           | Sri Lanka            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Colombo                           | Sri Lanka            | Home/Shop by Brand/Android/                    | 8295                       |
| Columbia                          | United States        | Home/Electronics/                              | 19272                      |
| Columbus                          | United States        | (not set)                                      | 21012                      |
| Columbus                          | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Copenhagen                        | Denmark              | Home/Accessories/                              | 5272                       |
| Cork                              | Ireland              | Home/Accessories/Stickers/                     | 3390                       |
| Council Bluffs                    | United States        | Home/Accessories/Fun/                          | 4753                       |
| Courbevoie                        | France               | Home/Shop by Brand/                            | 5918                       |
| Culiacan                          | Mexico               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Culiacan                          | Mexico               | Home/Drinkware/                                | 48455                      |
| Cupertino                         | United States        | (not set)                                      | 21012                      |
| Cupertino                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Cupertino                         | United States        | Home/Bags/                                     | 11458                      |
| Cupertino                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Cupertino                         | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Dallas                            | United States        | Home/Accessories/                              | 5272                       |
| Dallas                            | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Dallas                            | United States        | Home/Apparel/                                  | 7643                       |
| Dallas                            | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Dallas                            | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Dallas                            | United States        | Home/Apparel/Women's/                          | 2086                       |
| Dallas                            | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Dallas                            | United States        | Home/Bags/Backpacks/                           | 788                        |
| Dallas                            | United States        | Home/Electronics/                              | 19272                      |
| Dallas                            | United States        | Home/Electronics/Power/                        | 628                        |
| Dallas                            | United States        | Home/Lifestyle/                                | 4885                       |
| Dallas                            | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Dallas                            | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Denver                            | United States        | Home/Accessories/                              | 5272                       |
| Denver                            | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Denver                            | United States        | Home/Apparel/                                  | 7643                       |
| Denver                            | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Denver                            | United States        | Home/Bags/                                     | 11458                      |
| Detroit                           | United States        | Drinkware                                      | 23                         |
| Detroit                           | United States        | Home/Accessories/                              | 5272                       |
| Detroit                           | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Doha                              | Qatar                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Druid Hills                       | United States        | Home/Office/                                   | 52375                      |
| Dubai                             | United Arab Emirates | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Dubai                             | United Arab Emirates | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Dubai                             | United Arab Emirates | Home/Apparel/Women's/                          | 2086                       |
| Dubai                             | United Arab Emirates | Home/Electronics/Electronics Accessories/      | 1121                       |
| Dubai                             | United Arab Emirates | Home/Office/                                   | 52375                      |
| Dubai                             | United Arab Emirates | Home/Shop by Brand/YouTube/                    | 346065                     |
| Dublin                            | Ireland              | (not set)                                      | 21012                      |
| Dublin                            | Ireland              | Home/Accessories/                              | 5272                       |
| Dublin                            | Ireland              | Home/Accessories/Fun/                          | 4753                       |
| Dublin                            | Ireland              | Home/Apparel/                                  | 7643                       |
| Dublin                            | Ireland              | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Dublin                            | Ireland              | Home/Apparel/Men's/                            | 2172                       |
| Dublin                            | Ireland              | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Dublin                            | Ireland              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Dublin                            | Ireland              | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Dublin                            | Ireland              | Home/Bags/                                     | 11458                      |
| Dublin                            | Ireland              | Home/Bags/Backpacks/                           | 788                        |
| Dublin                            | Ireland              | Home/Drinkware/                                | 48455                      |
| Dublin                            | Ireland              | Home/Electronics/                              | 19272                      |
| Dublin                            | Netherlands          | Home/Electronics/                              | 19272                      |
| Dublin                            | Ireland              | Home/Electronics/Electronics Accessories/      | 1121                       |
| Dublin                            | Ireland              | Home/Lifestyle/                                | 4885                       |
| Dublin                            | Ireland              | Home/Nest/Nest-USA/                            | 24443                      |
| Dublin                            | Ireland              | Home/Office/                                   | 52375                      |
| Dublin                            | Ireland              | Home/Shop by Brand/Google/                     | 17021                      |
| Dublin                            | Ireland              | Home/Shop by Brand/YouTube/                    | 346065                     |
| East Lansing                      | United States        | Home/Bags/Backpacks/                           | 788                        |
| Eau Claire                        | United States        | Home/Accessories/Fun/                          | 4753                       |
| Eau Claire                        | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Eau Claire                        | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Edmonton                          | Canada               | (not set)                                      | 21012                      |
| Egham                             | United Kingdom       | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| El Paso                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Frankfurt                         | Germany              | Home/Shop by Brand/Google/                     | 17021                      |
| Fremont                           | United States        | (not set)                                      | 21012                      |
| Fremont                           | United States        | Home/Accessories/Fun/                          | 4753                       |
| Fremont                           | United States        | Home/Apparel/                                  | 7643                       |
| Fremont                           | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Fremont                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Fremont                           | United States        | Home/Apparel/Women's/                          | 2086                       |
| Fremont                           | United States        | Home/Bags/Backpacks/                           | 788                        |
| Fremont                           | United States        | Home/Clearance Sale/                           | 1043                       |
| Fremont                           | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Fremont                           | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Fremont                           | United States        | Home/Office/                                   | 52375                      |
| Ghent                             | Belgium              | Home/Accessories/                              | 5272                       |
| Glasgow                           | United Kingdom       | Home/Electronics/Audio/                        | 12395                      |
| Greer                             | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Gurgaon                           | India                | Home/Apparel/                                  | 7643                       |
| Gurgaon                           | India                | Home/Apparel/Men's/                            | 2172                       |
| Gurgaon                           | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Gurgaon                           | India                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Gurgaon                           | India                | Home/Drinkware/                                | 48455                      |
| Gurgaon                           | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Hamburg                           | Germany              | Home/Apparel/                                  | 7643                       |
| Hamburg                           | Germany              | Home/Bags/                                     | 11458                      |
| Hamburg                           | Germany              | Home/Drinkware/                                | 48455                      |
| Hamburg                           | Germany              | Home/Electronics/                              | 19272                      |
| Hamburg                           | Germany              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Hanoi                             | Vietnam              | Home/Apparel/                                  | 7643                       |
| Hanoi                             | Vietnam              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Hanoi                             | Vietnam              | Home/Drinkware/                                | 48455                      |
| Helsinki                          | Finland              | (not set)                                      | 21012                      |
| Ho Chi Minh City                  | Vietnam              | (not set)                                      | 21012                      |
| Ho Chi Minh City                  | Vietnam              | Home/Apparel/Men's/                            | 2172                       |
| Ho Chi Minh City                  | Vietnam              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Ho Chi Minh City                  | Vietnam              | Home/Bags/Backpacks/                           | 788                        |
| Ho Chi Minh City                  | Vietnam              | Home/Electronics/                              | 19272                      |
| Ho Chi Minh City                  | Vietnam              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Hong Kong                         | Hong Kong            | (not set)                                      | 21012                      |
| Hong Kong                         | Hong Kong            | Home/Accessories/                              | 5272                       |
| Hong Kong                         | Hong Kong            | Home/Accessories/Fun/                          | 4753                       |
| Hong Kong                         | Hong Kong            | Home/Apparel/                                  | 7643                       |
| Hong Kong                         | Hong Kong            | Home/Apparel/Men's/                            | 2172                       |
| Hong Kong                         | Hong Kong            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Hong Kong                         | Hong Kong            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Hong Kong                         | Hong Kong            | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Hong Kong                         | Hong Kong            | Home/Bags/                                     | 11458                      |
| Hong Kong                         | Hong Kong            | Home/Bags/More Bags/                           | 2773                       |
| Hong Kong                         | Hong Kong            | Home/Drinkware/                                | 48455                      |
| Hong Kong                         | Hong Kong            | Home/Electronics/                              | 19272                      |
| Hong Kong                         | Hong Kong            | Home/Electronics/Accessories/Drinkware/        | 167                        |
| Hong Kong                         | Hong Kong            | Home/Electronics/Electronics Accessories/      | 1121                       |
| Hong Kong                         | Hong Kong            | Home/Lifestyle/                                | 4885                       |
| Hong Kong                         | Hong Kong            | Home/Nest/Nest-USA/                            | 24443                      |
| Hong Kong                         | Hong Kong            | Home/Office/                                   | 52375                      |
| Hong Kong                         | United States        | Home/Office/                                   | 52375                      |
| Hong Kong                         | Hong Kong            | Home/Office/Notebooks & Journals/              | 32307                      |
| Hong Kong                         | Hong Kong            | Home/Shop by Brand/                            | 5918                       |
| Hong Kong                         | Hong Kong            | Home/Shop by Brand/Google/                     | 17021                      |
| Hong Kong                         | Hong Kong            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Houston                           | United States        | ${escCatTitle}                                 | 765                        |
| Houston                           | United States        | Home/Accessories/                              | 5272                       |
| Houston                           | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Houston                           | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Houston                           | United States        | Home/Apparel/                                  | 7643                       |
| Houston                           | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Houston                           | United States        | Home/Apparel/Men's/                            | 2172                       |
| Houston                           | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Houston                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Houston                           | United States        | Home/Bags/Backpacks/                           | 788                        |
| Houston                           | United States        | Home/Drinkware/                                | 48455                      |
| Houston                           | United States        | Home/Electronics/                              | 19272                      |
| Houston                           | United States        | Home/Lifestyle/                                | 4885                       |
| Houston                           | United States        | Home/Office/                                   | 52375                      |
| Houston                           | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Houston                           | United States        | Home/Shop by Brand/                            | 5918                       |
| Houston                           | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Houston                           | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Hyderabad                         | India                | (not set)                                      | 21012                      |
| Hyderabad                         | India                | Home/Accessories/                              | 5272                       |
| Hyderabad                         | India                | Home/Accessories/Fun/                          | 4753                       |
| Hyderabad                         | India                | Home/Accessories/Stickers/                     | 3390                       |
| Hyderabad                         | India                | Home/Apparel/                                  | 7643                       |
| Hyderabad                         | India                | Home/Apparel/Men's/                            | 2172                       |
| Hyderabad                         | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Hyderabad                         | India                | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Hyderabad                         | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Hyderabad                         | India                | Home/Apparel/Women's/                          | 2086                       |
| Hyderabad                         | India                | Home/Bags/                                     | 11458                      |
| Hyderabad                         | India                | Home/Drinkware/                                | 48455                      |
| Hyderabad                         | India                | Home/Electronics/                              | 19272                      |
| Hyderabad                         | India                | Home/Electronics/Audio/                        | 12395                      |
| Hyderabad                         | India                | Home/Nest/Nest-USA/                            | 24443                      |
| Hyderabad                         | India                | Home/Office/                                   | 52375                      |
| Hyderabad                         | India                | Home/Office/Notebooks & Journals/              | 32307                      |
| Hyderabad                         | India                | Home/Shop by Brand/Android/                    | 8295                       |
| Hyderabad                         | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Hyderabad                         | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Iasi                              | Romania              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Indore                            | India                | Home/Apparel/                                  | 7643                       |
| Indore                            | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Ipoh                              | Malaysia             | Home/Accessories/Housewares/                   | 1958                       |
| Irvine                            | United States        | (not set)                                      | 21012                      |
| Irvine                            | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Irvine                            | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Irvine                            | United States        | Home/Bags/                                     | 11458                      |
| Irvine                            | United States        | Home/Drinkware/                                | 48455                      |
| Irvine                            | United States        | Home/Electronics/                              | 19272                      |
| Irvine                            | United States        | Home/Lifestyle/                                | 4885                       |
| Irvine                            | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Irvine                            | United States        | Home/Office/Office Other/                      | 762                        |
| Irvine                            | United States        | Home/Shop by Brand/                            | 5918                       |
| Istanbul                          | Turkey               | (not set)                                      | 21012                      |
| Istanbul                          | Turkey               | Home/Accessories/Housewares/                   | 1958                       |
| Istanbul                          | Turkey               | Home/Accessories/Stickers/                     | 3390                       |
| Istanbul                          | Turkey               | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Istanbul                          | Turkey               | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Istanbul                          | Turkey               | Home/Apparel/Men's/                            | 2172                       |
| Istanbul                          | Turkey               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Istanbul                          | Turkey               | Home/Bags/Backpacks/                           | 788                        |
| Istanbul                          | Turkey               | Home/Drinkware/                                | 48455                      |
| Istanbul                          | Hungary              | Home/Shop by Brand/Android/                    | 8295                       |
| Istanbul                          | Turkey               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Izmir                             | Turkey               | Home/Apparel/                                  | 7643                       |
| Jacksonville                      | United States        | Home/Drinkware/                                | 48455                      |
| Jacksonville                      | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Jaipur                            | India                | Home/Apparel/                                  | 7643                       |
| Jaipur                            | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Jakarta                           | Indonesia            | Home/Apparel/                                  | 7643                       |
| Jakarta                           | Indonesia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Jakarta                           | Indonesia            | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Jakarta                           | Indonesia            | Home/Shop by Brand/                            | 5918                       |
| Jakarta                           | Indonesia            | Home/Shop by Brand/Android/                    | 8295                       |
| Jakarta                           | Indonesia            | Home/Shop by Brand/Google/                     | 17021                      |
| Jakarta                           | Indonesia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Jersey City                       | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Jersey City                       | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Kalamazoo                         | United States        | (not set)                                      | 21012                      |
| Kansas City                       | United States        | Home/Apparel/                                  | 7643                       |
| Kansas City                       | United States        | Home/Apparel/Women's/                          | 2086                       |
| Karachi                           | Pakistan             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Kharagpur                         | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Kharkiv                           | Ukraine              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Kiev                              | Ukraine              | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Kiev                              | Ukraine              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Kiev                              | Ukraine              | Home/Apparel/Women's/                          | 2086                       |
| Kiev                              | Ukraine              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Kiev                              | Ukraine              | Home/Bags/                                     | 11458                      |
| Kiev                              | Ukraine              | Home/Drinkware/                                | 48455                      |
| Kiev                              | Ukraine              | Home/Electronics/                              | 19272                      |
| Kiev                              | Ukraine              | Home/Electronics/Audio/                        | 12395                      |
| Kiev                              | Ukraine              | Home/Limited Supply/Bags/                      | 981                        |
| Kiev                              | Ukraine              | Home/Office/                                   | 52375                      |
| Kiev                              | Ukraine              | Home/Office/Writing Instruments/               | 4872                       |
| Kiev                              | Ukraine              | Home/Shop by Brand/Android/                    | 8295                       |
| Kiev                              | Ukraine              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Kirkland                          | United States        | (not set)                                      | 21012                      |
| Kirkland                          | United States        | Home/Accessories/Fun/                          | 4753                       |
| Kirkland                          | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Kirkland                          | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Kirkland                          | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Kirkland                          | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Kirkland                          | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Kirkland                          | United States        | Home/Apparel/Women's/                          | 2086                       |
| Kirkland                          | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Kirkland                          | United States        | Home/Bags/                                     | 11458                      |
| Kirkland                          | United States        | Home/Bags/Backpacks/                           | 788                        |
| Kirkland                          | United States        | Home/Drinkware/                                | 48455                      |
| Kirkland                          | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Kirkland                          | United States        | Home/Electronics/                              | 19272                      |
| Kirkland                          | United States        | Home/Electronics/Audio/                        | 12395                      |
| Kirkland                          | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Kirkland                          | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Kirkland                          | United States        | Home/Office/                                   | 52375                      |
| Kirkland                          | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Kirkland                          | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Kitchener                         | Canada               | Home/Accessories/Stickers/                     | 3390                       |
| Kitchener                         | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Kitchener                         | Canada               | Home/Electronics/                              | 19272                      |
| Kolkata                           | India                | (not set)                                      | 21012                      |
| Kolkata                           | India                | Home/Accessories/Stickers/                     | 3390                       |
| Kolkata                           | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Kolkata                           | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Kolkata                           | India                | Home/Bags/                                     | 11458                      |
| Kolkata                           | India                | Home/Bags/Backpacks/                           | 788                        |
| Kolkata                           | India                | Home/Electronics/Audio/                        | 12395                      |
| Kolkata                           | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Kolkata                           | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Kovrov                            | Russia               | Home/Shop by Brand/Google/                     | 17021                      |
| Kuala Lumpur                      | Malaysia             | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Kuala Lumpur                      | Malaysia             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Kuala Lumpur                      | Malaysia             | Home/Bags/                                     | 11458                      |
| Kuala Lumpur                      | Malaysia             | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Kuala Lumpur                      | Malaysia             | Home/Office/                                   | 52375                      |
| Kuala Lumpur                      | Malaysia             | Home/Shop by Brand/Android/                    | 8295                       |
| Kuala Lumpur                      | Malaysia             | Home/Shop by Brand/Google/                     | 17021                      |
| Kuala Lumpur                      | Malaysia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| La Victoria                       | Peru                 | (not set)                                      | 21012                      |
| La Victoria                       | Peru                 | Home/Apparel/                                  | 7643                       |
| La Victoria                       | Peru                 | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| La Victoria                       | Peru                 | Home/Shop by Brand/YouTube/                    | 346065                     |
| LaFayette                         | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Lahore                            | Pakistan             | Home/Electronics/                              | 19272                      |
| Lake Oswego                       | United States        | Home/Accessories/                              | 5272                       |
| Lake Oswego                       | United States        | Home/Apparel/Men's/                            | 2172                       |
| Lake Oswego                       | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Lake Oswego                       | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Las Vegas                         | United States        | Home/Apparel/                                  | 7643                       |
| Las Vegas                         | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Lisbon                            | Portugal             | Home/Apparel/Headgear/                         | 2141                       |
| London                            | United Kingdom       | (not set)                                      | 21012                      |
| London                            | United Kingdom       | Home/Accessories/                              | 5272                       |
| London                            | United Kingdom       | Home/Accessories/Drinkware/                    | 7306                       |
| London                            | United Kingdom       | Home/Accessories/Fun/                          | 4753                       |
| London                            | United Kingdom       | Home/Accessories/Stickers/                     | 3390                       |
| London                            | United Kingdom       | Home/Apparel/                                  | 7643                       |
| London                            | United Kingdom       | Home/Apparel/Headgear/                         | 2141                       |
| London                            | United Kingdom       | Home/Apparel/Kid's/                            | 1450                       |
| London                            | United Kingdom       | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| London                            | United Kingdom       | Home/Apparel/Men's/                            | 2172                       |
| London                            | United Kingdom       | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| London                            | United Kingdom       | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| London                            | United Kingdom       | Home/Apparel/Women's/                          | 2086                       |
| London                            | United Kingdom       | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| London                            | United Kingdom       | Home/Bags/                                     | 11458                      |
| London                            | United Kingdom       | Home/Bags/Backpacks/                           | 788                        |
| London                            | United Kingdom       | Home/Bags/More Bags/                           | 2773                       |
| London                            | United Kingdom       | Home/Drinkware/                                | 48455                      |
| London                            | United Kingdom       | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| London                            | United Kingdom       | Home/Electronics/                              | 19272                      |
| London                            | United Kingdom       | Home/Nest/Nest-USA/                            | 24443                      |
| London                            | United Kingdom       | Home/Office/                                   | 52375                      |
| London                            | United Kingdom       | Home/Office/Notebooks & Journals/              | 32307                      |
| London                            | United Kingdom       | Home/Office/Writing Instruments/               | 4872                       |
| London                            | United Kingdom       | Home/Shop by Brand/                            | 5918                       |
| London                            | United Kingdom       | Home/Shop by Brand/Android/                    | 8295                       |
| London                            | United Kingdom       | Home/Shop by Brand/Google/                     | 17021                      |
| London                            | United Kingdom       | Home/Shop by Brand/YouTube/                    | 346065                     |
| London                            | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Longtan District                  | Taiwan               | Home/Accessories/Drinkware/                    | 7306                       |
| Los Angeles                       | United States        | (not set)                                      | 21012                      |
| Los Angeles                       | United States        | Home/Accessories/                              | 5272                       |
| Los Angeles                       | United States        | Home/Accessories/Fun/                          | 4753                       |
| Los Angeles                       | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Los Angeles                       | United States        | Home/Apparel/                                  | 7643                       |
| Los Angeles                       | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Los Angeles                       | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Los Angeles                       | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Los Angeles                       | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Los Angeles                       | United States        | Home/Apparel/Men's/                            | 2172                       |
| Los Angeles                       | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Los Angeles                       | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Los Angeles                       | United States        | Home/Apparel/Women's/                          | 2086                       |
| Los Angeles                       | United States        | Home/Bags/                                     | 11458                      |
| Los Angeles                       | United States        | Home/Bags/Backpacks/                           | 788                        |
| Los Angeles                       | United States        | Home/Drinkware/                                | 48455                      |
| Los Angeles                       | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Los Angeles                       | United States        | Home/Electronics/                              | 19272                      |
| Los Angeles                       | United States        | Home/Electronics/Audio/                        | 12395                      |
| Los Angeles                       | United States        | Home/Electronics/Power/                        | 628                        |
| Los Angeles                       | United States        | Home/Lifestyle/                                | 4885                       |
| Los Angeles                       | United States        | Home/Limited Supply/Bags/                      | 981                        |
| Los Angeles                       | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Los Angeles                       | United States        | Home/Office/                                   | 52375                      |
| Los Angeles                       | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Los Angeles                       | United States        | Home/Shop by Brand/                            | 5918                       |
| Los Angeles                       | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Los Angeles                       | Australia            | Home/Shop by Brand/Google/                     | 17021                      |
| Los Angeles                       | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Los Angeles                       | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Los Angeles                       | United States        | Home/Spring Sale!/                             | 1079                       |
| Madison                           | United States        | Home/Bags/                                     | 11458                      |
| Madrid                            | Spain                | ${escCatTitle}                                 | 765                        |
| Madrid                            | Spain                | (not set)                                      | 21012                      |
| Madrid                            | Spain                | Home/Accessories/                              | 5272                       |
| Madrid                            | Spain                | Home/Accessories/Fun/                          | 4753                       |
| Madrid                            | Spain                | Home/Apparel/Headgear/                         | 2141                       |
| Madrid                            | Spain                | Home/Apparel/Women's/                          | 2086                       |
| Madrid                            | Spain                | Home/Shop by Brand/Android/                    | 8295                       |
| Madrid                            | Spain                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Makati                            | Philippines          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Manila                            | Philippines          | Home/Accessories/Stickers/                     | 3390                       |
| Manila                            | Philippines          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Maracaibo                         | Venezuela            | Home/Apparel/Men's/                            | 2172                       |
| Maracaibo                         | Venezuela            | Home/Electronics/                              | 19272                      |
| Marseille                         | France               | Home/Apparel/                                  | 7643                       |
| Medellin                          | Colombia             | Home/Apparel/                                  | 7643                       |
| Medellin                          | Colombia             | Home/Drinkware/                                | 48455                      |
| Melbourne                         | Australia            | (not set)                                      | 21012                      |
| Melbourne                         | Australia            | Home/Accessories/                              | 5272                       |
| Melbourne                         | Australia            | Home/Accessories/Fun/                          | 4753                       |
| Melbourne                         | Australia            | Home/Accessories/Stickers/                     | 3390                       |
| Melbourne                         | Australia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Melbourne                         | Australia            | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Melbourne                         | Australia            | Home/Bags/                                     | 11458                      |
| Melbourne                         | Australia            | Home/Drinkware/                                | 48455                      |
| Melbourne                         | Australia            | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Melbourne                         | Australia            | Home/Electronics/                              | 19272                      |
| Melbourne                         | Australia            | Home/Limited Supply/Bags/                      | 981                        |
| Melbourne                         | Australia            | Home/Nest/Nest-USA/                            | 24443                      |
| Melbourne                         | Australia            | Home/Office/                                   | 52375                      |
| Melbourne                         | Australia            | Home/Shop by Brand/Android/                    | 8295                       |
| Melbourne                         | Australia            | Home/Shop by Brand/Google/                     | 17021                      |
| Melbourne                         | Australia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Menlo Park                        | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Menlo Park                        | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Menlo Park                        | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Mexico City                       | Mexico               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Mexico City                       | Mexico               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Mexico City                       | Mexico               | Home/Bags/                                     | 11458                      |
| Mexico City                       | Mexico               | Home/Bags/More Bags/                           | 2773                       |
| Mexico City                       | Mexico               | Home/Drinkware/                                | 48455                      |
| Mexico City                       | Mexico               | Home/Electronics/                              | 19272                      |
| Mexico City                       | Mexico               | Home/Nest/Nest-USA/                            | 24443                      |
| Mexico City                       | Mexico               | Home/Shop by Brand/Google/                     | 17021                      |
| Mexico City                       | Mexico               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Milan                             | Italy                | (not set)                                      | 21012                      |
| Milan                             | Italy                | Home/Accessories/                              | 5272                       |
| Milan                             | Italy                | Home/Apparel/                                  | 7643                       |
| Milan                             | Italy                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Milan                             | Italy                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Milan                             | Italy                | Home/Lifestyle/                                | 4885                       |
| Milan                             | Italy                | Home/Office/                                   | 52375                      |
| Milan                             | Italy                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Milpitas                          | United States        | (not set)                                      | 21012                      |
| Milpitas                          | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Milpitas                          | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Milpitas                          | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Minato                            | Japan                | Home/Accessories/Fun/                          | 4753                       |
| Minato                            | Japan                | Home/Apparel/Headgear/                         | 2141                       |
| Minato                            | Japan                | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Minato                            | Japan                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Minato                            | Japan                | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Minato                            | Japan                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Minato                            | Japan                | Home/Apparel/Women's/                          | 2086                       |
| Minato                            | Japan                | Home/Bags/                                     | 11458                      |
| Minato                            | Japan                | Home/Bags/Backpacks/                           | 788                        |
| Minato                            | Japan                | Home/Electronics/Audio/                        | 12395                      |
| Minato                            | Japan                | Home/Lifestyle/                                | 4885                       |
| Minato                            | Japan                | Home/Office/                                   | 52375                      |
| Minato                            | Japan                | Home/Shop by Brand/Android/                    | 8295                       |
| Minato                            | Japan                | Home/Shop by Brand/Google/                     | 17021                      |
| Minato                            | Japan                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Mississauga                       | Canada               | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Mississauga                       | Canada               | Home/Bags/                                     | 11458                      |
| Mississauga                       | Canada               | Home/Electronics/                              | 19272                      |
| Mississauga                       | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Monterrey                         | Mexico               | Home/Apparel/                                  | 7643                       |
| Monterrey                         | Mexico               | Home/Office/                                   | 52375                      |
| Montevideo                        | Uruguay              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Montreal                          | Canada               | (not set)                                      | 21012                      |
| Montreal                          | Canada               | Home/Accessories/                              | 5272                       |
| Montreal                          | Canada               | Home/Accessories/Stickers/                     | 3390                       |
| Montreal                          | Canada               | Home/Apparel/                                  | 7643                       |
| Montreal                          | Canada               | Home/Apparel/Headgear/                         | 2141                       |
| Montreal                          | Canada               | Home/Apparel/Men's/                            | 2172                       |
| Montreal                          | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Montreal                          | Canada               | Home/Apparel/Women's/                          | 2086                       |
| Montreal                          | Canada               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Montreal                          | Canada               | Home/Bags/Backpacks/                           | 788                        |
| Montreal                          | Canada               | Home/Drinkware/                                | 48455                      |
| Montreal                          | Canada               | Home/Electronics/                              | 19272                      |
| Montreal                          | Canada               | Home/Office/                                   | 52375                      |
| Montreal                          | Canada               | Home/Office/Writing Instruments/               | 4872                       |
| Montreal                          | Canada               | Home/Shop by Brand/                            | 5918                       |
| Montreal                          | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Montreuil                         | France               | Home/Bags/                                     | 11458                      |
| Montreuil                         | France               | Home/Office/                                   | 52375                      |
| Moscow                            | Russia               | (not set)                                      | 21012                      |
| Moscow                            | Russia               | Home/Accessories/Fun/                          | 4753                       |
| Moscow                            | Russia               | Home/Accessories/Sports & Fitness/             | 318                        |
| Moscow                            | Russia               | Home/Apparel/                                  | 7643                       |
| Moscow                            | Russia               | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Moscow                            | Russia               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Moscow                            | Russia               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Mountain View                     | United States        | ${escCatTitle}                                 | 765                        |
| Mountain View                     | United States        | (not set)                                      | 21012                      |
| Mountain View                     | United States        | Apparel                                        | 480                        |
| Mountain View                     | United States        | Home/Accessories/                              | 5272                       |
| Mountain View                     | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Mountain View                     | United States        | Home/Accessories/Fun/                          | 4753                       |
| Mountain View                     | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Mountain View                     | United States        | Home/Accessories/Pet/                          | 764                        |
| Mountain View                     | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| Mountain View                     | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Mountain View                     | United States        | Home/Apparel/                                  | 7643                       |
| Mountain View                     | Japan                | Home/Apparel/Headgear/                         | 2141                       |
| Mountain View                     | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Mountain View                     | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Mountain View                     | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Mountain View                     | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Mountain View                     | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Mountain View                     | United States        | Home/Apparel/Men's/                            | 2172                       |
| Mountain View                     | Australia            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Mountain View                     | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Mountain View                     | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Mountain View                     | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Mountain View                     | United States        | Home/Apparel/Women's/                          | 2086                       |
| Mountain View                     | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Mountain View                     | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Mountain View                     | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Mountain View                     | United States        | Home/Bags/                                     | 11458                      |
| Mountain View                     | United States        | Home/Bags/Backpacks/                           | 788                        |
| Mountain View                     | United States        | Home/Bags/More Bags/                           | 2773                       |
| Mountain View                     | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| Mountain View                     | United States        | Home/Brands/YouTube/                           | 442                        |
| Mountain View                     | United States        | Home/Drinkware/                                | 48455                      |
| Mountain View                     | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Mountain View                     | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Mountain View                     | United States        | Home/Electronics/                              | 19272                      |
| Mountain View                     | United States        | Home/Electronics/Audio/                        | 12395                      |
| Mountain View                     | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Mountain View                     | United States        | Home/Electronics/Flashlights/                  | 704                        |
| Mountain View                     | United States        | Home/Electronics/Power/                        | 628                        |
| Mountain View                     | United States        | Home/Lifestyle/                                | 4885                       |
| Mountain View                     | United States        | Home/Limited Supply/Bags/                      | 981                        |
| Mountain View                     | United States        | Home/Limited Supply/Bags/Backpacks/            | 3                          |
| Mountain View                     | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Mountain View                     | United States        | Home/Office/                                   | 52375                      |
| Mountain View                     | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Mountain View                     | United States        | Home/Office/Office Other/                      | 762                        |
| Mountain View                     | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Mountain View                     | United States        | Home/Shop by Brand/                            | 5918                       |
| Mountain View                     | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Mountain View                     | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Mountain View                     | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Mountain View                     | United States        | Nest-USA                                       | 1007                       |
| Mountain View                     | United States        | Waze                                           | 9                          |
| Mumbai                            | India                | Home/Accessories/Fun/                          | 4753                       |
| Mumbai                            | India                | Home/Accessories/Stickers/                     | 3390                       |
| Mumbai                            | India                | Home/Apparel/                                  | 7643                       |
| Mumbai                            | India                | Home/Apparel/Headgear/                         | 2141                       |
| Mumbai                            | India                | Home/Apparel/Men's/                            | 2172                       |
| Mumbai                            | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Mumbai                            | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Mumbai                            | India                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Mumbai                            | India                | Home/Bags/                                     | 11458                      |
| Mumbai                            | India                | Home/Bags/Shopping and Totes/                  | 800                        |
| Mumbai                            | India                | Home/Drinkware/                                | 48455                      |
| Mumbai                            | India                | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Mumbai                            | India                | Home/Electronics/                              | 19272                      |
| Mumbai                            | India                | Home/Electronics/Electronics Accessories/      | 1121                       |
| Mumbai                            | India                | Home/Limited Supply/Bags/                      | 981                        |
| Mumbai                            | India                | Home/Nest/Nest-USA/                            | 24443                      |
| Mumbai                            | India                | Home/Office/                                   | 52375                      |
| Mumbai                            | India                | Home/Office/Notebooks & Journals/              | 32307                      |
| Mumbai                            | India                | Home/Shop by Brand/                            | 5918                       |
| Mumbai                            | India                | Home/Shop by Brand/Android/                    | 8295                       |
| Mumbai                            | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Mumbai                            | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Munich                            | Germany              | Home/Apparel/                                  | 7643                       |
| Munich                            | Germany              | Home/Apparel/Headgear/                         | 2141                       |
| Munich                            | Germany              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Munich                            | Germany              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Munich                            | Germany              | Home/Drinkware/                                | 48455                      |
| Munich                            | Germany              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Nanded                            | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Nashville                         | United States        | Nest-USA                                       | 1007                       |
| Neipu Township                    | Taiwan               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| New Delhi                         | India                | Home/Accessories/                              | 5272                       |
| New Delhi                         | India                | Home/Apparel/                                  | 7643                       |
| New Delhi                         | India                | Home/Apparel/Headgear/                         | 2141                       |
| New Delhi                         | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| New Delhi                         | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| New Delhi                         | India                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| New Delhi                         | India                | Home/Bags/                                     | 11458                      |
| New Delhi                         | India                | Home/Drinkware/                                | 48455                      |
| New Delhi                         | India                | Home/Electronics/Audio/                        | 12395                      |
| New Delhi                         | India                | Home/Lifestyle/                                | 4885                       |
| New Delhi                         | India                | Home/Shop by Brand/Google/                     | 17021                      |
| New Delhi                         | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| New York                          | United States        | ${escCatTitle}                                 | 765                        |
| New York                          | United States        | (not set)                                      | 21012                      |
| New York                          | United States        | Home/Accessories/                              | 5272                       |
| New York                          | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| New York                          | United States        | Home/Accessories/Fun/                          | 4753                       |
| New York                          | United States        | Home/Accessories/Housewares/                   | 1958                       |
| New York                          | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| New York                          | United States        | Home/Accessories/Stickers/                     | 3390                       |
| New York                          | United States        | Home/Apparel/                                  | 7643                       |
| New York                          | United States        | Home/Apparel/Headgear/                         | 2141                       |
| New York                          | United States        | Home/Apparel/Kid's/                            | 1450                       |
| New York                          | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| New York                          | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| New York                          | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| New York                          | United States        | Home/Apparel/Men's/                            | 2172                       |
| New York                          | Canada               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| New York                          | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| New York                          | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| New York                          | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| New York                          | United States        | Home/Apparel/Women's/                          | 2086                       |
| New York                          | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| New York                          | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| New York                          | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| New York                          | United States        | Home/Bags/                                     | 11458                      |
| New York                          | United States        | Home/Bags/Backpacks/                           | 788                        |
| New York                          | United States        | Home/Bags/More Bags/                           | 2773                       |
| New York                          | United States        | Home/Brands/Android/                           | 313                        |
| New York                          | United States        | Home/Brands/YouTube/                           | 442                        |
| New York                          | United States        | Home/Clearance Sale/                           | 1043                       |
| New York                          | United States        | Home/Drinkware/                                | 48455                      |
| New York                          | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| New York                          | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| New York                          | United States        | Home/Electronics/                              | 19272                      |
| New York                          | United States        | Home/Electronics/Audio/                        | 12395                      |
| New York                          | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| New York                          | United States        | Home/Electronics/Flashlights/                  | 704                        |
| New York                          | United States        | Home/Electronics/Power/                        | 628                        |
| New York                          | United States        | Home/Lifestyle/                                | 4885                       |
| New York                          | United States        | Home/Limited Supply/Bags/                      | 981                        |
| New York                          | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| New York                          | United States        | Home/Office/                                   | 52375                      |
| New York                          | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| New York                          | United States        | Home/Office/Office Other/                      | 762                        |
| New York                          | United States        | Home/Office/Writing Instruments/               | 4872                       |
| New York                          | United States        | Home/Shop by Brand/                            | 5918                       |
| New York                          | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| New York                          | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| New York                          | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| New York                          | United States        | Lifestyle/                                     | 84                         |
| Norfolk                           | United States        | Home/Lifestyle/                                | 4885                       |
| Norfolk                           | United States        | Home/Shop by Brand/                            | 5918                       |
| Oakland                           | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Oakland                           | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Oakland                           | United States        | Home/Limited Supply/Bags/                      | 981                        |
| Oakland                           | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Orlando                           | United States        | Home/Apparel/                                  | 7643                       |
| Orlando                           | United States        | Home/Apparel/Men's/                            | 2172                       |
| Orlando                           | United States        | Home/Bags/                                     | 11458                      |
| Orlando                           | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Osaka                             | Japan                | Home/Apparel/                                  | 7643                       |
| Osaka                             | Japan                | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Osaka                             | Japan                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Osaka                             | Japan                | Home/Bags/Backpacks/                           | 788                        |
| Osaka                             | Japan                | Home/Nest/Nest-USA/                            | 24443                      |
| Osaka                             | Japan                | Home/Shop by Brand/Google/                     | 17021                      |
| Oslo                              | Norway               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Ottawa                            | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Oviedo                            | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Oviedo                            | United States        | Home/Apparel/                                  | 7643                       |
| Oxford                            | United Kingdom       | Home/Electronics/Audio/                        | 12395                      |
| Palo Alto                         | United States        | (not set)                                      | 21012                      |
| Palo Alto                         | United States        | Home/Accessories/                              | 5272                       |
| Palo Alto                         | United States        | Home/Accessories/Fun/                          | 4753                       |
| Palo Alto                         | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Palo Alto                         | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Palo Alto                         | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Palo Alto                         | United States        | Home/Apparel/Men's/                            | 2172                       |
| Palo Alto                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Palo Alto                         | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Palo Alto                         | United States        | Home/Apparel/Women's/                          | 2086                       |
| Palo Alto                         | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Palo Alto                         | United States        | Home/Bags/                                     | 11458                      |
| Palo Alto                         | United States        | Home/Bags/Backpacks/                           | 788                        |
| Palo Alto                         | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| Palo Alto                         | United States        | Home/Drinkware/                                | 48455                      |
| Palo Alto                         | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Palo Alto                         | United States        | Home/Electronics/                              | 19272                      |
| Palo Alto                         | United States        | Home/Electronics/Audio/                        | 12395                      |
| Palo Alto                         | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Palo Alto                         | United States        | Home/Lifestyle/                                | 4885                       |
| Palo Alto                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Palo Alto                         | United States        | Home/Office/                                   | 52375                      |
| Palo Alto                         | United States        | Home/Shop by Brand/                            | 5918                       |
| Palo Alto                         | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Palo Alto                         | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Palo Alto                         | United States        | Nest-USA                                       | 1007                       |
| Paris                             | France               | (not set)                                      | 21012                      |
| Paris                             | France               | Home/Accessories/Stickers/                     | 3390                       |
| Paris                             | France               | Home/Apparel/                                  | 7643                       |
| Paris                             | France               | Home/Apparel/Men's/                            | 2172                       |
| Paris                             | France               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Paris                             | France               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Paris                             | France               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Paris                             | France               | Home/Bags/                                     | 11458                      |
| Paris                             | France               | Home/Drinkware/                                | 48455                      |
| Paris                             | France               | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Paris                             | France               | Home/Electronics/                              | 19272                      |
| Paris                             | France               | Home/Office/                                   | 52375                      |
| Paris                             | France               | Home/Office/Notebooks & Journals/              | 32307                      |
| Paris                             | France               | Home/Shop by Brand/                            | 5918                       |
| Paris                             | France               | Home/Shop by Brand/Android/                    | 8295                       |
| Paris                             | France               | Home/Shop by Brand/Google/                     | 17021                      |
| Paris                             | France               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Parsippany-Troy Hills             | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Patna                             | India                | (not set)                                      | 21012                      |
| Perth                             | Australia            | (not set)                                      | 21012                      |
| Perth                             | Australia            | Home/Shop by Brand/                            | 5918                       |
| Perth                             | Australia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Petaling Jaya                     | Malaysia             | Home/Apparel/                                  | 7643                       |
| Petaling Jaya                     | Malaysia             | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Philadelphia                      | United States        | Home/Apparel/Men's/                            | 2172                       |
| Philadelphia                      | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Philadelphia                      | United States        | Home/Apparel/Women's/                          | 2086                       |
| Philadelphia                      | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Philadelphia                      | United States        | Home/Bags/                                     | 11458                      |
| Philadelphia                      | United States        | Home/Electronics/                              | 19272                      |
| Philadelphia                      | United States        | Home/Electronics/Audio/                        | 12395                      |
| Philadelphia                      | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Philadelphia                      | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Philadelphia                      | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Phoenix                           | United States        | Home/Accessories/                              | 5272                       |
| Phoenix                           | United States        | Home/Apparel/Men's/                            | 2172                       |
| Phoenix                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Phoenix                           | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Phoenix                           | United States        | Home/Office/                                   | 52375                      |
| Piscataway Township               | United States        | Home/Apparel/Men's/                            | 2172                       |
| Pittsburgh                        | United States        | Home/Accessories/                              | 5272                       |
| Pittsburgh                        | United States        | Home/Accessories/Fun/                          | 4753                       |
| Pittsburgh                        | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Pittsburgh                        | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Pittsburgh                        | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Pittsburgh                        | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Pittsburgh                        | United States        | Home/Bags/                                     | 11458                      |
| Pittsburgh                        | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| Pittsburgh                        | United States        | Home/Drinkware/                                | 48455                      |
| Pittsburgh                        | United States        | Home/Electronics/                              | 19272                      |
| Pittsburgh                        | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Pittsburgh                        | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Portland                          | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Portland                          | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Poznan                            | Poland               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Poznan                            | Poland               | Home/Electronics/Audio/                        | 12395                      |
| Pozuelo de Alarcon                | Spain                | Home/Accessories/                              | 5272                       |
| Pozuelo de Alarcon                | Spain                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Prague                            | Czechia              | Home/Apparel/                                  | 7643                       |
| Prague                            | Czechia              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Pune                              | India                | Home/Apparel/Men's/                            | 2172                       |
| Pune                              | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Pune                              | India                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Pune                              | India                | Home/Drinkware/                                | 48455                      |
| Pune                              | India                | Home/Electronics/                              | 19272                      |
| Pune                              | India                | Home/Office/                                   | 52375                      |
| Pune                              | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Quebec City                       | Canada               | Home/Apparel/                                  | 7643                       |
| Quebec City                       | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Quebec City                       | Canada               | Home/Shop by Brand/Google/                     | 17021                      |
| Quebec City                       | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Quezon City                       | Philippines          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Quezon City                       | Philippines          | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Quezon City                       | Philippines          | Home/Shop by Brand/                            | 5918                       |
| Quezon City                       | Philippines          | Lifestyle/                                     | 84                         |
| Redmond                           | United States        | Home/Drinkware/                                | 48455                      |
| Redmond                           | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Redwood City                      | United States        | Home/Apparel/                                  | 7643                       |
| Redwood City                      | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Redwood City                      | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Rexburg                           | United States        | Home/Drinkware/                                | 48455                      |
| Richardson                        | United States        | Home/Electronics/Audio/                        | 12395                      |
| Rio de Janeiro                    | Brazil               | (not set)                                      | 21012                      |
| Rio de Janeiro                    | Brazil               | Home/Apparel/Headgear/                         | 2141                       |
| Rio de Janeiro                    | Brazil               | Home/Shop by Brand/Google/                     | 17021                      |
| Riyadh                            | Saudi Arabia         | Home/Apparel/Kid's/                            | 1450                       |
| Riyadh                            | Saudi Arabia         | Home/Office/                                   | 52375                      |
| Rome                              | Italy                | Home/Office/                                   | 52375                      |
| Rome                              | Italy                | Home/Office/Notebooks & Journals/              | 32307                      |
| Rome                              | Italy                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Rosario                           | Argentina            | Home/Apparel/Men's/                            | 2172                       |
| Sacramento                        | United States        | Home/Apparel/                                  | 7643                       |
| Saint Petersburg                  | Russia               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Saint Petersburg                  | Russia               | Home/Office/                                   | 52375                      |
| Saint Petersburg                  | Russia               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Sakai                             | Japan                | Home/Office/Notebooks & Journals/              | 32307                      |
| Salem                             | United States        | (not set)                                      | 21012                      |
| Salem                             | United States        | Home/Accessories/                              | 5272                       |
| Salem                             | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Salem                             | United States        | Home/Apparel/                                  | 7643                       |
| Salem                             | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Salem                             | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Salem                             | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Salem                             | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Salem                             | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Salem                             | United States        | Home/Bags/                                     | 11458                      |
| Salem                             | United States        | Home/Bags/Backpacks/                           | 788                        |
| Salem                             | United States        | Home/Drinkware/                                | 48455                      |
| Salem                             | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Salem                             | United States        | Home/Electronics/                              | 19272                      |
| Salem                             | United States        | Home/Lifestyle/                                | 4885                       |
| Salem                             | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Salem                             | United States        | Home/Office/                                   | 52375                      |
| Salem                             | United States        | Home/Office/Office Other/                      | 762                        |
| Salem                             | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Salem                             | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Salem                             | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Salford                           | United Kingdom       | Home/Electronics/Electronics Accessories/      | 1121                       |
| San Antonio                       | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| San Antonio                       | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| San Antonio                       | United States        | Home/Drinkware/                                | 48455                      |
| San Antonio                       | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| San Bruno                         | United States        | (not set)                                      | 21012                      |
| San Bruno                         | United States        | Home/Accessories/Housewares/                   | 1958                       |
| San Bruno                         | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| San Bruno                         | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| San Bruno                         | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| San Bruno                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| San Bruno                         | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| San Bruno                         | United States        | Home/Bags/                                     | 11458                      |
| San Bruno                         | United States        | Home/Bags/More Bags/                           | 2773                       |
| San Bruno                         | United States        | Home/Drinkware/                                | 48455                      |
| San Bruno                         | United States        | Home/Electronics/                              | 19272                      |
| San Bruno                         | United States        | Home/Electronics/Audio/                        | 12395                      |
| San Bruno                         | United States        | Home/Lifestyle/                                | 4885                       |
| San Bruno                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| San Bruno                         | United States        | Home/Office/                                   | 52375                      |
| San Bruno                         | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| San Bruno                         | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| San Diego                         | United States        | Home/Accessories/                              | 5272                       |
| San Diego                         | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| San Diego                         | United States        | Home/Accessories/Stickers/                     | 3390                       |
| San Diego                         | United States        | Home/Apparel/                                  | 7643                       |
| San Diego                         | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| San Diego                         | United States        | Home/Bags/                                     | 11458                      |
| San Diego                         | United States        | Home/Bags/More Bags/                           | 2773                       |
| San Diego                         | United States        | Home/Electronics/                              | 19272                      |
| San Diego                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| San Diego                         | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| San Diego                         | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| San Francisco                     | United States        | (not set)                                      | 21012                      |
| San Francisco                     | United States        | Home/Accessories/                              | 5272                       |
| San Francisco                     | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| San Francisco                     | United States        | Home/Accessories/Fun/                          | 4753                       |
| San Francisco                     | United States        | Home/Accessories/Pet/                          | 764                        |
| San Francisco                     | United States        | Home/Accessories/Stickers/                     | 3390                       |
| San Francisco                     | United States        | Home/Apparel/                                  | 7643                       |
| San Francisco                     | United States        | Home/Apparel/Headgear/                         | 2141                       |
| San Francisco                     | United States        | Home/Apparel/Kid's/                            | 1450                       |
| San Francisco                     | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| San Francisco                     | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| San Francisco                     | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| San Francisco                     | United States        | Home/Apparel/Men's/                            | 2172                       |
| San Francisco                     | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| San Francisco                     | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| San Francisco                     | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| San Francisco                     | United States        | Home/Apparel/Women's/                          | 2086                       |
| San Francisco                     | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| San Francisco                     | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| San Francisco                     | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| San Francisco                     | United States        | Home/Bags/                                     | 11458                      |
| San Francisco                     | United States        | Home/Bags/Backpacks/                           | 788                        |
| San Francisco                     | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| San Francisco                     | United States        | Home/Brands/Android/                           | 313                        |
| San Francisco                     | United States        | Home/Brands/YouTube/                           | 442                        |
| San Francisco                     | France               | Home/Drinkware/                                | 48455                      |
| San Francisco                     | United States        | Home/Drinkware/                                | 48455                      |
| San Francisco                     | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| San Francisco                     | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| San Francisco                     | United States        | Home/Electronics/                              | 19272                      |
| San Francisco                     | United States        | Home/Electronics/Audio/                        | 12395                      |
| San Francisco                     | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| San Francisco                     | United States        | Home/Electronics/Power/                        | 628                        |
| San Francisco                     | United States        | Home/Lifestyle/                                | 4885                       |
| San Francisco                     | United States        | Home/Limited Supply/Bags/                      | 981                        |
| San Francisco                     | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| San Francisco                     | United States        | Home/Office/                                   | 52375                      |
| San Francisco                     | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| San Francisco                     | United States        | Home/Office/Writing Instruments/               | 4872                       |
| San Francisco                     | United States        | Home/Shop by Brand/                            | 5918                       |
| San Francisco                     | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| San Francisco                     | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| San Francisco                     | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| San Francisco                     | United States        | Nest-USA                                       | 1007                       |
| San Jose                          | United States        | ${escCatTitle}                                 | 765                        |
| San Jose                          | United States        | (not set)                                      | 21012                      |
| San Jose                          | United States        | Apparel                                        | 480                        |
| San Jose                          | United States        | Home/Accessories/                              | 5272                       |
| San Jose                          | United States        | Home/Accessories/Fun/                          | 4753                       |
| San Jose                          | United States        | Home/Accessories/Housewares/                   | 1958                       |
| San Jose                          | United States        | Home/Accessories/Stickers/                     | 3390                       |
| San Jose                          | United States        | Home/Apparel/                                  | 7643                       |
| San Jose                          | United States        | Home/Apparel/Headgear/                         | 2141                       |
| San Jose                          | United States        | Home/Apparel/Kid's/                            | 1450                       |
| San Jose                          | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| San Jose                          | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| San Jose                          | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| San Jose                          | United States        | Home/Apparel/Men's/                            | 2172                       |
| San Jose                          | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| San Jose                          | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| San Jose                          | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| San Jose                          | United States        | Home/Apparel/Women's/                          | 2086                       |
| San Jose                          | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| San Jose                          | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| San Jose                          | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| San Jose                          | United States        | Home/Bags/                                     | 11458                      |
| San Jose                          | United States        | Home/Bags/Backpacks/                           | 788                        |
| San Jose                          | United States        | Home/Bags/More Bags/                           | 2773                       |
| San Jose                          | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| San Jose                          | United States        | Home/Clearance Sale/                           | 1043                       |
| San Jose                          | United States        | Home/Drinkware/                                | 48455                      |
| San Jose                          | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| San Jose                          | United States        | Home/Electronics/                              | 19272                      |
| San Jose                          | United States        | Home/Electronics/Audio/                        | 12395                      |
| San Jose                          | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| San Jose                          | United States        | Home/Lifestyle/                                | 4885                       |
| San Jose                          | United States        | Home/Limited Supply/Bags/                      | 981                        |
| San Jose                          | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| San Jose                          | United States        | Home/Office/                                   | 52375                      |
| San Jose                          | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| San Jose                          | United States        | Home/Shop by Brand/                            | 5918                       |
| San Jose                          | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| San Jose                          | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| San Jose                          | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| San Jose                          | United States        | Home/Spring Sale!/                             | 1079                       |
| San Mateo                         | United States        | Home/Apparel/Men's/                            | 2172                       |
| Sandton                           | South Africa         | Home/Electronics/                              | 19272                      |
| Santa Clara                       | United States        | (not set)                                      | 21012                      |
| Santa Clara                       | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Santa Clara                       | United States        | Home/Accessories/Fun/                          | 4753                       |
| Santa Clara                       | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Santa Clara                       | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| Santa Clara                       | United States        | Home/Apparel/                                  | 7643                       |
| Santa Clara                       | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Santa Clara                       | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Santa Clara                       | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Santa Clara                       | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Santa Clara                       | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Santa Clara                       | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Santa Clara                       | United States        | Home/Bags/                                     | 11458                      |
| Santa Clara                       | United States        | Home/Drinkware/                                | 48455                      |
| Santa Clara                       | United States        | Home/Electronics/                              | 19272                      |
| Santa Clara                       | United States        | Home/Electronics/Audio/                        | 12395                      |
| Santa Clara                       | United States        | Home/Electronics/Flashlights/                  | 704                        |
| Santa Clara                       | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Santa Clara                       | United States        | Home/Office/                                   | 52375                      |
| Santa Clara                       | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Santa Clara                       | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Santa Clara                       | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Santa Fe                          | Argentina            | Home/Accessories/                              | 5272                       |
| Santa Fe                          | Argentina            | Home/Electronics/Audio/                        | 12395                      |
| Santa Monica                      | United States        | Home/Bags/                                     | 11458                      |
| Santiago                          | Chile                | Home/Bags/                                     | 11458                      |
| Santiago                          | Chile                | Home/Electronics/Audio/                        | 12395                      |
| Santiago                          | Chile                | Home/Lifestyle/                                | 4885                       |
| Santiago                          | Chile                | Home/Office/                                   | 52375                      |
| Sao Paulo                         | Brazil               | Home/Accessories/Drinkware/                    | 7306                       |
| Sao Paulo                         | Brazil               | Home/Accessories/Fun/                          | 4753                       |
| Sao Paulo                         | Brazil               | Home/Accessories/Pet/                          | 764                        |
| Sao Paulo                         | Brazil               | Home/Accessories/Stickers/                     | 3390                       |
| Sao Paulo                         | Brazil               | Home/Apparel/Men's/                            | 2172                       |
| Sao Paulo                         | Brazil               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Sao Paulo                         | Brazil               | Home/Apparel/Women's/                          | 2086                       |
| Sao Paulo                         | Brazil               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Sao Paulo                         | Brazil               | Home/Bags/More Bags/                           | 2773                       |
| Sao Paulo                         | Brazil               | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Sao Paulo                         | Brazil               | Home/Electronics/                              | 19272                      |
| Sao Paulo                         | Brazil               | Home/Lifestyle/                                | 4885                       |
| Sao Paulo                         | Brazil               | Home/Limited Supply/Bags/                      | 981                        |
| Sao Paulo                         | Brazil               | Home/Office/                                   | 52375                      |
| Sao Paulo                         | Brazil               | Home/Shop by Brand/Android/                    | 8295                       |
| Sao Paulo                         | Brazil               | Home/Shop by Brand/Google/                     | 17021                      |
| Sao Paulo                         | Brazil               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Sapporo                           | Japan                | Home/Apparel/                                  | 7643                       |
| Seattle                           | United States        | (not set)                                      | 21012                      |
| Seattle                           | United States        | Home/Accessories/                              | 5272                       |
| Seattle                           | United States        | Home/Accessories/Fun/                          | 4753                       |
| Seattle                           | United States        | Home/Apparel/                                  | 7643                       |
| Seattle                           | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Seattle                           | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Seattle                           | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Seattle                           | United States        | Home/Apparel/Men's/                            | 2172                       |
| Seattle                           | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Seattle                           | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Seattle                           | United States        | Home/Apparel/Women's/                          | 2086                       |
| Seattle                           | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Seattle                           | United States        | Home/Bags/                                     | 11458                      |
| Seattle                           | United States        | Home/Bags/Backpacks/                           | 788                        |
| Seattle                           | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| Seattle                           | United States        | Home/Drinkware/                                | 48455                      |
| Seattle                           | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Seattle                           | United States        | Home/Electronics/                              | 19272                      |
| Seattle                           | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Seattle                           | United States        | Home/Electronics/Flashlights/                  | 704                        |
| Seattle                           | United States        | Home/Electronics/Power/                        | 628                        |
| Seattle                           | United States        | Home/Lifestyle/                                | 4885                       |
| Seattle                           | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Seattle                           | United States        | Home/Office/                                   | 52375                      |
| Seattle                           | United States        | Home/Office/Office Other/                      | 762                        |
| Seattle                           | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Seattle                           | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Seattle                           | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Seoul                             | South Korea          | (not set)                                      | 21012                      |
| Seoul                             | South Korea          | Home/Accessories/                              | 5272                       |
| Seoul                             | South Korea          | Home/Accessories/Stickers/                     | 3390                       |
| Seoul                             | South Korea          | Home/Apparel/                                  | 7643                       |
| Seoul                             | South Korea          | Home/Apparel/Kid's/                            | 1450                       |
| Seoul                             | South Korea          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Seoul                             | South Korea          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Seoul                             | South Korea          | Home/Bags/                                     | 11458                      |
| Seoul                             | South Korea          | Home/Drinkware/                                | 48455                      |
| Seoul                             | South Korea          | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Seoul                             | South Korea          | Home/Electronics/                              | 19272                      |
| Seoul                             | South Korea          | Home/Office/Writing Instruments/               | 4872                       |
| Seoul                             | South Korea          | Home/Shop by Brand/                            | 5918                       |
| Sherbrooke                        | Canada               | Home/Apparel/Headgear/                         | 2141                       |
| Sherbrooke                        | Canada               | Home/Nest/Nest-USA/                            | 24443                      |
| Shibuya                           | Japan                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Shinjuku                          | Japan                | Home/Accessories/Fun/                          | 4753                       |
| Shinjuku                          | Japan                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Shinjuku                          | Japan                | Home/Drinkware/                                | 48455                      |
| Shinjuku                          | Japan                | Home/Electronics/                              | 19272                      |
| Shinjuku                          | Japan                | Home/Office/                                   | 52375                      |
| Singapore                         | Singapore            | (not set)                                      | 21012                      |
| Singapore                         | Singapore            | Home/Accessories/                              | 5272                       |
| Singapore                         | Singapore            | Home/Accessories/Fun/                          | 4753                       |
| Singapore                         | Singapore            | Home/Apparel/                                  | 7643                       |
| Singapore                         | Singapore            | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Singapore                         | Singapore            | Home/Apparel/Men's/                            | 2172                       |
| Singapore                         | France               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Singapore                         | Singapore            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Singapore                         | Singapore            | Home/Apparel/Women's/                          | 2086                       |
| Singapore                         | Singapore            | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Singapore                         | Singapore            | Home/Bags/                                     | 11458                      |
| Singapore                         | Singapore            | Home/Bags/Backpacks/                           | 788                        |
| Singapore                         | Singapore            | Home/Drinkware/                                | 48455                      |
| Singapore                         | Singapore            | Home/Electronics/                              | 19272                      |
| Singapore                         | Singapore            | Home/Electronics/Electronics Accessories/      | 1121                       |
| Singapore                         | Singapore            | Home/Office/Notebooks & Journals/              | 32307                      |
| Singapore                         | Singapore            | Home/Shop by Brand/                            | 5918                       |
| Singapore                         | Singapore            | Home/Shop by Brand/Android/                    | 8295                       |
| Singapore                         | Singapore            | Home/Shop by Brand/Google/                     | 17021                      |
| Singapore                         | Singapore            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Albania              | Albania              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Algeria              | Algeria              | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Algeria              | Algeria              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Argentina            | Argentina            | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Argentina            | Argentina            | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Argentina            | Argentina            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Argentina            | Argentina            | Home/Bags/                                     | 11458                      |
| Somewhere in Argentina            | Argentina            | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Argentina            | Argentina            | Home/Office/                                   | 52375                      |
| Somewhere in Argentina            | Argentina            | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Argentina            | Argentina            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Armenia              | Armenia              | Home/Accessories/                              | 5272                       |
| Somewhere in Australia            | Australia            | (not set)                                      | 21012                      |
| Somewhere in Australia            | Australia            | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Australia            | Australia            | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Australia            | Australia            | Home/Apparel/                                  | 7643                       |
| Somewhere in Australia            | Australia            | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Australia            | Australia            | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Australia            | Australia            | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Australia            | Australia            | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Australia            | Australia            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Australia            | Australia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Australia            | Australia            | Home/Bags/                                     | 11458                      |
| Somewhere in Australia            | Australia            | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Australia            | Australia            | Home/Drinkware/                                | 48455                      |
| Somewhere in Australia            | Australia            | Home/Electronics/                              | 19272                      |
| Somewhere in Australia            | Australia            | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Australia            | Australia            | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Australia            | Australia            | Home/Lifestyle/                                | 4885                       |
| Somewhere in Australia            | Australia            | Home/Office/                                   | 52375                      |
| Somewhere in Australia            | Australia            | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Australia            | Australia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Austria              | Austria              | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Austria              | Austria              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Austria              | Austria              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Austria              | Austria              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Austria              | Austria              | Home/Bags/                                     | 11458                      |
| Somewhere in Austria              | Austria              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Austria              | Austria              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Bahamas              | Bahamas              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Bahamas              | Bahamas              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Bahrain              | Bahrain              | Home/Apparel/                                  | 7643                       |
| Somewhere in Bahrain              | Bahrain              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Bangladesh           | Bangladesh           | (not set)                                      | 21012                      |
| Somewhere in Bangladesh           | Bangladesh           | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Bangladesh           | Bangladesh           | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Bangladesh           | Bangladesh           | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Bangladesh           | Bangladesh           | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Bangladesh           | Bangladesh           | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Bangladesh           | Bangladesh           | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Bangladesh           | Bangladesh           | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Barbados             | Barbados             | Home/Apparel/                                  | 7643                       |
| Somewhere in Belarus              | Belarus              | (not set)                                      | 21012                      |
| Somewhere in Belarus              | Belarus              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Belarus              | Belarus              | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Belarus              | Belarus              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Belgium              | Belgium              | (not set)                                      | 21012                      |
| Somewhere in Belgium              | Belgium              | Home/Accessories/                              | 5272                       |
| Somewhere in Belgium              | Belgium              | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Belgium              | Belgium              | Home/Accessories/Pet/                          | 764                        |
| Somewhere in Belgium              | Belgium              | Home/Apparel/                                  | 7643                       |
| Somewhere in Belgium              | Belgium              | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Belgium              | Belgium              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Belgium              | Belgium              | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Belgium              | Belgium              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Belgium              | Belgium              | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Belgium              | Belgium              | Home/Brands/Android/                           | 313                        |
| Somewhere in Belgium              | Belgium              | Home/Drinkware/                                | 48455                      |
| Somewhere in Belgium              | Belgium              | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Belgium              | Belgium              | Home/Electronics/                              | 19272                      |
| Somewhere in Belgium              | Belgium              | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Belgium              | Belgium              | Home/Office/                                   | 52375                      |
| Somewhere in Belgium              | Belgium              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Belgium              | Belgium              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Bolivia              | Bolivia              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Bosnia & Herzegovina | Bosnia & Herzegovina | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Botswana             | Botswana             | Home/Bags/                                     | 11458                      |
| Somewhere in Brazil               | Brazil               | ${escCatTitle}                                 | 765                        |
| Somewhere in Brazil               | Brazil               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Brazil               | Brazil               | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Brazil               | Brazil               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Brazil               | Brazil               | Home/Apparel/                                  | 7643                       |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Brazil               | Brazil               | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Brazil               | Brazil               | Home/Bags/                                     | 11458                      |
| Somewhere in Brazil               | Brazil               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Brazil               | Brazil               | Home/Drinkware/                                | 48455                      |
| Somewhere in Brazil               | Brazil               | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Brazil               | Brazil               | Home/Electronics/                              | 19272                      |
| Somewhere in Brazil               | Brazil               | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Brazil               | Brazil               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Brazil               | Brazil               | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in Brazil               | Brazil               | Home/Office/                                   | 52375                      |
| Somewhere in Brazil               | Brazil               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Brazil               | Brazil               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Brazil               | Brazil               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Brunei               | Brunei               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Bulgaria             | Bulgaria             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Bulgaria             | Bulgaria             | Home/Drinkware/                                | 48455                      |
| Somewhere in Bulgaria             | Bulgaria             | Home/Electronics/                              | 19272                      |
| Somewhere in Bulgaria             | Bulgaria             | Home/Kids/                                     | 19                         |
| Somewhere in Bulgaria             | Bulgaria             | Home/Lifestyle/                                | 4885                       |
| Somewhere in Bulgaria             | Bulgaria             | Home/Office/                                   | 52375                      |
| Somewhere in Bulgaria             | Bulgaria             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Bulgaria             | Bulgaria             | Wearables/Men's T-Shirts/                      | 19                         |
| Somewhere in Cambodia             | Cambodia             | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Cambodia             | Cambodia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Canada               | Canada               | (not set)                                      | 21012                      |
| Somewhere in Canada               | Canada               | Apparel                                        | 480                        |
| Somewhere in Canada               | Canada               | Home/Accessories/                              | 5272                       |
| Somewhere in Canada               | Canada               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Canada               | Canada               | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Canada               | Canada               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Canada               | Canada               | Home/Apparel/                                  | 7643                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in Canada               | Canada               | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Somewhere in Canada               | Canada               | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Canada               | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Canada               | Canada               | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Canada               | Canada               | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Somewhere in Canada               | Canada               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Canada               | Canada               | Home/Bags/                                     | 11458                      |
| Somewhere in Canada               | Canada               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Canada               | Canada               | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Canada               | Canada               | Home/Brands/Android/                           | 313                        |
| Somewhere in Canada               | Canada               | Home/Brands/YouTube/                           | 442                        |
| Somewhere in Canada               | Canada               | Home/Drinkware/                                | 48455                      |
| Somewhere in Canada               | Canada               | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Somewhere in Canada               | Canada               | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Canada               | Canada               | Home/Electronics/                              | 19272                      |
| Somewhere in Canada               | Canada               | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Canada               | Canada               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Canada               | Canada               | Home/Electronics/Power/                        | 628                        |
| Somewhere in Canada               | Canada               | Home/Lifestyle/                                | 4885                       |
| Somewhere in Canada               | Canada               | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Canada               | Canada               | Home/Office/                                   | 52375                      |
| Somewhere in Canada               | Canada               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Canada               | Canada               | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Canada               | Canada               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Canada               | Canada               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Canada               | Canada               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Canada               | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Chile                | Chile                | Home/Accessories/                              | 5272                       |
| Somewhere in Chile                | Chile                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Chile                | Chile                | Home/Bags/                                     | 11458                      |
| Somewhere in Chile                | Chile                | Home/Electronics/                              | 19272                      |
| Somewhere in Chile                | Chile                | Home/Electronics/Power/                        | 628                        |
| Somewhere in Chile                | Chile                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in China                | China                | Home/Accessories/                              | 5272                       |
| Somewhere in China                | China                | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in China                | China                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in China                | China                | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in China                | China                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in China                | China                | Home/Bags/                                     | 11458                      |
| Somewhere in China                | China                | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in China                | China                | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in China                | China                | Home/Shop by Brand/                            | 5918                       |
| Somewhere in China                | China                | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in China                | China                | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in China                | China                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Colombia             | Colombia             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Colombia             | Colombia             | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in Colombia             | Colombia             | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Colombia             | Colombia             | Home/Office/Office Other/                      | 762                        |
| Somewhere in Colombia             | Colombia             | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Colombia             | Colombia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Costa Rica           | Costa Rica           | Home/Apparel/                                  | 7643                       |
| Somewhere in Costa Rica           | Costa Rica           | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Costa Rica           | Costa Rica           | Home/Electronics/                              | 19272                      |
| Somewhere in Costa Rica           | Costa Rica           | Home/Electronics/Power/                        | 628                        |
| Somewhere in Costa Rica           | Costa Rica           | Home/Office/                                   | 52375                      |
| Somewhere in Côte d’Ivoire        | Côte d’Ivoire        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Croatia              | Croatia              | (not set)                                      | 21012                      |
| Somewhere in Croatia              | Croatia              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Croatia              | Croatia              | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Croatia              | Croatia              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Cyprus               | Cyprus               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Cyprus               | Cyprus               | Home/Bags/                                     | 11458                      |
| Somewhere in Czechia              | Czechia              | (not set)                                      | 21012                      |
| Somewhere in Czechia              | Czechia              | Home/Accessories/                              | 5272                       |
| Somewhere in Czechia              | Czechia              | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Czechia              | Czechia              | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Czechia              | Czechia              | Home/Apparel/                                  | 7643                       |
| Somewhere in Czechia              | Czechia              | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Czechia              | Czechia              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Czechia              | Czechia              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Czechia              | Czechia              | Home/Bags/                                     | 11458                      |
| Somewhere in Czechia              | Czechia              | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Czechia              | Czechia              | Home/Drinkware/                                | 48455                      |
| Somewhere in Czechia              | Czechia              | Home/Electronics/                              | 19272                      |
| Somewhere in Czechia              | Czechia              | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Czechia              | Czechia              | Home/Electronics/Power/                        | 628                        |
| Somewhere in Czechia              | Czechia              | Home/Office/                                   | 52375                      |
| Somewhere in Czechia              | Czechia              | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Czechia              | Czechia              | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Czechia              | Czechia              | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Czechia              | Czechia              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Czechia              | Czechia              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Denmark              | Denmark              | (not set)                                      | 21012                      |
| Somewhere in Denmark              | Denmark              | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Denmark              | Denmark              | Home/Apparel/                                  | 7643                       |
| Somewhere in Denmark              | Denmark              | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Denmark              | Denmark              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Denmark              | Denmark              | Home/Bags/                                     | 11458                      |
| Somewhere in Denmark              | Denmark              | Home/Drinkware/                                | 48455                      |
| Somewhere in Denmark              | Denmark              | Home/Electronics/                              | 19272                      |
| Somewhere in Denmark              | Denmark              | Home/Lifestyle/                                | 4885                       |
| Somewhere in Denmark              | Denmark              | Home/Office/                                   | 52375                      |
| Somewhere in Denmark              | Denmark              | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Denmark              | Denmark              | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Denmark              | Denmark              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Denmark              | Denmark              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Dominican Republic   | Dominican Republic   | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Dominican Republic   | Dominican Republic   | Home/Drinkware/                                | 48455                      |
| Somewhere in Dominican Republic   | Dominican Republic   | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Ecuador              | Ecuador              | (not set)                                      | 21012                      |
| Somewhere in Ecuador              | Ecuador              | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Ecuador              | Ecuador              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Ecuador              | Ecuador              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Ecuador              | Ecuador              | Home/Electronics/                              | 19272                      |
| Somewhere in Ecuador              | Ecuador              | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Ecuador              | Ecuador              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Egypt                | Egypt                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Egypt                | Egypt                | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Egypt                | Egypt                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in El Salvador          | El Salvador          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in El Salvador          | El Salvador          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in El Salvador          | El Salvador          | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in El Salvador          | El Salvador          | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Estonia              | Estonia              | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Estonia              | Estonia              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Estonia              | Estonia              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Estonia              | Estonia              | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Estonia              | Estonia              | Home/Drinkware/                                | 48455                      |
| Somewhere in Estonia              | Estonia              | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Ethiopia             | Ethiopia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Finland              | Finland              | ${escCatTitle}                                 | 765                        |
| Somewhere in Finland              | Finland              | (not set)                                      | 21012                      |
| Somewhere in Finland              | Finland              | Home/Accessories/                              | 5272                       |
| Somewhere in Finland              | Finland              | Home/Apparel/                                  | 7643                       |
| Somewhere in Finland              | Finland              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Finland              | Finland              | Home/Electronics/                              | 19272                      |
| Somewhere in France               | France               | (not set)                                      | 21012                      |
| Somewhere in France               | France               | Home/Accessories/                              | 5272                       |
| Somewhere in France               | France               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in France               | France               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in France               | France               | Home/Apparel/                                  | 7643                       |
| Somewhere in France               | France               | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in France               | France               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in France               | France               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in France               | France               | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in France               | France               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in France               | France               | Home/Bags/                                     | 11458                      |
| Somewhere in France               | France               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in France               | France               | Home/Brands/Android/                           | 313                        |
| Somewhere in France               | France               | Home/Drinkware/                                | 48455                      |
| Somewhere in France               | France               | Home/Electronics/                              | 19272                      |
| Somewhere in France               | France               | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in France               | France               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in France               | France               | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in France               | France               | Home/Office/                                   | 52375                      |
| Somewhere in France               | France               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in France               | France               | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in France               | France               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in France               | France               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in France               | France               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in France               | France               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Georgia              | Georgia              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Georgia              | Georgia              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Germany              | Germany              | (not set)                                      | 21012                      |
| Somewhere in Germany              | Germany              | Home/Accessories/                              | 5272                       |
| Somewhere in Germany              | Germany              | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Germany              | Germany              | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Germany              | Germany              | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Germany              | Germany              | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Germany              | Germany              | Home/Apparel/                                  | 7643                       |
| Somewhere in Germany              | Germany              | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Germany              | Germany              | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Somewhere in Germany              | Germany              | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Germany              | Germany              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Germany              | Germany              | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Germany              | Germany              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Germany              | Germany              | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Germany              | Germany              | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Somewhere in Germany              | Germany              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Germany              | Germany              | Home/Bags/                                     | 11458                      |
| Somewhere in Germany              | Germany              | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Germany              | Germany              | Home/Brands/Android/                           | 313                        |
| Somewhere in Germany              | Germany              | Home/Drinkware/                                | 48455                      |
| Somewhere in Germany              | Germany              | Home/Electronics/                              | 19272                      |
| Somewhere in Germany              | Germany              | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Germany              | Germany              | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in Germany              | Germany              | Home/Lifestyle/                                | 4885                       |
| Somewhere in Germany              | Germany              | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Germany              | Germany              | Home/Office/                                   | 52375                      |
| Somewhere in Germany              | Germany              | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Germany              | Germany              | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Germany              | Germany              | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Germany              | Germany              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Germany              | Germany              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Ghana                | Ghana                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Gibraltar            | Gibraltar            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Greece               | Greece               | (not set)                                      | 21012                      |
| Somewhere in Greece               | Greece               | Home/Accessories/                              | 5272                       |
| Somewhere in Greece               | Greece               | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Greece               | Greece               | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Greece               | Greece               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Greece               | Greece               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Greece               | Greece               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Greece               | Greece               | Home/Bags/                                     | 11458                      |
| Somewhere in Greece               | Greece               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Greece               | Greece               | Home/Drinkware/                                | 48455                      |
| Somewhere in Greece               | Greece               | Home/Electronics/                              | 19272                      |
| Somewhere in Greece               | Greece               | Home/Lifestyle/                                | 4885                       |
| Somewhere in Greece               | Greece               | Home/Office/                                   | 52375                      |
| Somewhere in Greece               | Greece               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Greece               | Greece               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Greece               | Greece               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Guatemala            | Guatemala            | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Guatemala            | Guatemala            | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Guatemala            | Guatemala            | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Guatemala            | Guatemala            | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Guatemala            | Guatemala            | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Guatemala            | Guatemala            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Haiti                | Haiti                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Honduras             | Honduras             | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Hong Kong            | Hong Kong            | Home/Apparel/                                  | 7643                       |
| Somewhere in Hong Kong            | Hong Kong            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Hong Kong            | Hong Kong            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Hong Kong            | Hong Kong            | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Hong Kong            | Hong Kong            | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Hong Kong            | Hong Kong            | Home/Bags/                                     | 11458                      |
| Somewhere in Hong Kong            | Hong Kong            | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Hong Kong            | Hong Kong            | Home/Office/                                   | 52375                      |
| Somewhere in Hong Kong            | Hong Kong            | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Hong Kong            | Hong Kong            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Hungary              | Hungary              | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Hungary              | Hungary              | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Hungary              | Hungary              | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Hungary              | Hungary              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Hungary              | Hungary              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Hungary              | Hungary              | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Hungary              | Hungary              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in India                | India                | (not set)                                      | 21012                      |
| Somewhere in India                | India                | Home/Accessories/                              | 5272                       |
| Somewhere in India                | India                | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in India                | India                | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in India                | India                | Home/Apparel/                                  | 7643                       |
| Somewhere in India                | India                | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in India                | India                | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in India                | India                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in India                | India                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in India                | India                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in India                | India                | Home/Bags/                                     | 11458                      |
| Somewhere in India                | India                | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in India                | India                | Home/Drinkware/                                | 48455                      |
| Somewhere in India                | India                | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in India                | India                | Home/Electronics/                              | 19272                      |
| Somewhere in India                | India                | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in India                | India                | Home/Lifestyle/                                | 4885                       |
| Somewhere in India                | India                | Home/Limited Supply/                           | 11                         |
| Somewhere in India                | India                | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in India                | India                | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in India                | India                | Home/Office/                                   | 52375                      |
| Somewhere in India                | India                | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in India                | India                | Home/Office/Office Other/                      | 762                        |
| Somewhere in India                | India                | Home/Shop by Brand/                            | 5918                       |
| Somewhere in India                | India                | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in India                | India                | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in India                | India                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Indonesia            | Indonesia            | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Indonesia            | Indonesia            | Home/Apparel/                                  | 7643                       |
| Somewhere in Indonesia            | Indonesia            | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Indonesia            | Indonesia            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Indonesia            | Indonesia            | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Indonesia            | Indonesia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Indonesia            | Indonesia            | Home/Bags/                                     | 11458                      |
| Somewhere in Indonesia            | Indonesia            | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Indonesia            | Indonesia            | Home/Office/                                   | 52375                      |
| Somewhere in Indonesia            | Indonesia            | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Indonesia            | Indonesia            | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Indonesia            | Indonesia            | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Indonesia            | Indonesia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Iraq                 | Iraq                 | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Iraq                 | Iraq                 | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Ireland              | Ireland              | Home/Accessories/                              | 5272                       |
| Somewhere in Ireland              | Ireland              | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Ireland              | Ireland              | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Ireland              | Ireland              | Home/Apparel/                                  | 7643                       |
| Somewhere in Ireland              | Ireland              | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Ireland              | Ireland              | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in Ireland              | Ireland              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Ireland              | Ireland              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Ireland              | Ireland              | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Ireland              | Ireland              | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Ireland              | Ireland              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Ireland              | Ireland              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Israel               | Israel               | Home/Accessories/                              | 5272                       |
| Somewhere in Israel               | Israel               | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Israel               | Israel               | Home/Apparel/                                  | 7643                       |
| Somewhere in Israel               | Israel               | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Israel               | Israel               | Home/Bags/                                     | 11458                      |
| Somewhere in Israel               | Israel               | Home/Office/                                   | 52375                      |
| Somewhere in Israel               | Israel               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Israel               | Israel               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Italy                | Italy                | Home/Accessories/                              | 5272                       |
| Somewhere in Italy                | Italy                | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Italy                | Italy                | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Italy                | Italy                | Home/Apparel/                                  | 7643                       |
| Somewhere in Italy                | Italy                | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Italy                | Italy                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Italy                | Italy                | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Italy                | Italy                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Italy                | Italy                | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Italy                | Italy                | Home/Bags/                                     | 11458                      |
| Somewhere in Italy                | Italy                | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Italy                | Italy                | Home/Drinkware/                                | 48455                      |
| Somewhere in Italy                | Italy                | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Italy                | Italy                | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Italy                | Italy                | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in Italy                | Italy                | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Italy                | Italy                | Home/Office/                                   | 52375                      |
| Somewhere in Italy                | Italy                | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Italy                | Italy                | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Italy                | Italy                | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Italy                | Italy                | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Italy                | Italy                | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Italy                | Italy                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Jamaica              | Jamaica              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Japan                | Japan                | (not set)                                      | 21012                      |
| Somewhere in Japan                | Japan                | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Japan                | Japan                | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Japan                | Japan                | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Japan                | Japan                | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Japan                | Japan                | Home/Apparel/                                  | 7643                       |
| Somewhere in Japan                | Japan                | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Japan                | Japan                | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Japan                | Japan                | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Japan                | Japan                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Japan                | Japan                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Japan                | Japan                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Japan                | Japan                | Home/Bags/                                     | 11458                      |
| Somewhere in Japan                | Japan                | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Japan                | Japan                | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Japan                | Japan                | Home/Drinkware/                                | 48455                      |
| Somewhere in Japan                | Japan                | Home/Electronics/                              | 19272                      |
| Somewhere in Japan                | Japan                | Home/Lifestyle/                                | 4885                       |
| Somewhere in Japan                | Japan                | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in Japan                | Japan                | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Japan                | Japan                | Home/Office/                                   | 52375                      |
| Somewhere in Japan                | Japan                | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Japan                | Japan                | Home/Office/Office Other/                      | 762                        |
| Somewhere in Japan                | Japan                | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Japan                | Japan                | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Japan                | Japan                | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Japan                | Japan                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Jersey               | Jersey               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Jordan               | Jordan               | Home/Office/                                   | 52375                      |
| Somewhere in Kazakhstan           | Kazakhstan           | Home/Bags/Shopping and Totes/                  | 800                        |
| Somewhere in Kazakhstan           | Kazakhstan           | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Kenya                | Kenya                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Kenya                | Kenya                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Kosovo               | Kosovo               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Kuwait               | Kuwait               | Home/Apparel/                                  | 7643                       |
| Somewhere in Kuwait               | Kuwait               | Home/Bags/                                     | 11458                      |
| Somewhere in Kuwait               | Kuwait               | Home/Drinkware/                                | 48455                      |
| Somewhere in Kuwait               | Kuwait               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Kyrgyzstan           | Kyrgyzstan           | Home/Electronics/                              | 19272                      |
| Somewhere in Laos                 | Laos                 | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Latvia               | Latvia               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Latvia               | Latvia               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Latvia               | Latvia               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Latvia               | Latvia               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Lebanon              | Lebanon              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Lithuania            | Lithuania            | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Lithuania            | Lithuania            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Lithuania            | Lithuania            | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Lithuania            | Lithuania            | Home/Electronics/                              | 19272                      |
| Somewhere in Lithuania            | Lithuania            | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Lithuania            | Lithuania            | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Lithuania            | Lithuania            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Luxembourg           | Luxembourg           | (not set)                                      | 21012                      |
| Somewhere in Macau                | Macau                | (not set)                                      | 21012                      |
| Somewhere in Macau                | Macau                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Macau                | Macau                | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Macedonia (FYROM)    | Macedonia (FYROM)    | (not set)                                      | 21012                      |
| Somewhere in Macedonia (FYROM)    | Macedonia (FYROM)    | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Macedonia (FYROM)    | Macedonia (FYROM)    | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Malaysia             | Malaysia             | (not set)                                      | 21012                      |
| Somewhere in Malaysia             | Malaysia             | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Malaysia             | Malaysia             | Home/Apparel/                                  | 7643                       |
| Somewhere in Malaysia             | Malaysia             | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Malaysia             | Malaysia             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Malaysia             | Malaysia             | Home/Bags/                                     | 11458                      |
| Somewhere in Malaysia             | Malaysia             | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Somewhere in Malaysia             | Malaysia             | Home/Electronics/                              | 19272                      |
| Somewhere in Malaysia             | Malaysia             | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in Malaysia             | Malaysia             | Home/Office/                                   | 52375                      |
| Somewhere in Malaysia             | Malaysia             | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Malaysia             | Malaysia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Maldives             | Maldives             | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Mali                 | Mali                 | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Malta                | Malta                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Martinique           | Martinique           | Home/Electronics/                              | 19272                      |
| Somewhere in Martinique           | Martinique           | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Mauritius            | Mauritius            | Home/Electronics/                              | 19272                      |
| Somewhere in Mauritius            | Mauritius            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Mexico               | Mexico               | Home/Accessories/Pet/                          | 764                        |
| Somewhere in Mexico               | Mexico               | Home/Apparel/                                  | 7643                       |
| Somewhere in Mexico               | Mexico               | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Mexico               | Mexico               | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Mexico               | Mexico               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Mexico               | Mexico               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Mexico               | Mexico               | Home/Bags/                                     | 11458                      |
| Somewhere in Mexico               | Mexico               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Mexico               | Mexico               | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Mexico               | Mexico               | Home/Electronics/                              | 19272                      |
| Somewhere in Mexico               | Mexico               | Home/Lifestyle/                                | 4885                       |
| Somewhere in Mexico               | Mexico               | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Mexico               | Mexico               | Home/Office/                                   | 52375                      |
| Somewhere in Mexico               | Mexico               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Mexico               | Mexico               | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Mexico               | Mexico               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Mexico               | Mexico               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Mexico               | Mexico               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Moldova              | Moldova              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Montenegro           | Montenegro           | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Morocco              | Morocco              | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Morocco              | Morocco              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Morocco              | Morocco              | Home/Bags/Shopping and Totes/                  | 800                        |
| Somewhere in Morocco              | Morocco              | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Morocco              | Morocco              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Morocco              | Morocco              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Myanmar (Burma)      | Myanmar (Burma)      | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Myanmar (Burma)      | Myanmar (Burma)      | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Nepal                | Nepal                | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Nepal                | Nepal                | Home/Bags/                                     | 11458                      |
| Somewhere in Nepal                | Nepal                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Netherlands          | Netherlands          | (not set)                                      | 21012                      |
| Somewhere in Netherlands          | Netherlands          | Home/Accessories/                              | 5272                       |
| Somewhere in Netherlands          | Netherlands          | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Netherlands          | Netherlands          | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Netherlands          | Netherlands          | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Netherlands          | Netherlands          | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Netherlands          | Netherlands          | Home/Apparel/                                  | 7643                       |
| Somewhere in Netherlands          | Netherlands          | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Netherlands          | Netherlands          | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Netherlands          | Netherlands          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Netherlands          | Netherlands          | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Netherlands          | Netherlands          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Netherlands          | Netherlands          | Home/Bags/                                     | 11458                      |
| Somewhere in Netherlands          | Netherlands          | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Netherlands          | Netherlands          | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Netherlands          | Netherlands          | Home/Drinkware/                                | 48455                      |
| Somewhere in Netherlands          | Netherlands          | Home/Electronics/                              | 19272                      |
| Somewhere in Netherlands          | Netherlands          | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Netherlands          | Netherlands          | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in Netherlands          | Netherlands          | Home/Electronics/Power/                        | 628                        |
| Somewhere in Netherlands          | Netherlands          | Home/Lifestyle/                                | 4885                       |
| Somewhere in Netherlands          | Netherlands          | Home/Office/                                   | 52375                      |
| Somewhere in Netherlands          | Netherlands          | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Netherlands          | Netherlands          | Home/Office/Office Other/                      | 762                        |
| Somewhere in Netherlands          | Netherlands          | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Netherlands          | Netherlands          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in New Zealand          | New Zealand          | (not set)                                      | 21012                      |
| Somewhere in New Zealand          | New Zealand          | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in New Zealand          | New Zealand          | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in New Zealand          | New Zealand          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in New Zealand          | New Zealand          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in New Zealand          | New Zealand          | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in New Zealand          | New Zealand          | Home/Bags/                                     | 11458                      |
| Somewhere in New Zealand          | New Zealand          | Home/Electronics/                              | 19272                      |
| Somewhere in New Zealand          | New Zealand          | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in New Zealand          | New Zealand          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Nicaragua            | Nicaragua            | Home/Electronics/                              | 19272                      |
| Somewhere in Nigeria              | Nigeria              | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Nigeria              | Nigeria              | Home/Apparel/                                  | 7643                       |
| Somewhere in Nigeria              | Nigeria              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Nigeria              | Nigeria              | Home/Bags/                                     | 11458                      |
| Somewhere in Nigeria              | Nigeria              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Norway               | Norway               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Norway               | Norway               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Norway               | Norway               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Norway               | Norway               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Norway               | Norway               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Norway               | Norway               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Norway               | Norway               | Home/Brands/YouTube/                           | 442                        |
| Somewhere in Norway               | Norway               | Home/Drinkware/                                | 48455                      |
| Somewhere in Norway               | Norway               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Norway               | Norway               | Home/Office/                                   | 52375                      |
| Somewhere in Norway               | Norway               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Norway               | Norway               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Norway               | Norway               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Oman                 | Oman                 | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Pakistan             | Pakistan             | (not set)                                      | 21012                      |
| Somewhere in Pakistan             | Pakistan             | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Pakistan             | Pakistan             | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in Pakistan             | Pakistan             | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Pakistan             | Pakistan             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Pakistan             | Pakistan             | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Pakistan             | Pakistan             | Home/Electronics/                              | 19272                      |
| Somewhere in Pakistan             | Pakistan             | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in Pakistan             | Pakistan             | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Pakistan             | Pakistan             | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Pakistan             | Pakistan             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Palestine            | Palestine            | Home/Bags/                                     | 11458                      |
| Somewhere in Panama               | Panama               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Panama               | Panama               | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Panama               | Panama               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Panama               | Panama               | Home/Drinkware/                                | 48455                      |
| Somewhere in Panama               | Panama               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Panama               | Panama               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Panama               | Panama               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Panama               | Panama               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Papua New Guinea     | Papua New Guinea     | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Peru                 | Peru                 | Home/Accessories/                              | 5272                       |
| Somewhere in Peru                 | Peru                 | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Peru                 | Peru                 | Home/Drinkware/                                | 48455                      |
| Somewhere in Peru                 | Peru                 | Home/Electronics/                              | 19272                      |
| Somewhere in Peru                 | Peru                 | Home/Lifestyle/                                | 4885                       |
| Somewhere in Peru                 | Peru                 | Home/Office/                                   | 52375                      |
| Somewhere in Peru                 | Peru                 | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Peru                 | Peru                 | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Philippines          | Philippines          | (not set)                                      | 21012                      |
| Somewhere in Philippines          | Philippines          | Home/Accessories/                              | 5272                       |
| Somewhere in Philippines          | Philippines          | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Philippines          | Philippines          | Home/Apparel/                                  | 7643                       |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Philippines          | Philippines          | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Philippines          | Philippines          | Home/Bags/                                     | 11458                      |
| Somewhere in Philippines          | Philippines          | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Philippines          | Philippines          | Home/Electronics/                              | 19272                      |
| Somewhere in Philippines          | Philippines          | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Philippines          | Philippines          | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in Philippines          | Philippines          | Home/Lifestyle/                                | 4885                       |
| Somewhere in Philippines          | Philippines          | Home/Office/                                   | 52375                      |
| Somewhere in Philippines          | Philippines          | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Philippines          | Philippines          | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Philippines          | Philippines          | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Philippines          | Philippines          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Philippines          | Philippines          | Lifestyle/                                     | 84                         |
| Somewhere in Poland               | Poland               | Home/Accessories/                              | 5272                       |
| Somewhere in Poland               | Poland               | Home/Apparel/                                  | 7643                       |
| Somewhere in Poland               | Poland               | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Poland               | Poland               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Poland               | Poland               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Poland               | Poland               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Poland               | Poland               | Home/Drinkware/                                | 48455                      |
| Somewhere in Poland               | Poland               | Home/Electronics/                              | 19272                      |
| Somewhere in Poland               | Poland               | Home/Lifestyle/                                | 4885                       |
| Somewhere in Poland               | Poland               | Home/Office/                                   | 52375                      |
| Somewhere in Poland               | Poland               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Poland               | Poland               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Poland               | Poland               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Poland               | Poland               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Portugal             | Portugal             | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Portugal             | Portugal             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Portugal             | Portugal             | Home/Bags/                                     | 11458                      |
| Somewhere in Portugal             | Portugal             | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Portugal             | Portugal             | Home/Office/                                   | 52375                      |
| Somewhere in Portugal             | Portugal             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Puerto Rico          | Puerto Rico          | (not set)                                      | 21012                      |
| Somewhere in Puerto Rico          | Puerto Rico          | Home/Office/                                   | 52375                      |
| Somewhere in Puerto Rico          | Puerto Rico          | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Réunion              | Réunion              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Réunion              | Réunion              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Romania              | Romania              | Home/Accessories/                              | 5272                       |
| Somewhere in Romania              | Romania              | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Romania              | Romania              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Romania              | Romania              | Home/Bags/                                     | 11458                      |
| Somewhere in Romania              | Romania              | Home/Drinkware/                                | 48455                      |
| Somewhere in Romania              | Romania              | Home/Lifestyle/                                | 4885                       |
| Somewhere in Romania              | Romania              | Home/Office/                                   | 52375                      |
| Somewhere in Romania              | Romania              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Russia               | Russia               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Russia               | Russia               | Home/Apparel/                                  | 7643                       |
| Somewhere in Russia               | Russia               | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Russia               | Russia               | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Russia               | Russia               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Russia               | Russia               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Russia               | Russia               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Russia               | Russia               | Home/Bags/                                     | 11458                      |
| Somewhere in Russia               | Russia               | Home/Brands/Android/                           | 313                        |
| Somewhere in Russia               | Russia               | Home/Drinkware/                                | 48455                      |
| Somewhere in Russia               | Russia               | Home/Electronics/                              | 19272                      |
| Somewhere in Russia               | Russia               | Home/Lifestyle/                                | 4885                       |
| Somewhere in Russia               | Russia               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Russia               | Russia               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Russia               | Russia               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Russia               | Russia               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in San Marino           | San Marino           | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Saudi Arabia         | Saudi Arabia         | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Saudi Arabia         | Saudi Arabia         | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Saudi Arabia         | Saudi Arabia         | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Saudi Arabia         | Saudi Arabia         | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Saudi Arabia         | Saudi Arabia         | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Serbia               | Serbia               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Serbia               | Serbia               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Serbia               | Serbia               | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Somewhere in Serbia               | Serbia               | Home/Drinkware/                                | 48455                      |
| Somewhere in Serbia               | Serbia               | Home/Office/                                   | 52375                      |
| Somewhere in Serbia               | Serbia               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Singapore            | Singapore            | (not set)                                      | 21012                      |
| Somewhere in Singapore            | Singapore            | Home/Accessories/                              | 5272                       |
| Somewhere in Singapore            | Singapore            | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Singapore            | Singapore            | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Singapore            | Singapore            | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Singapore            | Singapore            | Home/Apparel/                                  | 7643                       |
| Somewhere in Singapore            | Singapore            | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in Singapore            | Singapore            | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in Singapore            | Singapore            | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Singapore            | Singapore            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Singapore            | Singapore            | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Singapore            | Singapore            | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Somewhere in Singapore            | Singapore            | Home/Bags/                                     | 11458                      |
| Somewhere in Singapore            | Singapore            | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Singapore            | Singapore            | Home/Brands/YouTube/                           | 442                        |
| Somewhere in Singapore            | Singapore            | Home/Drinkware/                                | 48455                      |
| Somewhere in Singapore            | Singapore            | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Somewhere in Singapore            | Singapore            | Home/Electronics/                              | 19272                      |
| Somewhere in Singapore            | Singapore            | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Singapore            | Singapore            | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Singapore            | Singapore            | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Singapore            | Singapore            | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Singapore            | Singapore            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Sint Maarten         | Sint Maarten         | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Slovakia             | Slovakia             | Home/Accessories/                              | 5272                       |
| Somewhere in Slovakia             | Slovakia             | Home/Apparel/                                  | 7643                       |
| Somewhere in Slovakia             | Slovakia             | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Slovakia             | Slovakia             | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Slovakia             | Slovakia             | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Slovakia             | Slovakia             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Slovakia             | Slovakia             | Home/Bags/                                     | 11458                      |
| Somewhere in Slovakia             | Slovakia             | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Slovakia             | Slovakia             | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Slovakia             | Slovakia             | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Slovakia             | Slovakia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Slovenia             | Slovenia             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Somalia              | Somalia              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in South Africa         | South Africa         | Home/Apparel/                                  | 7643                       |
| Somewhere in South Africa         | South Africa         | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in South Africa         | South Africa         | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in South Africa         | South Africa         | Home/Bags/                                     | 11458                      |
| Somewhere in South Africa         | South Africa         | Home/Electronics/                              | 19272                      |
| Somewhere in South Africa         | South Africa         | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in South Africa         | South Africa         | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in South Korea          | South Korea          | (not set)                                      | 21012                      |
| Somewhere in South Korea          | South Korea          | Home/Accessories/                              | 5272                       |
| Somewhere in South Korea          | South Korea          | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in South Korea          | South Korea          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in South Korea          | South Korea          | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in South Korea          | South Korea          | Home/Office/                                   | 52375                      |
| Somewhere in South Korea          | South Korea          | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in South Korea          | South Korea          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Spain                | Spain                | (not set)                                      | 21012                      |
| Somewhere in Spain                | Spain                | Home/Accessories/                              | 5272                       |
| Somewhere in Spain                | Spain                | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Spain                | Spain                | Home/Apparel/                                  | 7643                       |
| Somewhere in Spain                | Spain                | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Spain                | Spain                | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Spain                | Spain                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Spain                | Spain                | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Spain                | Spain                | Home/Bags/                                     | 11458                      |
| Somewhere in Spain                | Spain                | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Spain                | Spain                | Home/Drinkware/                                | 48455                      |
| Somewhere in Spain                | Spain                | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Somewhere in Spain                | Spain                | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Spain                | Spain                | Home/Electronics/                              | 19272                      |
| Somewhere in Spain                | Spain                | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Spain                | Spain                | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Spain                | Spain                | Home/Lifestyle/                                | 4885                       |
| Somewhere in Spain                | Spain                | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Spain                | Spain                | Home/Office/                                   | 52375                      |
| Somewhere in Spain                | Spain                | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Spain                | Spain                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Sri Lanka            | Sri Lanka            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Sri Lanka            | Sri Lanka            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Sri Lanka            | Sri Lanka            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Sudan                | Sudan                | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Sweden               | Sweden               | (not set)                                      | 21012                      |
| Somewhere in Sweden               | Sweden               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Sweden               | Sweden               | Home/Apparel/                                  | 7643                       |
| Somewhere in Sweden               | Sweden               | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Somewhere in Sweden               | Sweden               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Sweden               | Sweden               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Sweden               | Sweden               | Home/Drinkware/                                | 48455                      |
| Somewhere in Sweden               | Sweden               | Home/Electronics/                              | 19272                      |
| Somewhere in Sweden               | Sweden               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Sweden               | Sweden               | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Sweden               | Sweden               | Home/Office/                                   | 52375                      |
| Somewhere in Sweden               | Sweden               | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in Sweden               | Sweden               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Sweden               | Sweden               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Switzerland          | Switzerland          | (not set)                                      | 21012                      |
| Somewhere in Switzerland          | Switzerland          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Switzerland          | Switzerland          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Switzerland          | Switzerland          | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Switzerland          | Switzerland          | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Switzerland          | Switzerland          | Home/Bags/                                     | 11458                      |
| Somewhere in Switzerland          | Switzerland          | Home/Electronics/                              | 19272                      |
| Somewhere in Switzerland          | Switzerland          | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Switzerland          | Switzerland          | Home/Lifestyle/                                | 4885                       |
| Somewhere in Switzerland          | Switzerland          | Home/Office/                                   | 52375                      |
| Somewhere in Switzerland          | Switzerland          | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Switzerland          | Switzerland          | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Switzerland          | Switzerland          | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Switzerland          | Switzerland          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Taiwan               | Taiwan               | (not set)                                      | 21012                      |
| Somewhere in Taiwan               | Taiwan               | Home/Accessories/                              | 5272                       |
| Somewhere in Taiwan               | Taiwan               | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in Taiwan               | Taiwan               | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Taiwan               | Taiwan               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/                                  | 7643                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Somewhere in Taiwan               | Taiwan               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Taiwan               | Taiwan               | Home/Bags/                                     | 11458                      |
| Somewhere in Taiwan               | Taiwan               | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in Taiwan               | Taiwan               | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in Taiwan               | Taiwan               | Home/Drinkware/                                | 48455                      |
| Somewhere in Taiwan               | Taiwan               | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in Taiwan               | Taiwan               | Home/Electronics/                              | 19272                      |
| Somewhere in Taiwan               | Taiwan               | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Taiwan               | Taiwan               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in Taiwan               | Taiwan               | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in Taiwan               | Taiwan               | Home/Lifestyle/                                | 4885                       |
| Somewhere in Taiwan               | Taiwan               | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in Taiwan               | Taiwan               | Home/Office/                                   | 52375                      |
| Somewhere in Taiwan               | Taiwan               | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in Taiwan               | Taiwan               | Home/Office/Office Other/                      | 762                        |
| Somewhere in Taiwan               | Taiwan               | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Taiwan               | Taiwan               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Taiwan               | Taiwan               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Taiwan               | Taiwan               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Tanzania             | Tanzania             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Thailand             | Thailand             | (not set)                                      | 21012                      |
| Somewhere in Thailand             | Thailand             | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in Thailand             | Thailand             | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Thailand             | Thailand             | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Thailand             | Thailand             | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Trinidad & Tobago    | Trinidad & Tobago    | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Tunisia              | Tunisia              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Turkey               | Turkey               | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in Turkey               | Turkey               | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Turkey               | Turkey               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Turkey               | Turkey               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Turkey               | Turkey               | Home/Bags/                                     | 11458                      |
| Somewhere in Turkey               | Turkey               | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in Turkey               | Turkey               | Home/Office/                                   | 52375                      |
| Somewhere in Turkey               | Turkey               | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Turkey               | Turkey               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Uganda               | Uganda               | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Ukraine              | Ukraine              | (not set)                                      | 21012                      |
| Somewhere in Ukraine              | Ukraine              | Home/Accessories/                              | 5272                       |
| Somewhere in Ukraine              | Ukraine              | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in Ukraine              | Ukraine              | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in Ukraine              | Ukraine              | Home/Apparel/                                  | 7643                       |
| Somewhere in Ukraine              | Ukraine              | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Ukraine              | Ukraine              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Ukraine              | Ukraine              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Ukraine              | Ukraine              | Home/Electronics/                              | 19272                      |
| Somewhere in Ukraine              | Ukraine              | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in Ukraine              | Ukraine              | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in Ukraine              | Ukraine              | Home/Office/                                   | 52375                      |
| Somewhere in Ukraine              | Ukraine              | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in Ukraine              | Ukraine              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Ukraine              | Ukraine              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in United Arab Emirates | United Arab Emirates | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in United Arab Emirates | United Arab Emirates | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in United Arab Emirates | United Arab Emirates | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in United Arab Emirates | United Arab Emirates | Home/Office/                                   | 52375                      |
| Somewhere in United Arab Emirates | United Arab Emirates | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in United Arab Emirates | United Arab Emirates | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in United Kingdom       | United Kingdom       | (not set)                                      | 21012                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Accessories/                              | 5272                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/                                  | 7643                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Bags/                                     | 11458                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Brands/YouTube/                           | 442                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Drinkware/                                | 48455                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Electronics/                              | 19272                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Gift Cards/                               | 7                          |
| Somewhere in United Kingdom       | United Kingdom       | Home/Lifestyle/Fun/                            | 271                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Office/                                   | 52375                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Office/Office Other/                      | 762                        |
| Somewhere in United Kingdom       | United Kingdom       | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Shop by Brand/                            | 5918                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in United Kingdom       | United Kingdom       | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in United Kingdom       | United Kingdom       | Home/Shop by Brand/Waze/                       | 5                          |
| Somewhere in United Kingdom       | United Kingdom       | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in United Kingdom       | United Kingdom       | Home/Spring Sale!/                             | 1079                       |
| Somewhere in United States        | United States        | ${escCatTitle}                                 | 765                        |
| Somewhere in United States        | United States        | (not set)                                      | 21012                      |
| Somewhere in United States        | United States        | Apparel                                        | 480                        |
| Somewhere in United States        | United States        | Bottles/                                       | 2                          |
| Somewhere in United States        | United States        | Electronics                                    | 85                         |
| Somewhere in United States        | United States        | Home/Accessories/                              | 5272                       |
| Somewhere in United States        | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Somewhere in United States        | United States        | Home/Accessories/Fun/                          | 4753                       |
| Somewhere in United States        | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Somewhere in United States        | United States        | Home/Accessories/Pet/                          | 764                        |
| Somewhere in United States        | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| Somewhere in United States        | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Somewhere in United States        | United States        | Home/Apparel/                                  | 7643                       |
| Somewhere in United States        | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Somewhere in United States        | United States        | Home/Apparel/Kid's/                            | 1450                       |
| Somewhere in United States        | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Somewhere in United States        | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Somewhere in United States        | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Somewhere in United States        | United States        | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in United States        | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in United States        | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Somewhere in United States        | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in United States        | United States        | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in United States        | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Somewhere in United States        | United States        | Home/Apparel/Women's/Women's-Performance Wear/ | 93                         |
| Somewhere in United States        | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in United States        | United States        | Home/Bags/                                     | 11458                      |
| Somewhere in United States        | United States        | Home/Bags/Backpacks/                           | 788                        |
| Somewhere in United States        | United States        | Home/Bags/More Bags/                           | 2773                       |
| Somewhere in United States        | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| Somewhere in United States        | United States        | Home/Brands/                                   | 20                         |
| Somewhere in United States        | United States        | Home/Brands/Android/                           | 313                        |
| Somewhere in United States        | United States        | Home/Brands/YouTube/                           | 442                        |
| Somewhere in United States        | United States        | Home/Clearance Sale/                           | 1043                       |
| Somewhere in United States        | United States        | Home/Drinkware/                                | 48455                      |
| Somewhere in United States        | United States        | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Somewhere in United States        | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Somewhere in United States        | United States        | Home/Electronics/                              | 19272                      |
| Somewhere in United States        | United States        | Home/Electronics/Audio/                        | 12395                      |
| Somewhere in United States        | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Somewhere in United States        | United States        | Home/Electronics/Flashlights/                  | 704                        |
| Somewhere in United States        | United States        | Home/Electronics/Power/                        | 628                        |
| Somewhere in United States        | United States        | Home/Fun/                                      | 65                         |
| Somewhere in United States        | United States        | Home/Gift Cards/                               | 7                          |
| Somewhere in United States        | United States        | Home/Lifestyle/                                | 4885                       |
| Somewhere in United States        | United States        | Home/Lifestyle/Fun/                            | 271                        |
| Somewhere in United States        | United States        | Home/Limited Supply/                           | 11                         |
| Somewhere in United States        | United States        | Home/Limited Supply/Bags/                      | 981                        |
| Somewhere in United States        | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Somewhere in United States        | United States        | Home/Office/                                   | 52375                      |
| Somewhere in United States        | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Somewhere in United States        | United States        | Home/Office/Office Other/                      | 762                        |
| Somewhere in United States        | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Somewhere in United States        | United States        | Home/Shop by Brand/                            | 5918                       |
| Somewhere in United States        | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Somewhere in United States        | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in United States        | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in United States        | United States        | Home/Spring Sale!/                             | 1079                       |
| Somewhere in United States        | United States        | Lifestyle                                      | 40                         |
| Somewhere in United States        | United States        | Nest-USA                                       | 1007                       |
| Somewhere in United States        | United States        | Office                                         | 22                         |
| Somewhere in United States        | United States        | Waze                                           | 9                          |
| Somewhere in United States        | United States        | Wearables/Men's T-Shirts/                      | 19                         |
| Somewhere in Uruguay              | Uruguay              | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Uruguay              | Uruguay              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Venezuela            | Venezuela            | Home/Apparel/                                  | 7643                       |
| Somewhere in Venezuela            | Venezuela            | Home/Apparel/Men's/                            | 2172                       |
| Somewhere in Venezuela            | Venezuela            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Venezuela            | Venezuela            | Home/Apparel/Women's/                          | 2086                       |
| Somewhere in Venezuela            | Venezuela            | Home/Brands/Android/                           | 313                        |
| Somewhere in Venezuela            | Venezuela            | Home/Drinkware/                                | 48455                      |
| Somewhere in Venezuela            | Venezuela            | Home/Office/                                   | 52375                      |
| Somewhere in Venezuela            | Venezuela            | Home/Shop by Brand/                            | 5918                       |
| Somewhere in Venezuela            | Venezuela            | Home/Shop by Brand/Google/                     | 17021                      |
| Somewhere in Venezuela            | Venezuela            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Vietnam              | Vietnam              | Home/Accessories/                              | 5272                       |
| Somewhere in Vietnam              | Vietnam              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Somewhere in Vietnam              | Vietnam              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Somewhere in Vietnam              | Vietnam              | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Somewhere in Vietnam              | Vietnam              | Home/Bags/                                     | 11458                      |
| Somewhere in Vietnam              | Vietnam              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Somewhere in Zimbabwe             | Zimbabwe             | Home/Electronics/                              | 19272                      |
| South San Francisco               | United States        | Home/Accessories/Stickers/                     | 3390                       |
| South San Francisco               | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| South San Francisco               | United States        | Home/Bags/Backpacks/                           | 788                        |
| St. Louis                         | United States        | Home/Apparel/                                  | 7643                       |
| Stanford                          | United States        | Home/Apparel/Men's/                            | 2172                       |
| Stockholm                         | Sweden               | (not set)                                      | 21012                      |
| Stockholm                         | Sweden               | Home/Accessories/Pet/                          | 764                        |
| Stockholm                         | Sweden               | Home/Apparel/                                  | 7643                       |
| Stockholm                         | Sweden               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Stockholm                         | Sweden               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Stockholm                         | Sweden               | Home/Bags/Backpacks/                           | 788                        |
| Stockholm                         | Sweden               | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Stockholm                         | Sweden               | Home/Electronics/                              | 19272                      |
| Stockholm                         | Sweden               | Home/Office/Writing Instruments/               | 4872                       |
| Stockholm                         | Sweden               | Home/Shop by Brand/Google/                     | 17021                      |
| Stuttgart                         | Germany              | Home/Electronics/Audio/                        | 12395                      |
| Sunnyvale                         | United States        | (not set)                                      | 21012                      |
| Sunnyvale                         | United States        | Home/Accessories/                              | 5272                       |
| Sunnyvale                         | United States        | Home/Accessories/Drinkware/                    | 7306                       |
| Sunnyvale                         | United States        | Home/Accessories/Fun/                          | 4753                       |
| Sunnyvale                         | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Sunnyvale                         | United States        | Home/Accessories/Sports & Fitness/             | 318                        |
| Sunnyvale                         | United States        | Home/Accessories/Stickers/                     | 3390                       |
| Sunnyvale                         | United States        | Home/Apparel/                                  | 7643                       |
| Sunnyvale                         | United States        | Home/Apparel/Headgear/                         | 2141                       |
| Sunnyvale                         | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Sunnyvale                         | United States        | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Sunnyvale                         | United States        | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Sunnyvale                         | United States        | Home/Apparel/Men's/                            | 2172                       |
| Sunnyvale                         | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Sunnyvale                         | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Sunnyvale                         | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Sunnyvale                         | United States        | Home/Apparel/Women's/                          | 2086                       |
| Sunnyvale                         | United States        | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Sunnyvale                         | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Sunnyvale                         | United States        | Home/Bags/                                     | 11458                      |
| Sunnyvale                         | United States        | Home/Bags/Backpacks/                           | 788                        |
| Sunnyvale                         | United States        | Home/Bags/More Bags/                           | 2773                       |
| Sunnyvale                         | United States        | Home/Bags/Shopping and Totes/                  | 800                        |
| Sunnyvale                         | United States        | Home/Drinkware/                                | 48455                      |
| Sunnyvale                         | United States        | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Sunnyvale                         | United States        | Home/Electronics/                              | 19272                      |
| Sunnyvale                         | United States        | Home/Electronics/Audio/                        | 12395                      |
| Sunnyvale                         | United States        | Home/Electronics/Electronics Accessories/      | 1121                       |
| Sunnyvale                         | United States        | Home/Electronics/Power/                        | 628                        |
| Sunnyvale                         | United States        | Home/Lifestyle/                                | 4885                       |
| Sunnyvale                         | United States        | Home/Limited Supply/Bags/                      | 981                        |
| Sunnyvale                         | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Sunnyvale                         | United States        | Home/Office/                                   | 52375                      |
| Sunnyvale                         | United States        | Home/Office/Notebooks & Journals/              | 32307                      |
| Sunnyvale                         | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Sunnyvale                         | United States        | Home/Shop by Brand/                            | 5918                       |
| Sunnyvale                         | United States        | Home/Shop by Brand/Android/                    | 8295                       |
| Sunnyvale                         | United States        | Home/Shop by Brand/Google/                     | 17021                      |
| Sunnyvale                         | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Sunnyvale                         | United States        | Home/Spring Sale!/                             | 1079                       |
| Sunnyvale                         | United States        | Housewares                                     | 60                         |
| Sunnyvale                         | United States        | Nest-USA                                       | 1007                       |
| Sydney                            | Australia            | (not set)                                      | 21012                      |
| Sydney                            | Australia            | Home/Accessories/Stickers/                     | 3390                       |
| Sydney                            | Australia            | Home/Apparel/                                  | 7643                       |
| Sydney                            | Australia            | Home/Apparel/Kid's/                            | 1450                       |
| Sydney                            | Australia            | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Sydney                            | Australia            | Home/Apparel/Kid's/Kid's-Toddler/              | 508                        |
| Sydney                            | Australia            | Home/Apparel/Kid's/Kids-Youth/                 | 697                        |
| Sydney                            | Australia            | Home/Apparel/Men's/                            | 2172                       |
| Sydney                            | Australia            | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Sydney                            | Australia            | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Sydney                            | Australia            | Home/Apparel/Women's/                          | 2086                       |
| Sydney                            | Australia            | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Sydney                            | Australia            | Home/Bags/                                     | 11458                      |
| Sydney                            | Australia            | Home/Bags/Shopping and Totes/                  | 800                        |
| Sydney                            | Australia            | Home/Drinkware/                                | 48455                      |
| Sydney                            | Australia            | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Sydney                            | Australia            | Home/Electronics/                              | 19272                      |
| Sydney                            | Australia            | Home/Lifestyle/                                | 4885                       |
| Sydney                            | Australia            | Home/Office/Notebooks & Journals/              | 32307                      |
| Sydney                            | Australia            | Home/Office/Office Other/                      | 762                        |
| Sydney                            | Australia            | Home/Shop by Brand/                            | 5918                       |
| Sydney                            | Australia            | Home/Shop by Brand/Google/                     | 17021                      |
| Sydney                            | Australia            | Home/Shop by Brand/YouTube/                    | 346065                     |
| Sydney                            | Australia            | Nest-USA                                       | 1007                       |
| Taguig                            | Philippines          | Home/Accessories/Fun/                          | 4753                       |
| Taguig                            | Philippines          | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Taguig                            | Philippines          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Taguig                            | Philippines          | Home/Office/Writing Instruments/               | 4872                       |
| Taguig                            | Philippines          | Home/Shop by Brand/YouTube/                    | 346065                     |
| Tel Aviv-Yafo                     | Israel               | Home/Accessories/Drinkware/                    | 7306                       |
| Tel Aviv-Yafo                     | Israel               | Home/Apparel/                                  | 7643                       |
| Tel Aviv-Yafo                     | Israel               | Home/Apparel/Men's/                            | 2172                       |
| Tel Aviv-Yafo                     | Israel               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Tel Aviv-Yafo                     | Israel               | Home/Apparel/Women's/Women's-Outerwear/        | 1068                       |
| Tel Aviv-Yafo                     | Israel               | Home/Bags/                                     | 11458                      |
| Tel Aviv-Yafo                     | Israel               | Home/Bags/Backpacks/                           | 788                        |
| Tel Aviv-Yafo                     | Israel               | Home/Drinkware/                                | 48455                      |
| Tel Aviv-Yafo                     | Israel               | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Tel Aviv-Yafo                     | Israel               | Home/Electronics/                              | 19272                      |
| Tel Aviv-Yafo                     | Israel               | Home/Electronics/Power/                        | 628                        |
| Tel Aviv-Yafo                     | Israel               | Home/Office/Notebooks & Journals/              | 32307                      |
| Tel Aviv-Yafo                     | Israel               | Home/Shop by Brand/                            | 5918                       |
| Tel Aviv-Yafo                     | Israel               | Home/Shop by Brand/Android/                    | 8295                       |
| Tel Aviv-Yafo                     | Israel               | Home/Shop by Brand/Google/                     | 17021                      |
| Tel Aviv-Yafo                     | Israel               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Tempe                             | United States        | Home/Accessories/Fun/                          | 4753                       |
| Tempe                             | United States        | Home/Accessories/Housewares/                   | 1958                       |
| Tempe                             | United States        | Home/Apparel/                                  | 7643                       |
| Tempe                             | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| The Dalles                        | United States        | Home/Electronics/                              | 19272                      |
| Thessaloniki                      | Greece               | Home/Shop by Brand/                            | 5918                       |
| Timisoara                         | Romania              | Home/Electronics/                              | 19272                      |
| Toronto                           | Canada               | ${escCatTitle}                                 | 765                        |
| Toronto                           | Canada               | (not set)                                      | 21012                      |
| Toronto                           | Canada               | Home/Accessories/                              | 5272                       |
| Toronto                           | Canada               | Home/Accessories/Drinkware/                    | 7306                       |
| Toronto                           | Canada               | Home/Accessories/Fun/                          | 4753                       |
| Toronto                           | Canada               | Home/Accessories/Stickers/                     | 3390                       |
| Toronto                           | Canada               | Home/Apparel/                                  | 7643                       |
| Toronto                           | Canada               | Home/Apparel/Headgear/                         | 2141                       |
| Toronto                           | Canada               | Home/Apparel/Men's/                            | 2172                       |
| Toronto                           | Canada               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Toronto                           | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Toronto                           | Canada               | Home/Bags/                                     | 11458                      |
| Toronto                           | Canada               | Home/Bags/Backpacks/                           | 788                        |
| Toronto                           | Canada               | Home/Brands/YouTube/                           | 442                        |
| Toronto                           | Canada               | Home/Drinkware/                                | 48455                      |
| Toronto                           | Canada               | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Toronto                           | Canada               | Home/Electronics/                              | 19272                      |
| Toronto                           | Canada               | Home/Electronics/Electronics Accessories/      | 1121                       |
| Toronto                           | Canada               | Home/Fun/                                      | 65                         |
| Toronto                           | Canada               | Home/Lifestyle/                                | 4885                       |
| Toronto                           | Canada               | Home/Office/                                   | 52375                      |
| Toronto                           | Canada               | Home/Office/Notebooks & Journals/              | 32307                      |
| Toronto                           | Canada               | Home/Shop by Brand/                            | 5918                       |
| Toronto                           | Canada               | Home/Shop by Brand/Android/                    | 8295                       |
| Toronto                           | Canada               | Home/Shop by Brand/Google/                     | 17021                      |
| Toronto                           | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| University Park                   | United States        | Home/Bags/Backpacks/                           | 788                        |
| Unknown                           | Unknown              | Home/Accessories/Stickers/                     | 3390                       |
| Unknown                           | Unknown              | Home/Apparel/                                  | 7643                       |
| Unknown                           | Unknown              | Home/Apparel/Men's/                            | 2172                       |
| Unknown                           | Unknown              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Unknown                           | Unknown              | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Unknown                           | Unknown              | Home/Bags/                                     | 11458                      |
| Unknown                           | Unknown              | Home/Drinkware/                                | 48455                      |
| Unknown                           | Unknown              | Home/Electronics/Audio/                        | 12395                      |
| Unknown                           | Unknown              | Home/Shop by Brand/YouTube/                    | 346065                     |
| Vancouver                         | Canada               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Vancouver                         | Canada               | Home/Bags/Backpacks/                           | 788                        |
| Vancouver                         | Canada               | Home/Electronics/                              | 19272                      |
| Vancouver                         | Canada               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Vienna                            | Austria              | Home/Apparel/Men's/                            | 2172                       |
| Vienna                            | Austria              | Home/Shop by Brand/Google/                     | 17021                      |
| Villeneuve-d'Ascq                 | France               | Home/Shop by Brand/Google/                     | 17021                      |
| Vilnius                           | Lithuania            | Home/Apparel/Men's/                            | 2172                       |
| Vladivostok                       | Russia               | Home/Limited Supply/Bags/                      | 981                        |
| Warsaw                            | Poland               | (not set)                                      | 21012                      |
| Warsaw                            | Poland               | Home/Accessories/                              | 5272                       |
| Warsaw                            | Poland               | Home/Accessories/Pet/                          | 764                        |
| Warsaw                            | Poland               | Home/Accessories/Stickers/                     | 3390                       |
| Warsaw                            | Poland               | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Warsaw                            | Poland               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Warsaw                            | Poland               | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Warsaw                            | Poland               | Home/Drinkware/Mugs and Cups/                  | 133                        |
| Warsaw                            | Poland               | Home/Electronics/                              | 19272                      |
| Warsaw                            | Poland               | Home/Office/                                   | 52375                      |
| Warsaw                            | Poland               | Home/Shop by Brand/Android/                    | 8295                       |
| Warsaw                            | Poland               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Washington                        | United States        | Home/Accessories/Pet/                          | 764                        |
| Washington                        | United States        | Home/Apparel/                                  | 7643                       |
| Washington                        | United States        | Home/Apparel/Kid's/Kid's-Infant/               | 939                        |
| Washington                        | United States        | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Washington                        | United States        | Home/Apparel/Men's/Men's-Performance Wear/     | 309                        |
| Washington                        | United States        | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Washington                        | United States        | Home/Apparel/Women's/Women's-T-Shirts/         | 3473                       |
| Washington                        | United States        | Home/Bags/                                     | 11458                      |
| Washington                        | United States        | Home/Bags/Backpacks/                           | 788                        |
| Washington                        | United States        | Home/Drinkware/                                | 48455                      |
| Washington                        | United States        | Home/Electronics/                              | 19272                      |
| Washington                        | United States        | Home/Electronics/Audio/                        | 12395                      |
| Washington                        | United States        | Home/Nest/Nest-USA/                            | 24443                      |
| Washington                        | United States        | Home/Office/                                   | 52375                      |
| Washington                        | United States        | Home/Shop by Brand/YouTube/                    | 346065                     |
| Wellesley                         | United States        | Home/Office/Writing Instruments/               | 4872                       |
| Westville                         | South Africa         | Home/Bags/                                     | 11458                      |
| Wrexham                           | United Kingdom       | Home/Shop by Brand/                            | 5918                       |
| Yokohama                          | Japan                | (not set)                                      | 21012                      |
| Yokohama                          | Japan                | Home/Apparel/                                  | 7643                       |
| Yokohama                          | United States        | Home/Apparel/                                  | 7643                       |
| Yokohama                          | Japan                | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Yokohama                          | Japan                | Home/Shop by Brand/Google/                     | 17021                      |
| Zagreb                            | Croatia              | Home/Apparel/Men's/                            | 2172                       |
| Zagreb                            | Croatia              | Home/Apparel/Men's/Men's-Outerwear/            | 3818                       |
| Zhongli District                  | Taiwan               | (not set)                                      | 21012                      |
| Zhongli District                  | Taiwan               | Home/Apparel/                                  | 7643                       |
| Zhongli District                  | Taiwan               | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Zhongli District                  | Taiwan               | Home/Shop by Brand/YouTube/                    | 346065                     |
| Zurich                            | Switzerland          | (not set)                                      | 21012                      |
| Zurich                            | Switzerland          | Home/Apparel/Men's/                            | 2172                       |
| Zurich                            | Switzerland          | Home/Apparel/Men's/Men's-T-Shirts/             | 10412                      |
| Zurich                            | Switzerland          | Home/Bags/                                     | 11458                      |
| Zurich                            | Switzerland          | Home/Drinkware/Water Bottles and Tumblers/     | 14447                      |
| Zurich                            | Switzerland          | Home/Electronics/                              | 19272                      |
| Zurich                            | Switzerland          | Home/Electronics/Electronics Accessories/      | 1121                       |
| Zurich                            | Switzerland          | Home/Shop by Brand/YouTube/                    | 346065                     |



# Question 4: What is the top-selling product from each city/country? Can we find any pattern worthy of noting in the products sold?

**Solution Note**:  I used the same base CTEs to join together and clean data as Q2 and Q3, with some changes.  The assumptions from Q2 and Q3 hold.  In this question, I also did a bit of data cleaning within the CTEs by using the products.name if it existed, and if it did not, using the allsessions.v2productname as well.  In my final query, I removed any rows where the num_ordered was not > 0 since a product cannot be considered a top-selling product if it was never ordered!

### SQL Queries:
```SQL
WITH city_country_sordered_pordered AS (
	SELECT
		city,
		country,
		alls.productsku AS sessions_sku,
		s.productsku AS sales_sku,
		p.name AS productname,
		alls.v2productname AS sessions_name,
	    alls.v2productcategory AS category,
		s.total_ordered AS sales_ordered,
		p.sku AS products_sku,
		p.orderedquantity as products_ordered
    FROM allsessions_clean alls
    LEFT JOIN salesbysku s
    ON alls.productsku = s.productsku
    LEFT JOIN products_clean p
    ON alls.productsku = p.sku
)
,
city_country_sku_clean_ordered AS (
	SELECT
		city,
		country,
		sessions_sku,
		productname,
		sessions_name,
		category,
		CASE
			WHEN sales_ordered IS NOT NULL AND sales_ordered > 0 THEN sales_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NOT NULL THEN products_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NULL THEN 0::int
			ELSE 0::int
		END AS ordered
	FROM city_country_sordered_pordered
)
,
clean_city_country_sku_clean_ordered AS (
	SELECT
		CASE
			WHEN (city IS NULL OR city = '(not set)' OR city = 'not available in demo dataset') AND (country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset') THEN 'Unknown'
			WHEN city = '(not set)' OR city = 'not available in demo dataset' THEN 'Somewhere in ' || country
			ELSE city
		END AS city,
		CASE
			WHEN country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset' THEN 'Unknown'
			ELSE country
		END AS country,
		sessions_sku,
		productname,
		sessions_name,
		category,
		ordered
	FROM city_country_sku_clean_ordered
)
,
clean_city_country_sku_clean_name_clean_ordered AS (
	SELECT
		city,
		country,
		sessions_sku,
		CASE
			WHEN productname IS NULL OR productname = '' THEN sessions_name
			ELSE productname
		END AS name,
		category,
		ordered
	FROM clean_city_country_sku_clean_ordered
)
,
clean_city_country_sku_clean_ordered_clean_name_ranked_on_order AS (
	SELECT
		city,
		country,
		sessions_sku AS product_sku_from_allsessions,
		ordered AS num_ordered,
		name AS productname,
		RANK() OVER (partition by city, country ORDER BY ordered DESC) AS rank_on_num_ordered
	FROM clean_city_country_sku_clean_name_clean_ordered
)

SELECT DISTINCT * 
FROM clean_city_country_sku_clean_ordered_clean_name_ranked_on_order
WHERE num_ordered > 0 AND rank_on_num_ordered = 1
ORDER BY city, country
```


## Answer:

There isn't an **obvious** pattern worthy of noting in my final result set.  It appears that people in the United States really like their leatherette notebook combos and generally like office supplies.  These journals and writing books seem to show up as top order in other parts of the world too.  Further analysis incorporating categories would be interesting.  It would also be good to do histogram and other graphical and statistical analysis to find patterns that are not discerable to the human eye but may be statsitically significant, nevertheless.  The result set also highlights that the data set requires a deeper dive on order quantities.  The choices I made, to use the salesbysku total_ordered first, and then the products.orderedquantity next, and to ignore the allsessions.unitssold altogether, may not be correct, and that would skew the results shown in this result set.

The result set from the above query is as follows:
| city                              | country              | product_sku_from_allsessions | num_ordered | productname                                        | rank_on_num_ordered |
|-----------------------------------|----------------------|------------------------------|-------------|----------------------------------------------------|---------------------|
| Adelaide                          | Australia            | GGOEGAAX0568                 | 3           | Men's Watershed Full Zip Hoodie Grey               | 1                   |
| Ahmedabad                         | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Akron                             | United States        | GGOEGAAX0106                 | 13          | Men's 100% Cotton Short Sleeve Hero Tee Navy       | 1                   |
| Amsterdam                         | Netherlands          | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Ann Arbor                         | United States        | GGOEGFYQ016599               | 253         | Foam Can and Bottle Cooler                         | 1                   |
| Antwerp                           | Belgium              | GGOEGOBG023599               | 3           | Colored Pencil Set                                 | 1                   |
| Appleton                          | United States        | GGOEGESC014699               | 66          | Aluminum Handy Emergency Flashlight                | 1                   |
| Ashburn                           | United States        | GGOEGFQB013799               | 5           | Compact Selfie Stick                               | 1                   |
| Asuncion                          | Paraguay             | GGOEGAAX0106                 | 13          | Men's 100% Cotton Short Sleeve Hero Tee Navy       | 1                   |
| Athens                            | Greece               | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Atlanta                           | United States        | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Austin                            | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Avon                              | United States        | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Bandung                           | Indonesia            | GGOEGAAX0325                 | 6           | Men's Short Sleeve Hero Tee Charcoal               | 1                   |
| Bangkok                           | Thailand             | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Barcelona                         | Spain                | GGOEGDHQ015399               | 97          | 26 oz Double Wall Insulated Bottle                 | 1                   |
| Bellflower                        | United States        | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Bellingham                        | United States        | GGOENEBJ079499               | 94          | Learning Thermostat 3rd Gen-USA - Stainless Steel  | 1                   |
| Belo Horizonte                    | Brazil               | GGOEGBJC019999               | 43          | Collapsible Shopping Bag                           | 1                   |
| Bengaluru                         | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Berkeley                          | United States        | GGOEGAAX0614                 | 25          | Onesie Heather                                     | 1                   |
| Berlin                            | Germany              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Bhubaneswar                       | India                | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| Bilbao                            | Spain                | GGOEGAAX0291                 | 27          | Women's Short Sleeve Hero Tee Sky Blue             | 1                   |
| Bogota                            | Colombia             | GGOENEBJ079499               | 94          | Learning Thermostat 3rd Gen-USA - Stainless Steel  | 1                   |
| Boston                            | United States        | GGOEGOAQ012899               | 456         | Ballpoint LED Light Pen                            | 1                   |
| Boulder                           | United States        | GGOEGFAQ016699               | 11          | Bottle Opener Clip                                 | 1                   |
| Brisbane                          | Australia            | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Brno                              | Czechia              | GGOEGOCB017499               | 319         | Leatherette Journal                                | 1                   |
| Brussels                          | Belgium              | GGOEGAAX0105                 | 4           | Men's 100% Cotton Short Sleeve Hero Tee Black      | 1                   |
| Bucharest                         | Romania              | GGOEGBJL013999               | 62          | Canvas Tote Natural/Navy                           | 1                   |
| Budapest                          | Hungary              | GGOEGODR017799               | 39          | Recycled Mouse Pad                                 | 1                   |
| Buenos Aires                      | Argentina            | GGOEGBMJ013399               | 70          | Sport Bag                                          | 1                   |
| Burnaby                           | Canada               | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Calgary                           | Canada               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Cambridge                         | United States        | GGOENEBQ078999               | 112         | Cam Outdoor Security Camera - USA                  | 1                   |
| Cape Town                         | South Africa         | GGOEGAAX0362                 | 25          | Men's Pullover Hoodie Grey                         | 1                   |
| Chandigarh                        | India                | GGOEGAAX0106                 | 13          | Men's 100% Cotton Short Sleeve Hero Tee Navy       | 1                   |
| Charlotte                         | United States        | GGOEGAAX0098                 | 104         | 7 Dog Frisbee                                      | 1                   |
| Chennai                           | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Chicago                           | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Chico                             | United States        | GGOEGAAX0334                 | 2           | Android Men's Long Sleeve Badge Crew Tee Heather   | 1                   |
| Cluj-Napoca                       | Romania              | GGOEGAAX0105                 | 4           | Men's 100% Cotton Short Sleeve Hero Tee Black      | 1                   |
| Colombo                           | Sri Lanka            | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Columbia                          | United States        | GGOEGCBC074299               | 5           | Device Stand                                       | 1                   |
| Columbus                          | United States        | GGOEGAAX0359                 | 12          | Android Men's  Zip Hoodie                          | 1                   |
| Copenhagen                        | Denmark              | GGOEGESC014099               | 16          | Rocket Flashlight                                  | 1                   |
| Cork                              | Ireland              | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Council Bluffs                    | United States        | GGOEGFSR022099               | 3           | Kick Ball                                          | 1                   |
| Courbevoie                        | France               | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Culiacan                          | Mexico               | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| Cupertino                         | United States        | GGOENEBB078899               | 102         | Cam Indoor Security Camera - USA                   | 1                   |
| Dallas                            | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Denver                            | United States        | GGOEGPJC203399               | 14          | Crunch Noise Dog Toy                               | 1                   |
| Detroit                           | United States        | GGOEGDHC018299               | 23          | 22 oz Water Bottle                                 | 1                   |
| Doha                              | Qatar                | GGOEGAAX0105                 | 4           | Men's 100% Cotton Short Sleeve Hero Tee Black      | 1                   |
| Druid Hills                       | United States        | GGOEGOBG023599               | 3           | Colored Pencil Set                                 | 1                   |
| Dubai                             | United Arab Emirates | GGOEGOCB017499               | 319         | Leatherette Journal                                | 1                   |
| Dublin                            | Ireland              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Dublin                            | Netherlands          | GGOEGOFH020299               | 12          | Screen Cleaning Cloth                              | 1                   |
| East Lansing                      | United States        | GGOEGAAX0795                 | 7           | 25L Classic Rucksack                               | 1                   |
| Eau Claire                        | United States        | GGOEGAAX0291                 | 27          | Women's Short Sleeve Hero Tee Sky Blue             | 1                   |
| Edmonton                          | Canada               | GGOEGOAR013099               | 2           | Stylus Pen w/ LED Light                            | 1                   |
| Egham                             | United Kingdom       | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| El Paso                           | United States        | GGOEGAAX0321                 | 2           | Men's Short Sleeve Hero Tee Light Blue             | 1                   |
| Frankfurt                         | Germany              | GGOEGKWQ060910               | 7           | Bib White                                          | 1                   |
| Fremont                           | United States        | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Ghent                             | Belgium              | GGOEAKDH019899               | 16          | Windup Android                                     | 1                   |
| Glasgow                           | United Kingdom       | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Greer                             | United States        | GGOEGAAX0596                 | 1           | Men's Quilted Insulated Vest Black                 | 1                   |
| Gurgaon                           | India                | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Hamburg                           | Germany              | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Hanoi                             | Vietnam              | GGOEGDHQ015399               | 97          | 26 oz Double Wall Insulated Bottle                 | 1                   |
| Helsinki                          | Finland              | GGOEGAAX0617                 | 6           | Android Onesie Gold                                | 1                   |
| Ho Chi Minh City                  | Vietnam              | GGOEGFKA022299               | 53          | Keyboard DOT Sticker                               | 1                   |
| Hong Kong                         | Hong Kong            | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Hong Kong                         | United States        | GGOEAFKQ020599               | 11          | Android Sticker Sheet Ultra Removable              | 1                   |
| Houston                           | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Hyderabad                         | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Iasi                              | Romania              | GGOEGAAX0351                 | 2           | Men's Vintage Henley                               | 1                   |
| Indore                            | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Ipoh                              | Malaysia             | GGOEGFAQ016699               | 11          | Bottle Opener Clip                                 | 1                   |
| Irvine                            | United States        | GGOEADHH073999               | 167         | Android 17oz Stainless Steel Sport Bottle          | 1                   |
| Istanbul                          | Hungary              | GGOEAFKQ020599               | 11          | Android Sticker Sheet Ultra Removable              | 1                   |
| Istanbul                          | Turkey               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Izmir                             | Turkey               | GGOEGAAX0279                 | 4           | Women's Short Sleeve Hero Tee White                | 1                   |
| Jacksonville                      | United States        | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| Jaipur                            | India                | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Jakarta                           | Indonesia            | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Jersey City                       | United States        | GGOEGAAX0690                 | 7           | Youth Short Sleeve Tee White                       | 1                   |
| Kalamazoo                         | United States        | GGOEGOAB022499               | 105         | Satin Black Ballpoint Pen                          | 1                   |
| Kansas City                       | United States        | GGOEGAAX0297                 | 1           | Women's Short Sleeve Hero Tee Red Heather          | 1                   |
| Kansas City                       | United States        | GGOEGAAX0733                 | 1           | Women's Short Sleeve Hero Tee Heather              | 1                   |
| Karachi                           | Pakistan             | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Kharagpur                         | India                | GGOEGAAX0313                 | 11          | Tri-blend Hoodie Grey                              | 1                   |
| Kharkiv                           | Ukraine              | GGOEGAAX0299                 | 4           | Women's V-Neck Tee Charcoal                        | 1                   |
| Kiev                              | Ukraine              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Kirkland                          | United States        | GGOENEBJ079499               | 94          | Learning Thermostat 3rd Gen-USA - Stainless Steel  | 1                   |
| Kitchener                         | Canada               | GGOEGFKA022299               | 53          | Keyboard DOT Sticker                               | 1                   |
| Kolkata                           | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Kovrov                            | Russia               | GGOEGAAX0360                 | 58          | Women's Fleece Hoodie                              | 1                   |
| Kuala Lumpur                      | Malaysia             | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| La Victoria                       | Peru                 | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| LaFayette                         | United States        | GGOEGAAX0106                 | 13          | Men's 100% Cotton Short Sleeve Hero Tee Navy       | 1                   |
| Lahore                            | Pakistan             | GGOEGETR014599               | 1           | Tube Power Bank                                    | 1                   |
| Lake Oswego                       | United States        | GGOEGODR017799               | 39          | Recycled Mouse Pad                                 | 1                   |
| Las Vegas                         | United States        | GGOEGAAX0105                 | 4           | Men's 100% Cotton Short Sleeve Hero Tee Black      | 1                   |
| Lisbon                            | Portugal             | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| London                            | United Kingdom       | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| London                            | United States        | GGOEGAAX0614                 | 25          | Onesie Heather                                     | 1                   |
| Longtan District                  | Taiwan               | GGOEGDHQ015399               | 97          | 26 oz Double Wall Insulated Bottle                 | 1                   |
| Los Angeles                       | Australia            | GGOEGHPB071610               | 3           | Twill Cap                                          | 1                   |
| Los Angeles                       | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Madison                           | United States        | GGOEGBJC014399               | 3           | Tote Bag                                           | 1                   |
| Madrid                            | Spain                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Makati                            | Philippines          | GGOEGAAX0321                 | 2           | Men's Short Sleeve Hero Tee Light Blue             | 1                   |
| Manila                            | Philippines          | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Maracaibo                         | Venezuela            | GGOEGAAX0352                 | 2           | Android Men's Vintage Henley                       | 1                   |
| Marseille                         | France               | GGOEGAAX0105                 | 4           | Men's 100% Cotton Short Sleeve Hero Tee Black      | 1                   |
| Medellin                          | Colombia             | GGOEGFAQ016699               | 11          | Bottle Opener Clip                                 | 1                   |
| Melbourne                         | Australia            | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Menlo Park                        | United States        | GGOENEBB078899               | 102         | Cam Indoor Security Camera - USA                   | 1                   |
| Mexico City                       | Mexico               | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Milan                             | Italy                | GGOEGOLC013299               | 69          | Spiral Notebook and Pen Set                        | 1                   |
| Milpitas                          | United States        | GGOEGAAX0291                 | 27          | Women's Short Sleeve Hero Tee Sky Blue             | 1                   |
| Minato                            | Japan                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Mississauga                       | Canada               | GGOEGBJC019999               | 43          | Collapsible Shopping Bag                           | 1                   |
| Monterrey                         | Mexico               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Montevideo                        | Uruguay              | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Montreal                          | Canada               | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Montreuil                         | France               | GGOEGBJC019999               | 43          | Collapsible Shopping Bag                           | 1                   |
| Moscow                            | Russia               | GGOEGCBQ016499               | 60          | SPF-15 Slim & Slender Lip Balm                     | 1                   |
| Mountain View                     | Australia            | GGOEGAAX0569                 | 1           | Men's Performance Full Zip Jacket Black            | 1                   |
| Mountain View                     | Japan                | GGOEGHPB080410               | 7           | 5-Panel Snapback Cap                               | 1                   |
| Mountain View                     | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Mumbai                            | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Munich                            | Germany              | GGOEGFYQ016599               | 253         | Foam Can and Bottle Cooler                         | 1                   |
| Nanded                            | India                | GGOEGAAX0325                 | 6           | Men's Short Sleeve Hero Tee Charcoal               | 1                   |
| Nashville                         | United States        | GGOENEBJ079499               | 94          | Learning Thermostat 3rd Gen-USA - Stainless Steel  | 1                   |
| Neipu Township                    | Taiwan               | GGOEGAAX0360                 | 58          | Women's Fleece Hoodie                              | 1                   |
| New Delhi                         | India                | GGOEGEVB071799               | 214         | Pocket Bluetooth Speaker                           | 1                   |
| New York                          | Canada               | GGOEGAAX0358                 | 3           | Men's  Zip Hoodie                                  | 1                   |
| New York                          | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Norfolk                           | United States        | GGOEGCMB020932               | 5           | Suitcase Organizer Cubes                           | 1                   |
| Oakland                           | United States        | GGOEGFKA022299               | 53          | Keyboard DOT Sticker                               | 1                   |
| Orlando                           | United States        | GGOEGAAX0284                 | 22          | Women's  Short Sleeve Hero Tee   Black             | 1                   |
| Osaka                             | Japan                | GGOENEBQ079199               | 42          | Protect Smoke + CO White Wired Alarm-USA           | 1                   |
| Oslo                              | Norway               | GGOEGAAX0107                 | 1           | Men's 100% Cotton Short Sleeve Hero Tee Red        | 1                   |
| Ottawa                            | Canada               | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Oviedo                            | United States        | GGOEGHPB080410               | 7           | 5-Panel Snapback Cap                               | 1                   |
| Oxford                            | United Kingdom       | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Palo Alto                         | United States        | GGOEGOAQ012899               | 456         | Ballpoint LED Light Pen                            | 1                   |
| Paris                             | France               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Parsippany-Troy Hills             | United States        | GGOEGAAX0573                 | 18          | Men's Lightweight Microfleece Jacket Black         | 1                   |
| Patna                             | India                | GGOEADHH055999               | 2           | 22 oz Android Bottle                               | 1                   |
| Perth                             | Australia            | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Petaling Jaya                     | Malaysia             | GGOEGAAX0278                 | 3           | Women's Short Sleeve Hero Tee Black                | 1                   |
| Petaling Jaya                     | Malaysia             | GGOEGAAX0602                 | 3           | Women's 1/4 Zip Performance Pullover Black         | 1                   |
| Philadelphia                      | United States        | GGOEGOCB017499               | 319         | Leatherette Journal                                | 1                   |
| Phoenix                           | United States        | GGOENEBB078899               | 102         | Cam Indoor Security Camera - USA                   | 1                   |
| Piscataway Township               | United States        | GGOEGAAX0107                 | 1           | Men's 100% Cotton Short Sleeve Hero Tee Red        | 1                   |
| Pittsburgh                        | United States        | GGOEGDHJ082599               | 456         | 17 oz Double Wall Stainless Steel Insulated Bottle | 1                   |
| Portland                          | United States        | GGOEGAAX0106                 | 13          | Men's 100% Cotton Short Sleeve Hero Tee Navy       | 1                   |
| Poznan                            | Poland               | GGOEGEVB070899               | 85          | Bongo Cupholder Bluetooth Speaker                  | 1                   |
| Pozuelo de Alarcon                | Spain                | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Prague                            | Czechia              | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Pune                              | India                | GGOEGOAQ012899               | 456         | Ballpoint LED Light Pen                            | 1                   |
| Quebec City                       | Canada               | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Quezon City                       | Philippines          | GGOEGAAX0106                 | 13          | Men's 100% Cotton Short Sleeve Hero Tee Navy       | 1                   |
| Redmond                           | United States        | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Redwood City                      | United States        | GGOENEBB078899               | 102         | Cam Indoor Security Camera - USA                   | 1                   |
| Rexburg                           | United States        | GGOEGDHC074099               | 334         | 17oz Stainless Steel Sport Bottle                  | 1                   |
| Richardson                        | United States        | GGOEGEVA022399               | 4           | Micro Wireless Earbud                              | 1                   |
| Rio de Janeiro                    | Brazil               | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Riyadh                            | Saudi Arabia         | GGOEGOCB017499               | 319         | Leatherette Journal                                | 1                   |
| Rome                              | Italy                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Rosario                           | Argentina            | GGOEGAAX0338                 | 1           | Men's Vintage Badge Tee Black                      | 1                   |
| Sacramento                        | United States        | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Saint Petersburg                  | Russia               | GGOEGOCB017499               | 319         | Leatherette Journal                                | 1                   |
| Sakai                             | Japan                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Salem                             | United States        | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| Salford                           | United Kingdom       | GGOEGCBB074199               | 2           | Car Clip Phone Holder                              | 1                   |
| San Antonio                       | United States        | GGOEADHH073999               | 167         | Android 17oz Stainless Steel Sport Bottle          | 1                   |
| San Bruno                         | United States        | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| San Diego                         | United States        | GGOEGBMB073799               | 162         | Zipper-front Sports Bag                            | 1                   |
| San Francisco                     | France               | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| San Francisco                     | United States        | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| San Jose                          | United States        | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| San Mateo                         | United States        | GGOEGAAX0313                 | 11          | Tri-blend Hoodie Grey                              | 1                   |
| Sandton                           | South Africa         | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Santa Clara                       | United States        | GGOEGOAQ012899               | 456         | Ballpoint LED Light Pen                            | 1                   |
| Santa Fe                          | Argentina            | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Santa Monica                      | United States        | GGOEGBPB021199               | 13          | Slim Utility Travel Bag                            | 1                   |
| Santiago                          | Chile                | GGOEGBMJ013399               | 70          | Sport Bag                                          | 1                   |
| Sao Paulo                         | Brazil               | GGOEGOCT019199               | 316         | Red Spiral  Notebook                               | 1                   |
| Sapporo                           | Japan                | GGOEGAAX0663                 | 12          | Android Toddler Short Sleeve T-shirt Pink          | 1                   |
| Seattle                           | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Seoul                             | South Korea          | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Sherbrooke                        | Canada               | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Shibuya                           | Japan                | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Shinjuku                          | Japan                | GGOEGDHC074099               | 334         | 17oz Stainless Steel Sport Bottle                  | 1                   |
| Singapore                         | France               | GGOEGAAX0338                 | 1           | Men's Vintage Badge Tee Black                      | 1                   |
| Singapore                         | Singapore            | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Somewhere in Albania              | Albania              | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Algeria              | Algeria              | GGOEGAAX0659                 | 12          | Toddler Raglan Shirt Blue Heather/Navy             | 1                   |
| Somewhere in Argentina            | Argentina            | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Somewhere in Armenia              | Armenia              | GGOEAKDH019899               | 16          | Windup Android                                     | 1                   |
| Somewhere in Australia            | Australia            | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Austria              | Austria              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Bahamas              | Bahamas              | GGOEGHPB071610               | 3           | Twill Cap                                          | 1                   |
| Somewhere in Bahrain              | Bahrain              | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Somewhere in Bangladesh           | Bangladesh           | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Barbados             | Barbados             | GGOEGAAX0362                 | 25          | Men's Pullover Hoodie Grey                         | 1                   |
| Somewhere in Belarus              | Belarus              | GGOEGAAX0291                 | 27          | Women's Short Sleeve Hero Tee Sky Blue             | 1                   |
| Somewhere in Belgium              | Belgium              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Bolivia              | Bolivia              | GGOEGAAX0319                 | 18          | Android Men's Short Sleeve Hero Tee Heather        | 1                   |
| Somewhere in Bosnia & Herzegovina | Bosnia & Herzegovina | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Botswana             | Botswana             | GGOEGBRA037499               | 5           | Waterproof Backpack                                | 1                   |
| Somewhere in Brazil               | Brazil               | GGOEGFYQ016599               | 253         | Foam Can and Bottle Cooler                         | 1                   |
| Somewhere in Brunei               | Brunei               | GGOEGAAX0351                 | 2           | Men's Vintage Henley                               | 1                   |
| Somewhere in Brunei               | Brunei               | GGOEGAAX0661                 | 2           | Toddler Short Sleeve Tee Red                       | 1                   |
| Somewhere in Bulgaria             | Bulgaria             | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| Somewhere in Cambodia             | Cambodia             | GGOEGAAX0284                 | 22          | Women's  Short Sleeve Hero Tee   Black             | 1                   |
| Somewhere in Canada               | Canada               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Chile                | Chile                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in China                | China                | GGOEGBMB073799               | 162         | Zipper-front Sports Bag                            | 1                   |
| Somewhere in Colombia             | Colombia             | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Costa Rica           | Costa Rica           | GGOEGETB023799               | 56          | Power Bank                                         | 1                   |
| Somewhere in Côte d’Ivoire        | Côte d’Ivoire        | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Somewhere in Croatia              | Croatia              | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Cyprus               | Cyprus               | GGOEGCMB020932               | 5           | Suitcase Organizer Cubes                           | 1                   |
| Somewhere in Czechia              | Czechia              | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Denmark              | Denmark              | GGOEGOCT019199               | 316         | Red Spiral  Notebook                               | 1                   |
| Somewhere in Dominican Republic   | Dominican Republic   | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Ecuador              | Ecuador              | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Egypt                | Egypt                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in El Salvador          | El Salvador          | GGOEGAAX0343                 | 33          | Women's Vintage Hero Tee Lavender                  | 1                   |
| Somewhere in Estonia              | Estonia              | GGOEGBMB073799               | 162         | Zipper-front Sports Bag                            | 1                   |
| Somewhere in Ethiopia             | Ethiopia             | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Finland              | Finland              | GGOEGFKA022299               | 53          | Keyboard DOT Sticker                               | 1                   |
| Somewhere in France               | France               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Georgia              | Georgia              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Germany              | Germany              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Ghana                | Ghana                | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Somewhere in Gibraltar            | Gibraltar            | GGOEGAAX0351                 | 2           | Men's Vintage Henley                               | 1                   |
| Somewhere in Greece               | Greece               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Guatemala            | Guatemala            | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Somewhere in Haiti                | Haiti                | GGOEGAAX0734                 | 1           | Men's Short Sleeve Hero Tee Heather                | 1                   |
| Somewhere in Honduras             | Honduras             | GGOENEBQ078999               | 112         | Cam Outdoor Security Camera - USA                  | 1                   |
| Somewhere in Hong Kong            | Hong Kong            | GGOEGOCB017499               | 319         | Leatherette Journal                                | 1                   |
| Somewhere in Hungary              | Hungary              | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Somewhere in Hungary              | Hungary              | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in India                | India                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Indonesia            | Indonesia            | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Iraq                 | Iraq                 | GGOEGAAX0359                 | 12          | Android Men's  Zip Hoodie                          | 1                   |
| Somewhere in Ireland              | Ireland              | GGOEADHH073999               | 167         | Android 17oz Stainless Steel Sport Bottle          | 1                   |
| Somewhere in Israel               | Israel               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Italy                | Italy                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Jamaica              | Jamaica              | GGOEGAAX0358                 | 3           | Men's  Zip Hoodie                                  | 1                   |
| Somewhere in Japan                | Japan                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Jersey               | Jersey               | GGOEGAAX0313                 | 11          | Tri-blend Hoodie Grey                              | 1                   |
| Somewhere in Jordan               | Jordan               | GGOEGCMB020932               | 5           | Suitcase Organizer Cubes                           | 1                   |
| Somewhere in Kazakhstan           | Kazakhstan           | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Somewhere in Kenya                | Kenya                | GGOEGAAX0597                 | 24          | Men's Quilted Insulated Vest Battleship Grey       | 1                   |
| Somewhere in Kosovo               | Kosovo               | GGOEGAAX0338                 | 1           | Men's Vintage Badge Tee Black                      | 1                   |
| Somewhere in Kuwait               | Kuwait               | GGOEGDHC074099               | 334         | 17oz Stainless Steel Sport Bottle                  | 1                   |
| Somewhere in Kyrgyzstan           | Kyrgyzstan           | GGOEGESC014099               | 16          | Rocket Flashlight                                  | 1                   |
| Somewhere in Laos                 | Laos                 | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Somewhere in Latvia               | Latvia               | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Lebanon              | Lebanon              | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Lithuania            | Lithuania            | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Somewhere in Luxembourg           | Luxembourg           | GGOEGAAX0659                 | 12          | Toddler Raglan Shirt Blue Heather/Navy             | 1                   |
| Somewhere in Macau                | Macau                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Macedonia (FYROM)    | Macedonia (FYROM)    | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Malaysia             | Malaysia             | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Maldives             | Maldives             | GGOEGAAX0569                 | 1           | Men's Performance Full Zip Jacket Black            | 1                   |
| Somewhere in Mali                 | Mali                 | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Somewhere in Malta                | Malta                | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Martinique           | Martinique           | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Somewhere in Mauritius            | Mauritius            | GGOEGETB023799               | 56          | Power Bank                                         | 1                   |
| Somewhere in Mexico               | Mexico               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Moldova              | Moldova              | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Somewhere in Montenegro           | Montenegro           | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Somewhere in Morocco              | Morocco              | GGOENEBJ079499               | 94          | Learning Thermostat 3rd Gen-USA - Stainless Steel  | 1                   |
| Somewhere in Myanmar (Burma)      | Myanmar (Burma)      | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Nepal                | Nepal                | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Netherlands          | Netherlands          | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in New Zealand          | New Zealand          | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Nicaragua            | Nicaragua            | GGOEGFQB013799               | 5           | Compact Selfie Stick                               | 1                   |
| Somewhere in Nigeria              | Nigeria              | GGOEGFKA022299               | 53          | Keyboard DOT Sticker                               | 1                   |
| Somewhere in Norway               | Norway               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Oman                 | Oman                 | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Somewhere in Pakistan             | Pakistan             | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Palestine            | Palestine            | GGOEGBRB013899               | 3           | Laptop Backpack                                    | 1                   |
| Somewhere in Panama               | Panama               | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Papua New Guinea     | Papua New Guinea     | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Somewhere in Peru                 | Peru                 | GGOEGDHJ082599               | 456         | 17 oz Double Wall Stainless Steel Insulated Bottle | 1                   |
| Somewhere in Philippines          | Philippines          | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Poland               | Poland               | GGOEAOCH077899               | 665         | Android Spiral Journal with Pen                    | 1                   |
| Somewhere in Portugal             | Portugal             | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Puerto Rico          | Puerto Rico          | GGOEYFKQ020699               | 30          | Custom Decals                                      | 1                   |
| Somewhere in Réunion              | Réunion              | GGOEGFKQ020799               | 34          | Doodle Decal                                       | 1                   |
| Somewhere in Romania              | Romania              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Russia               | Russia               | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in San Marino           | San Marino           | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Saudi Arabia         | Saudi Arabia         | GGOEGDHC074099               | 334         | 17oz Stainless Steel Sport Bottle                  | 1                   |
| Somewhere in Serbia               | Serbia               | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Singapore            | Singapore            | GGOEGDHC074099               | 334         | 17oz Stainless Steel Sport Bottle                  | 1                   |
| Somewhere in Sint Maarten         | Sint Maarten         | GGOEGOAR013099               | 2           | Stylus Pen w/ LED Light                            | 1                   |
| Somewhere in Slovakia             | Slovakia             | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Slovenia             | Slovenia             | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Somalia              | Somalia              | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Somewhere in South Africa         | South Africa         | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Somewhere in South Korea          | South Korea          | GGOEGHPJ080310               | 189         | Blackout Cap                                       | 1                   |
| Somewhere in Spain                | Spain                | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Sri Lanka            | Sri Lanka            | GGOEGAAX0317                 | 50          | Men's Short Sleeve Hero Tee White                  | 1                   |
| Somewhere in Sudan                | Sudan                | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Somewhere in Sweden               | Sweden               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Switzerland          | Switzerland          | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Taiwan               | Taiwan               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Tanzania             | Tanzania             | GGOEYHPB072210               | 9           | Twill Cap                                          | 1                   |
| Somewhere in Thailand             | Thailand             | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Trinidad & Tobago    | Trinidad & Tobago    | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Somewhere in Tunisia              | Tunisia              | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Turkey               | Turkey               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Uganda               | Uganda               | GGOEGFKQ020399               | 13          | Laptop and Cell Phone Stickers                     | 1                   |
| Somewhere in Ukraine              | Ukraine              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in United Arab Emirates | United Arab Emirates | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Somewhere in United Kingdom       | United Kingdom       | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in United States        | United States        | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Uruguay              | Uruguay              | GGOEYDHJ056099               | 50          | 22 oz  Bottle Infuser                              | 1                   |
| Somewhere in Venezuela            | Venezuela            | GGOEGOCT019199               | 316         | Red Spiral  Notebook                               | 1                   |
| Somewhere in Vietnam              | Vietnam              | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Somewhere in Zimbabwe             | Zimbabwe             | GGOEGOFH020299               | 12          | Screen Cleaning Cloth                              | 1                   |
| South San Francisco               | United States        | GGOEAFKQ020599               | 11          | Android Sticker Sheet Ultra Removable              | 1                   |
| South San Francisco               | United States        | GGOEGBRJ037399               | 11          | Rucksack                                           | 1                   |
| St. Louis                         | United States        | GGOEGKWR060810               | 7           | Bib Red                                            | 1                   |
| Stanford                          | United States        | GGOEGAAX0358                 | 3           | Men's  Zip Hoodie                                  | 1                   |
| Stockholm                         | Sweden               | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Stuttgart                         | Germany              | GGOEGEVB071799               | 214         | Pocket Bluetooth Speaker                           | 1                   |
| Sunnyvale                         | United States        | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |
| Sydney                            | Australia            | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| Taguig                            | Philippines          | GGOEYOCR077399               | 935         | RFID Journal                                       | 1                   |
| Tel Aviv-Yafo                     | Israel               | GGOEGOCT019199               | 316         | Red Spiral  Notebook                               | 1                   |
| Tempe                             | United States        | GGOEAKDH019899               | 16          | Windup Android                                     | 1                   |
| The Dalles                        | United States        | GGOEGFKQ020399               | 13          | Laptop and Cell Phone Stickers                     | 1                   |
| Thessaloniki                      | Greece               | GGOEGOCT019199               | 316         | Red Spiral  Notebook                               | 1                   |
| Timisoara                         | Romania              | GGOEGOFH020299               | 12          | Screen Cleaning Cloth                              | 1                   |
| Toronto                           | Canada               | GGOEYOLR018699               | 1148        | Leatherette Notebook Combo                         | 1                   |
| University Park                   | United States        | GGOEGBRA037499               | 5           | Waterproof Backpack                                | 1                   |
| Unknown                           | Unknown              | GGOEGEVR014999               | 183         | Water Resistant Bluetooth Speaker                  | 1                   |
| Vancouver                         | Canada               | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Vienna                            | Austria              | GGOEGAAX0104                 | 15          | Men's 100% Cotton Short Sleeve Hero Tee White      | 1                   |
| Villeneuve-d'Ascq                 | France               | GGOEGHPB003410               | 1           | Snapback Hat Black                                 | 1                   |
| Vilnius                           | Lithuania            | GGOEGAAX0107                 | 1           | Men's 100% Cotton Short Sleeve Hero Tee Red        | 1                   |
| Vladivostok                       | Russia               | GGOEGBJL013999               | 62          | Canvas Tote Natural/Navy                           | 1                   |
| Warsaw                            | Poland               | GGOEGAAX0098                 | 104         | 7 Dog Frisbee                                      | 1                   |
| Washington                        | United States        | GGOEGEVB071799               | 214         | Pocket Bluetooth Speaker                           | 1                   |
| Wellesley                         | United States        | GGOEGOBG023599               | 3           | Colored Pencil Set                                 | 1                   |
| Westville                         | South Africa         | GGOEGBMJ013399               | 70          | Sport Bag                                          | 1                   |
| Wrexham                           | United Kingdom       | GGOEGFKQ020399               | 13          | Laptop and Cell Phone Stickers                     | 1                   |
| Yokohama                          | Japan                | GGOEGAAX0362                 | 25          | Men's Pullover Hoodie Grey                         | 1                   |
| Yokohama                          | United States        | GGOEGAAX0362                 | 25          | Men's Pullover Hoodie Grey                         | 1                   |
| Zagreb                            | Croatia              | GGOEGAAX0568                 | 3           | Men's Watershed Full Zip Hoodie Grey               | 1                   |
| Zhongli District                  | Taiwan               | GGOEYOCR077799               | 85          | Hard Cover Journal                                 | 1                   |
| Zurich                            | Switzerland          | GGOEGDHQ014899               | 499         | 20 oz Stainless Steel Insulated Tumbler            | 1                   |



# Question 5: Can we summarize the impact of revenue generated from each city/country?

**Solution Notes**:  I generally think it is very difficult to summarize the **impact** of revenue generated from each city/country.  This is because the impact of revenue is typically measured by such data points as:
- Investment back into the country from which the revenue was generated (e.g. charitable support for social programs, infrastructure, etc.) to make a positive impact on the countries in which a company operates.
- Investment back into the company's own resources such as staffing, IT equipment, improvement of ecommerce site, etc.

We do not have any data representing any of those impacts.

However, we do have sentimentscore and sentimentmagnitude in the product table.  Ostensibly, this is some form of sentimentscore per product, and sentimentmagnitude.  Going out on a limb completely, I decided to try to use the sentimentscore and sentimentmagnitude for each product, as a very rough measure of "impact".  That would be, "impact" on the end customer, whether positive or negative. I decided to try to evaluate both the total revenue per country (I collapsed all cities into a singular country) **and** an "impact" score, which would be a measure of the following **PER PRODUCT SKU**:  number of units of the product * sentimentscore of the product * sentimentmagnitude of the prouct.

I then ranked the revenue and the impact per country to see if there was a correlation.  That is the result set below.


### SQL Queries:
```SQL
WITH city_country_sordered_pordered AS (
	SELECT
		city,
		country,
		alls.productsku AS sessions_sku,
		s.productsku AS sales_sku,
		s.total_ordered AS sales_ordered,
		p.sku AS products_sku,
		p.orderedquantity as products_ordered,
		alls.productprice,
		p.sentimentscore,
		p.sentimentmagnitude
    FROM allsessions_clean alls
    LEFT JOIN salesbysku s
    ON alls.productsku = s.productsku
    LEFT JOIN products_clean p
    ON alls.productsku = p.sku
)
,
city_country_sku_clean_ordered AS (
	SELECT
		city,
		country,
		sessions_sku,
		CASE
			WHEN sales_ordered IS NOT NULL AND sales_ordered > 0 THEN sales_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NOT NULL THEN products_ordered::int
			WHEN sales_ordered IS NULL AND products_ordered IS NULL THEN 0::int
			ELSE 0::int
		END AS ordered,
		productprice,
		sentimentscore,
		sentimentmagnitude
	FROM city_country_sordered_pordered
)
,
clean_city_country_sku_clean_ordered AS (
	SELECT
		CASE
			WHEN (city IS NULL OR city = '(not set)' OR city = 'not available in demo dataset') AND (country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset') THEN 'Unknown'
			WHEN city = '(not set)' OR city = 'not available in demo dataset' THEN 'Somewhere in ' || country
			ELSE city
		END AS city,
		CASE
			WHEN country IS NULL OR country = '(not set)' OR country = 'not available in demo dataset' THEN 'Unknown'
			ELSE country
		END AS country,
		sessions_sku,
		ordered,
		productprice,
		sentimentscore,
		sentimentmagnitude
	FROM city_country_sku_clean_ordered
)
,
clean_city_country_sku_clean_ordered_clean_sentiments AS (
	SELECT
		city,
		country,
		sessions_sku,
		ordered,
		productprice,
		CASE  -- cleaning sentimentscore "on the fly" using interpolation
			WHEN sentimentscore IS NULL THEN (SELECT AVG(sentimentscore)::real FROM clean_city_country_sku_clean_ordered)
			ELSE sentimentscore
		END AS sentimentscore,
		CASE  -- cleaning sentimentmagnitude "on the fly" using interpolation
			WHEN sentimentmagnitude IS NULL THEN (SELECT AVG(sentimentmagnitude)::real FROM clean_city_country_sku_clean_ordered)
			ELSE sentimentmagnitude
		END AS sentimentmagnitude
	FROM clean_city_country_sku_clean_ordered
)
,
country_sku_revenue_impact AS (
	SELECT
		country,
		sessions_sku,
		ordered,
		productprice,
		(ordered*productprice) AS revenue,
		sentimentscore,
		sentimentmagnitude,
		(ordered::real*productprice::real*sentimentscore::real*sentimentmagnitude::real) AS impact
	FROM clean_city_country_sku_clean_ordered_clean_sentiments
)
,
country_revenue_impact AS (
	SELECT DISTINCT
	country,
	SUM(revenue) OVER (PARTITION BY country) AS total_revenue,
	AVG(impact) OVER (PARTITION BY country) AS country_impact
FROM country_sku_revenue_impact
WHERE ordered > 0 AND productprice > 0
ORDER BY country_impact DESC, total_revenue DESC, country ASC
)

SELECT
	country,
	total_revenue,
	country_impact,
	RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank,
	RANK() OVER (ORDER BY country_impact DESC) AS impact_rank
FROM country_revenue_impact
ORDER BY impact_rank, revenue_rank
```


## Answer:

There does not appear to be a clear correlation between the revenue generated from a country and the impact.  That is, we do not see a clear and obvious pattern where the highest revenue causes the highest impact or the highest revenue causes the lowest impact.  This was a "long-shot" at best; I did not think this question could be easily answered, but wanted to do some SQL analysis to see if any kind of pattern showed up that could be investigated more thoroughly.  I do not think there is a pattern.  One of the problems is the assumptions (possibly faulty) made about what the sentimentscore and sentimentmagnitude values are, and the assumption made about calculating an "impact" score.  Not having a data dictionary or "the business" to return to, to ask further probing questions about the source of the data, has been problematic all along, and these results are no exception.

The result set is below:
| country              | total_revenue | country_impact | revenue_rank | impact_rank |
|----------------------|---------------|----------------|--------------|-------------|
| Honduras             | 22288         | 13372.8        | 46           | 1           |
| Kuwait               | 10531.77      | 1440.87        | 63           | 2           |
| Saudi Arabia         | 15385.29      | 1159.539       | 57           | 3           |
| Barbados             | 1299.75       | 974.8125       | 87           | 4           |
| Kenya                | 1833.74       | 865.0742       | 83           | 5           |
| Serbia               | 25741.91      | 695.0907       | 42           | 6           |
| El Salvador          | 3253.18       | 685.1708       | 77           | 7           |
| Morocco              | 23807.14      | 598.5221       | 43           | 8           |
| Estonia              | 4151.08       | 515.1538       | 75           | 9           |
| Laos                 | 2548.3        | 509.66         | 79           | 10          |
| Oman                 | 1274.15       | 509.66         | 88           | 10          |
| Latvia               | 22128.3       | 490.8014       | 47           | 12          |
| Sweden               | 90270.03      | 475.6708       | 18           | 13          |
| Italy                | 184757        | 451.4205       | 7            | 14          |
| Lithuania            | 10931.04      | 428.6437       | 61           | 15          |
| Martinique           | 6515.15       | 402.6669       | 71           | 16          |
| Belgium              | 92716.43      | 372.0256       | 16           | 17          |
| United States        | 8696471       | 371.6561       | 1            | 18          |
| Ethiopia             | 9298.67       | 335.0752       | 65           | 19          |
| United Arab Emirates | 18289.11      | 328.4263       | 53           | 20          |
| Jersey               | 439.89        | 307.923        | 101          | 21          |
| Venezuela            | 12497.21      | 306.7216       | 60           | 22          |
| South Korea          | 22961.08      | 304.9777       | 45           | 23          |
| Luxembourg           | 287.88        | 299.3952       | 106          | 24          |
| Hong Kong            | 69616.15      | 292.8299       | 22           | 25          |
| Malaysia             | 43747.87      | 281.9486       | 33           | 26          |
| United Kingdom       | 848744.7      | 279.4334       | 2            | 27          |
| Greece               | 35110.57      | 270.2248       | 38           | 28          |
| Trinidad & Tobago    | 1373.06       | 263.7319       | 85           | 29          |
| Poland               | 47821.56      | 257.1826       | 30           | 30          |
| Papua New Guinea     | 1333.85       | 255.7255       | 86           | 31          |
| Colombia             | 48666.1       | 249.6579       | 29           | 32          |
| Unknown              | 10646.54      | 249.0779       | 62           | 33          |
| Czechia              | 119735.1      | 241.3573       | 13           | 34          |
| China                | 6392.33       | 238.3568       | 72           | 35          |
| Ireland              | 87573.67      | 237.8598       | 19           | 36          |
| Taiwan               | 147688.6      | 234.6803       | 11           | 37          |
| Canada               | 538229.8      | 226.4229       | 4            | 38          |
| Japan                | 164676.9      | 225.3003       | 8            | 39          |
| Peru                 | 16146.14      | 204.9851       | 55           | 40          |
| Singapore            | 39744.9       | 200.3883       | 35           | 41          |
| Germany              | 315217.9      | 195.7178       | 5            | 42          |
| Spain                | 159663.7      | 194.5454       | 9            | 43          |
| Bulgaria             | 14128.65      | 190.2097       | 58           | 44          |
| Mexico               | 148290.1      | 190.1599       | 10           | 45          |
| Ukraine              | 60042.09      | 187.7468       | 25           | 46          |
| Turkey               | 60309.83      | 181.5214       | 24           | 47          |
| India                | 714429.7      | 173.8249       | 3            | 48          |
| Pakistan             | 93934.39      | 172.8083       | 15           | 49          |
| Israel               | 49427.89      | 166.8322       | 28           | 50          |
| Brazil               | 57044.73      | 162.5204       | 26           | 51          |
| Denmark              | 23159.03      | 160.9359       | 44           | 52          |
| Algeria              | 386.79        | 158.5995       | 103          | 53          |
| Thailand             | 39610.65      | 156.7416       | 36           | 54          |
| Guatemala            | 5695.51       | 145.8455       | 73           | 55          |
| Slovenia             | 9601.14       | 145.8047       | 64           | 56          |
| Russia               | 78747.92      | 141.688        | 20           | 57          |
| Netherlands          | 114534.9      | 134.9957       | 14           | 58          |
| Argentina            | 12552.73      | 134.5086       | 59           | 59          |
| Norway               | 31479.55      | 132.602        | 40           | 60          |
| Nepal                | 1532.12       | 132.112        | 84           | 61          |
| Philippines          | 91778.28      | 130.451        | 17           | 62          |
| Hungary              | 5422.37       | 128.4838       | 74           | 63          |
| France               | 136342.2      | 113.7337       | 12           | 64          |
| Australia            | 253846.7      | 111.559        | 6            | 65          |
| South Africa         | 15743.92      | 111.2183       | 56           | 66          |
| New Zealand          | 76047.96      | 111.1007       | 21           | 67          |
| Vietnam              | 33371.54      | 109.2066       | 39           | 68          |
| Slovakia             | 44327.78      | 107.2586       | 32           | 69          |
| Belarus              | 1069.53       | 106.53         | 90           | 70          |
| Bosnia & Herzegovina | 8058.5        | 84.3228        | 70           | 71          |
| Kazakhstan           | 1064.07       | 78.01075       | 91           | 72          |
| Sri Lanka            | 2030.76       | 75.11225       | 81           | 73          |
| Macau                | 8074.48       | 72.0622        | 69           | 74          |
| Romania              | 17136.22      | 71.22817       | 54           | 75          |
| Iraq                 | 690.87        | 67.59045       | 95           | 76          |
| Somalia              | 849.5         | 59.465         | 94           | 77          |
| Georgia              | 8278.61       | 57.16948       | 68           | 78          |
| Switzerland          | 37233.45      | 55.39539       | 37           | 79          |
| Nigeria              | 1148.11       | 51.795         | 89           | 80          |
| Ecuador              | 21078.91      | 48.08908       | 49           | 81          |
| Panama               | 2011.74       | 47.8502        | 82           | 82          |
| Portugal             | 43187.86      | 47.53625       | 34           | 83          |
| Bangladesh           | 47488.26      | 46.24697       | 31           | 84          |
| Ghana                | 254.85        | 45.873         | 107          | 85          |
| Indonesia            | 66027.55      | 45.46472       | 23           | 86          |
| Bolivia              | 441.74        | 43.8342        | 100          | 87          |
| Paraguay             | 220.87        | 39.7566        | 110          | 88          |
| Albania              | 249.5         | 37.425         | 108          | 89          |
| San Marino           | 249.5         | 37.425         | 108          | 89          |
| Austria              | 50377.26      | 37.11408       | 27           | 91          |
| Dominican Republic   | 8626.04       | 32.85501       | 67           | 92          |
| Finland              | 2342.81       | 30.87044       | 80           | 93          |
| Tunisia              | 348.41        | 27.6144        | 104          | 94          |
| Cambodia             | 632.65        | 27.45293       | 96           | 95          |
| Jordan               | 109.95        | 26.388         | 116          | 96          |
| Uruguay              | 1045          | 24.9096        | 92           | 97          |
| Myanmar (Burma)      | 606.29        | 24.368         | 97           | 98          |
| Réunion              | 161.36        | 23.769         | 113          | 99          |
| Egypt                | 9064.61       | 22.66566       | 66           | 100         |
| Kyrgyzstan           | 79.84         | 22.3552        | 120          | 101         |
| Cyprus               | 147.93        | 22.3092        | 114          | 102         |
| Bahrain              | 390.77        | 20.8977        | 102          | 103         |
| Croatia              | 21630.79      | 18.00931       | 48           | 104         |
| Tanzania             | 98.91         | 17.8038        | 117          | 105         |
| Puerto Rico          | 914.26        | 17.39          | 93           | 106         |
| Sint Maarten         | 11            | 13.86          | 132          | 107         |
| Zimbabwe             | 23.88         | 13.3728        | 129          | 108         |
| Haiti                | 18.99         | 13.293         | 130          | 109         |
| Kosovo               | 16.99         | 12.7425        | 131          | 110         |
| Uganda               | 38.87         | 10.8836        | 128          | 111         |
| Armenia              | 63.84         | 10.2144        | 122          | 112         |
| Macedonia (FYROM)    | 506.47        | 6.811865       | 98           | 113         |
| Palestine            | 299.97        | 5.9994         | 105          | 114         |
| Nicaragua            | 44.95         | 2.697          | 127          | 115         |
| Moldova              | 59.7          | 1.791          | 124          | 116         |
| Mali                 | 59.7          | 1.791          | 124          | 116         |
| Montenegro           | 59.7          | 1.791          | 124          | 116         |
| Sudan                | 18690.65      | 0              | 50           | 119         |
| Lebanon              | 18690.65      | 0              | 50           | 119         |
| Malta                | 18690.65      | 0              | 50           | 119         |
| Botswana             | 499.95        | 0              | 99           | 119         |
| Qatar                | 67.96         | -2.7184        | 121          | 123         |
| Bahamas              | 92.95         | -3.8486        | 119          | 124         |
| Jamaica              | 167.97        | -6.7188        | 112          | 125         |
| Côte d’Ivoire        | 179.38        | -12.8013       | 111          | 126         |
| Maldives             | 119.99        | -14.3988       | 115          | 127         |
| Brunei               | 93.96         | -16.9154       | 118          | 128         |
| Chile                | 30893.62      | -17.7556       | 41           | 129         |
| Gibraltar            | 59.98         | -41.986        | 123          | 130         |
| Costa Rica           | 3459.21       | -48.1913       | 76           | 131         |
| Mauritius            | 3249.38       | -156.772       | 78           | 132         |





