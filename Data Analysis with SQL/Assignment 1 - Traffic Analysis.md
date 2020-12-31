# Assignment 1 - Traffic Analysis

First let's set up our database on which we will work during this assignment.

List out all the database availables.
~~~~sql
SHOW DATABASES;
~~~~

Select the database for this assignment.
~~~~sql
USE mavenfuzzyfactory;
~~~~

List out tables in the **mavenfuzzyfactory** database.
~~~~sql
SHOW TABLES;
~~~~

## *Test Skill* - Finding Traffic Sources

### Question 1 : Where the bulk of website sesssions are coming from, through yesterday? Breakdown results by **UTM Source**, **Campaign** and **Referring domain**.

### Solution: 
For this we will use *website_sessions* table, lets see first 5 rows of this table.
~~~~sql
SELECT * FROM website_sessions LIMIT 5
~~~~

We will use *utm_source*, *utm_campaign* and *http_referer* fields to find the traffic volume.
~~~~sql
SELECT 
	utm_source, utm_campaign, http_referer, 
	COUNT(DISTINCT(website_session_id)) as sessions
FROM website_sessions
WHERE created_at < '2020-12-31'
GROUP BY 
	utm_source,
	utm_campaign,
	http_referer
ORDER BY sessions DESC;
~~~~

**gsearch nonbrand** has highest sessions, drill deeper into **gsearch nonbrand** campaign traffic to explore potential optimization opportunities. 

**NOTE**: Date in where condition will be in brackets and GROUP BY will have all three columns. \n 
If GROUP BY is only used with *utm_source* then statement will return first row for each utm_source and total count as sessions.\n
Which is not a proper breakdown nor does it provides complete picture.


## *Test Skill* - Traffic Conversion Rate

### Question 2: Calculate session to order conversion rate for traffic sources. Minimum required CVR is 4%, if less than that then need to reduce bids.

Lets first look into the **orders** table and see on which key we can do join to find orders count for each session.
~~~~sql
 SELECT * FROM orders LIMIT 5;
~~~~

*website_session_id* is present in **orders** table so we can do join on it. Lets find out the session to order CVR for **gsearch nonbrand**.
~~~~sql
SELECT
	COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
	COUNT(DISTINCT(orders.order_id)) AS orders,
	(COUNT(DISTINCT(orders.order_id)) / COUNT(DISTINCT(website_sessions.website_session_id))) AS session_to_order_cvr
FROM website_sessions
LEFT JOIN orders ON  
	website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at < '2020-12-31'
	AND website_sessions.utm_source = 'gsearch'
	AND website_sessions.utm_campaign = 'nonbrand'
;
~~~~
From the results it is clear that **gsearch nonbrand** has 6.66% session to order conversion rate.

Let's see the all non-null sources drilled down to campaign type by source to order conversion rate.

~~~~sql
SELECT
	website_sessions.utm_source AS utm_source,
	website_sessions.utm_campaign AS utm_campaign,
	website_sessions.http_referer AS http_referer,
	COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
	COUNT(DISTINCT(orders.order_id)) AS orders,
	(COUNT(DISTINCT(orders.order_id)) / COUNT(DISTINCT(website_sessions.website_session_id))) AS session_to_order_cvr
FROM website_sessions
LEFT JOIN orders ON  
	website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.utm_source IS NOT NULL
GROUP BY 
	utm_source,
	utm_campaign,
	http_referer
ORDER BY orders_per_session DESC
~~~~

As per results, **socialbook pilot** has CVR of 1.08%, rest have more than 4%. Bids should be reduced for **socialbook pilot**











