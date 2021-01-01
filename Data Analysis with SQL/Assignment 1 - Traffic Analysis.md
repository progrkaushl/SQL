# Assignment 1 - Traffic Analysis

First let's set up our database on which we will work during this assignment.

List out all the database availables.
~~~~sql
SHOW DATABASES;
~~~~

*Output :-*


| Database           |
|--------------------|
| employees          |
| information_schema |
| mavenfuzzyfactory  |
| mysql              |
| performance_schema |
| sakila             |
| sql_hr             |
| sql_inventory      |
| sql_invoicing      |
| sql_store          |
| sys                |
| world              |


Select the database for this assignment.
~~~~sql
USE mavenfuzzyfactory;
~~~~

List out tables in the **mavenfuzzyfactory** database.
~~~~sql
SHOW TABLES;
~~~~

*Output :-*

| Tables_in_mavenfuzzyfactory |
|-----------------------------|
| order_item_refunds          |
| order_items                 |
| orders                      |
| products                    |
| website_pageviews           |
| website_sessions            |


## *Test Skill* - Finding Traffic Sources

### Task 1 : Where the bulk of website sesssions are coming from, before 2012-04-12? Breakdown results by **UTM Source**, **Campaign** and **Referring domain**.

### Solution: 
For this we will use *website_sessions* table, lets see first 5 rows of this table.
~~~~sql
SELECT * FROM website_sessions LIMIT 5
~~~~

*Output :-*


| website_session_id | created_at          | user_id | is_repeat_session | utm_source | utm_campaign | utm_content | device_type | http_referer            |
|--------------------|---------------------|---------|-------------------|------------|--------------|-------------|-------------|-------------------------|
|                  1 | 2012-03-19 08:04:16 |       1 |                 0 | gsearch    | nonbrand     | g_ad_1      | mobile      | https://www.gsearch.com |
|                  2 | 2012-03-19 08:16:49 |       2 |                 0 | gsearch    | nonbrand     | g_ad_1      | desktop     | https://www.gsearch.com |
|                  3 | 2012-03-19 08:26:55 |       3 |                 0 | gsearch    | nonbrand     | g_ad_1      | desktop     | https://www.gsearch.com |
|                  4 | 2012-03-19 08:37:33 |       4 |                 0 | gsearch    | nonbrand     | g_ad_1      | desktop     | https://www.gsearch.com |
|                  5 | 2012-03-19 09:00:55 |       5 |                 0 | gsearch    | nonbrand     | g_ad_1      | mobile      | https://www.gsearch.com ||--------------------|---------------------|---------|-------------------|------------|--------------|-------------|-------------|-------------------------|

We will use *utm_source*, *utm_campaign* and *http_referer* fields to find the traffic volume.
~~~~sql
SELECT 
	utm_source, utm_campaign, http_referer, 
	COUNT(DISTINCT(website_session_id)) as sessions
FROM website_sessions
WHERE created_at < '2012-04-12'
GROUP BY 
	utm_source,
	utm_campaign,
	http_referer
ORDER BY sessions DESC;
~~~~

*Output :-*

| utm_source | utm_campaign | http_referer            | sessions |
|------------|--------------|-------------------------|----------|
| gsearch    | nonbrand     | https://www.gsearch.com |     3613 |
| NULL       | NULL         | NULL                    |       28 |
| NULL       | NULL         | https://www.gsearch.com |       27 |
| gsearch    | brand        | https://www.gsearch.com |       26 |
| NULL       | NULL         | https://www.bsearch.com |        7 |
| bsearch    | brand        | https://www.bsearch.com |        7 |

**gsearch nonbrand** has highest sessions, drill deeper into **gsearch nonbrand** campaign traffic to explore potential optimization opportunities. 

**NOTE**: Date in where condition will be in brackets and GROUP BY will have all three columns. \n 
If GROUP BY is only used with *utm_source* then statement will return first row for each utm_source and total count as sessions.\n
Which is not a proper breakdown nor does it provides complete picture.


## *Test Skill* - Traffic Conversion Rate

### Task 2: Calculate session to order conversion rate for traffic sources. Minimum required CVR is 4%, if less than that then need to reduce bids.

### Solution: 
Lets first look into the **orders** table and see on which key we can do join to find orders count for each session.
~~~~sql
SELECT * FROM orders LIMIT 5;
~~~~

*Output :-*

| order_id | created_at          | website_session_id | user_id | primary_product_id | items_purchased | price_usd | cogs_usd |
|----------|---------------------|--------------------|---------|--------------------|-----------------|-----------|----------|
|        1 | 2012-03-19 10:42:46 |                 20 |      20 |                  1 |               1 |     49.99 |    19.49 |
|        2 | 2012-03-19 19:27:37 |                104 |     104 |                  1 |               1 |     49.99 |    19.49 |
|        3 | 2012-03-20 06:44:45 |                147 |     147 |                  1 |               1 |     49.99 |    19.49 |
|        4 | 2012-03-20 09:41:45 |                160 |     160 |                  1 |               1 |     49.99 |    19.49 |
|        5 | 2012-03-20 11:28:15 |                177 |     177 |                  1 |               1 |     49.99 |    19.49 |

*website_session_id* is present in **orders** table so we can do join on it. Lets find out the session to order CVR for **gsearch nonbrand**.
~~~~sql
SELECT
	COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
	COUNT(DISTINCT(orders.order_id)) AS orders,
	(COUNT(DISTINCT(orders.order_id)) / COUNT(DISTINCT(website_sessions.website_session_id))) AS session_to_order_cvr
FROM website_sessions
LEFT JOIN orders ON  
	website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-04-12'
	AND website_sessions.utm_source = 'gsearch'
	AND website_sessions.utm_campaign = 'nonbrand'
;
~~~~

*Output :-*

| sessions | orders | session_to_order_cvr |
|----------|--------|----------------------|
|     3613 |    107 |               0.0296 |

From the results it is clear that **gsearch nonbrand** has 2.96% session to order conversion rate.

Let's see the all non-null sources drilled down to campaign type by source to order conversion rate.

~~~~sql
SELECT
	website_sessions.utm_source AS utm_source,
	website_sessions.utm_campaign AS utm_campaign,
	website_sessions.http_referer AS http_referer,
	COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
	COUNT(DISTINCT(orders.order_id)) AS orders,
	(COUNT(DISTINCT(orders.order_id)) / COUNT(DISTINCT(website_sessions.website_session_id))) AS orders_per_session
FROM website_sessions
LEFT JOIN orders ON  
	website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2012-04-12' 
	AND website_sessions.utm_source IS NOT NULL
GROUP BY 
	utm_source,
	utm_campaign,
	http_referer
ORDER BY orders_per_session DESC;
~~~~

*Output :-* 

| utm_source | utm_campaign | http_referer            | sessions | orders | orders_per_session |
|------------|--------------|-------------------------|----------|--------|--------------------|
| gsearch    | brand        | https://www.gsearch.com |       26 |      2 |             0.0769 |
| gsearch    | nonbrand     | https://www.gsearch.com |     3613 |    107 |             0.0296 |
| bsearch    | brand        | https://www.bsearch.com |        7 |      0 |             0.0000 |

As per results, **bsearch brand** has CVR of 0% which is lowest.

**NEXT STEP**
- Monitor the impact of bid reduction
- Analyze the performance trending by device type in order to find bidding strategy

![img](https://i.imgur.com/TjGWKN4.png)


## *Test Skill* - Traffic Source Trending

### Task 3 - Get the data for *gsearch nonbrand* trended session volume by week before 2012-05-12.

### Solution:

- Filter the data by *created_at*, *utm_source*, *utm_campaign*
- Group by the data using **YEARWEEK()** on *created_at*, which is equivalent of using 
~~~~sql
GROUP BY 
	YEAR(created_at),
	WEEK(created_at)
~~~~
- Extract the date from *created_at* using **DATE()** and return minimum / lowest date for each row by wrapping it in **MIN()**
- Count the distinct session id

After applying all the steps we willl get this code:
~~~sql
SELECT
	MIN(DATE(created_at)) AS week_start_date,
	COUNT(DISTINCT (website_session_id)) AS sessions
FROM website_sessions
WHERE created_at < '2012-05-12'
	AND utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand'
GROUP BY 
	YEARWEEK(created_at)
ORDER BY week_start_date;
~~~

*Output :-*

| week_start_date | sessions |
|-----------------|----------|
| 2012-03-19      |      896 |
| 2012-03-25      |      956 |
| 2012-04-01      |     1152 |
| 2012-04-08      |      983 |
| 2012-04-15      |      621 |
| 2012-04-22      |      594 |
| 2012-04-29      |      681 |
| 2012-05-06      |      651 |

From the data it is evident that after bidding down **gsearch nonbrand**, session volume has decreased.

## *Test Skill* - Traffic Source Bid Optimization

### TASK 4 : Find sessions to order conversion rate by device type.

### Solution:

- Filter the data by *created_at*, *utm_source*, *utm_campaign*
- Do left join of table **website_sessions** on **orders**
- Distinct count session id and order id
- Group by device type

~~~~sql
SELECT 
	device_type,
	COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
	COUNT(DISTINCT(orders.order_id)) AS orders,
	(COUNT(DISTINCT(orders.order_id)) / COUNT(DISTINCT(website_sessions.website_session_id))) AS session_to_order_cvr
FROM website_sessions
LEFT JOIN orders
	ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-05-12'
	AND website_sessions.utm_source = 'gsearch'
	AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY 1;
~~~~


*Output :-*

| device_type | sessions | orders | session_to_order_cvr |
|-------------|----------|--------|----------------------|
| desktop     |     3986 |    148 |               0.0371 |
| mobile      |     2548 |     24 |               0.0094 |

From analysis it is clear that desktop has higher CVR than mobile.


## *Test Skill* - Traffic Source Segment Trending

### Task 5 - Get the *gsearch nonbrand* weekly sessions data by device type, before 2012-06-09 and use 2012-04-15 as baseline.

### Solution : 

Use **CASE** method to count distinct records for each device type.

~~~~sql
SELECT 
	MIN(DATE(created_at)) AS week_start_date,
	COUNT(
		DISTINCT(
			CASE
				WHEN device_type = 'desktop'
				THEN website_session_id
				ELSE NULL
			END
		)
	) AS desktop_sessions,    
	COUNT(
		DISTINCT(
			CASE
				WHEN device_type = 'desktop'
				THEN website_session_id
				ELSE NULL
			END
		)
	) AS desktop_sessions

FROM website_sessions
WHERE created_at < '2012-06-09'
	AND created_at > '2012-04-15'
	AND utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand'
GROUP BY
	YEARWEEK(created_at)
ORDER BY 1;
~~~~

*Output :-*


| week_start_date | desktop_sessions | desktop_sessions |
|-----------------|------------------|------------------|
| 2012-04-15      |              383 |              383 |
| 2012-04-22      |              360 |              360 |
| 2012-04-29      |              425 |              425 |
| 2012-05-06      |              430 |              430 |
| 2012-05-13      |              403 |              403 |
| 2012-05-20      |              661 |              661 |
| 2012-05-27      |              585 |              585 |
| 2012-06-03      |              582 |              582 |


Clearly desktop sessions has increased after bidding down.
_____________
## End of Assignment - Traffic Analysis