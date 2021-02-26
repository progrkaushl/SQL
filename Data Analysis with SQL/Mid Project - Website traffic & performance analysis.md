# Mid Project 1 - Website Traffic & Performance Analysis

First let's set up our database on which we will work during this project.

List out all the database availables.
~~~~mysql
SHOW DATABASES;
~~~~

*Output :-*


| Database           |
|--------------------|
| information_schema |
| mavenfuzzyfactory  |
| mysql              |
| performance_schema |
| sys                |

Select the database for this assignment.
~~~~mysql
USE mavenfuzzyfactory;
~~~~

List out tables in the **mavenfuzzyfactory** database.
~~~~mysql
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


## Requirement 1

### Gsearch seems to be the biggest driver for business. Pull monthly trends for gsearch sessions and orders so that we can showcase growth there?

### Solution

~~~~mysql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	MONTH(website_sessions.created_at) AS mnth,
    MONTHNAME(website_sessions.created_at) AS mnth_name,
    COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
    COUNT(DISTINCT(orders.order_id)) AS orders
FROM website_sessions
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.utm_source = "gsearch" AND
website_sessions.created_at <= '2012-11-27'
GROUP BY 
	yr,	
	mnth
ORDER BY 
	mnth ASC
;
~~~~

*Output :-*

| yr   | mnth | mnth_name | sessions | orders |
|------|------|-----------|----------|--------|
| 2012 |    3 | March     |     1816 |     59 |
| 2012 |    4 | April     |     3558 |     93 |
| 2012 |    5 | May       |     3398 |     95 |
| 2012 |    6 | June      |     3616 |    120 |
| 2012 |    7 | July      |     3752 |    146 |
| 2012 |    8 | August    |     4895 |    185 |
| 2012 |    9 | September |     4517 |    185 |
| 2012 |   10 | October   |     5452 |    233 |
| 2012 |   11 | November  |     8045 |    338 |

## Requirement 2

### Check for similarity monthly trend for Gsearch, split out the nonbrand and brand campaign sperately.

### Solution

~~~~mysql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	MONTH(website_sessions.created_at) AS mnth,
    MONTHNAME(website_sessions.created_at) AS mnth_name,
    COUNT(DISTINCT(CASE WHEN utm_campaign = "nonbrand" THEN website_sessions.website_session_id ELSE NULL END)) AS nonbrand_sessions,
    COUNT(DISTINCT(CASE WHEN utm_campaign = "brand" THEN orders.order_id ELSE NULL END)) AS brand_sessions,
    COUNT(DISTINCT(CASE WHEN utm_campaign = "nonbrand" THEN website_sessions.website_session_id ELSE NULL END)) AS nonbrand_orders,
    COUNT(DISTINCT(CASE WHEN utm_campaign = "brand" THEN orders.order_id ELSE NULL END)) AS brand_orders
FROM website_sessions
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.utm_source = "gsearch" AND
website_sessions.created_at <= '2012-11-27'
GROUP BY 
	yr,	
	mnth
ORDER BY 
	mnth ASC
;
~~~~

*Output :-*

| yr   | mnth | mnth_name | nonbrand_sessions | brand_sessions | nonbrand_orders | brand_orders |
|------|------|-----------|-------------------|----------------|-----------------|--------------|
| 2012 |    3 | March     |              1808 |              0 |            1808 |            0 |
| 2012 |    4 | April     |              3497 |              6 |            3497 |            6 |
| 2012 |    5 | May       |              3283 |              6 |            3283 |            6 |
| 2012 |    6 | June      |              3477 |              6 |            3477 |            6 |
| 2012 |    7 | July      |              3599 |             10 |            3599 |           10 |
| 2012 |    8 | August    |              4694 |             10 |            4694 |           10 |
| 2012 |    9 | September |              4250 |             16 |            4250 |           16 |
| 2012 |   10 | October   |              5125 |             14 |            5125 |           14 |
| 2012 |   11 | November  |              7677 |             18 |            7677 |           18 |

## Requirement 3

### Dive into gsearch nonbrand and pull monthly sessions and orders split by device type.

### Solution

~~~~mysql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	MONTH(website_sessions.created_at) AS mnth,
    MONTHNAME(website_sessions.created_at) AS mnth_name,
    COUNT(DISTINCT(CASE WHEN website_sessions.device_type="mobile" THEN website_sessions.website_session_id ELSE NULL END)) AS mobile_sessions,
    COUNT(DISTINCT(CASE WHEN website_sessions.device_type="desktop" THEN website_sessions.website_session_id ELSE NULL END)) AS desktop_sessions,
    COUNT(DISTINCT(CASE WHEN website_sessions.device_type="mobile" THEN orders.order_id ELSE NULL END)) AS mobile_orders,
    COUNT(DISTINCT(CASE WHEN website_sessions.device_type="desktop" THEN orders.order_id ELSE NULL END)) AS desktop_orders
FROM website_sessions
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.utm_source = "gsearch" AND
website_sessions.utm_campaign = "nonbrand" AND
website_sessions.created_at <= '2012-11-27'
GROUP BY 
	yr,	
	mnth
ORDER BY 
	mnth ASC
;
~~~~

*Output :-*

| yr   | mnth | mnth_name | mobile_sessions | desktop_sessions | mobile_orders | desktop_orders |
|------|------|-----------|-----------------|------------------|---------------|----------------|
| 2012 |    3 | March     |             702 |             1106 |            10 |             49 |
| 2012 |    4 | April     |            1366 |             2131 |            11 |             76 |
| 2012 |    5 | May       |            1031 |             2252 |             8 |             81 |
| 2012 |    6 | June      |             776 |             2701 |             8 |            106 |
| 2012 |    7 | July      |             866 |             2733 |            14 |            122 |
| 2012 |    8 | August    |            1163 |             3531 |             9 |            166 |
| 2012 |    9 | September |            1061 |             3189 |            15 |            154 |
| 2012 |   10 | October   |            1253 |             3872 |            20 |            199 |
| 2012 |   11 | November  |            1844 |             5833 |            29 |            291 |

## Requirement 4

### Pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?

### Solution

~~~mysql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	MONTH(website_sessions.created_at) AS mnth,
    MONTHNAME(website_sessions.created_at) AS mnth_name,
    COUNT(DISTINCT(CASE WHEN website_sessions.utm_source="gsearch" THEN website_sessions.website_session_id ELSE NULL END)) AS gsearch_sessions,
    COUNT(DISTINCT(CASE WHEN website_sessions.utm_source="bsearch" THEN website_sessions.website_session_id ELSE NULL END)) AS bsearch_sessions,
    COUNT(DISTINCT(CASE WHEN website_sessions.utm_source="socialbook" THEN website_sessions.website_session_id ELSE NULL END)) AS socialbook_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN website_sessions.website_session_id ELSE NULL END) AS referral_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_sessions.website_session_id ELSE NULL END) AS organic_sessions
FROM website_sessions
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
WHERE 
website_sessions.created_at <= '2012-11-27'
GROUP BY 
	yr,	
	mnth
ORDER BY 
	mnth ASC
;
~~~~

*Output :-*

| yr   | mnth | mnth_name | gsearch_sessions | bsearch_sessions | socialbook_sessions | referral_sessions | organic_sessions |
|------|------|-----------|------------------|------------------|---------------------|-------------------|------------------|
| 2012 |    3 | March     |             1816 |                2 |                   0 |                 8 |                9 |
| 2012 |    4 | April     |             3558 |               11 |                   0 |                75 |               69 |
| 2012 |    5 | May       |             3398 |               24 |                   0 |               147 |              148 |
| 2012 |    6 | June      |             3616 |               26 |                   0 |               196 |              173 |
| 2012 |    7 | July      |             3752 |               43 |                   0 |               202 |              183 |
| 2012 |    8 | August    |             4895 |              680 |                   0 |               266 |              249 |
| 2012 |    9 | September |             4517 |             1443 |                   0 |               333 |              288 |
| 2012 |   10 | October   |             5452 |             1757 |                   0 |               420 |              437 |
| 2012 |   11 | November  |             8045 |             2581 |                   0 |               508 |              466 |

## Requirement 5

### Pull session to order conversion rates, by months for first 8 months.

### Solution

~~~~mysql
SELECT 
	YEAR(website_sessions.created_at) AS yr,
	MONTH(website_sessions.created_at) AS mnth,
    MONTHNAME(website_sessions.created_at) AS mnth_name,
    COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
    COUNT(DISTINCT(orders.order_id)) AS orders,
    (COUNT(DISTINCT(orders.order_id)) / 
    COUNT(DISTINCT(website_sessions.website_session_id))) AS session_order_cvr
FROM website_sessions
	LEFT JOIN orders
		ON website_sessions.website_session_id = orders.website_session_id
WHERE 
website_sessions.created_at <= '2012-11-27'
GROUP BY 
	yr,	
	mnth
ORDER BY 
	mnth ASC
LIMIT 8
;
~~~~

*Output :-*

| yr   | mnth | mnth_name | sessions | orders | session_order_cvr |
|------|------|-----------|----------|--------|-------------------|
| 2012 |    3 | March     |     1835 |     59 |            0.0322 |
| 2012 |    4 | April     |     3713 |    100 |            0.0269 |
| 2012 |    5 | May       |     3717 |    106 |            0.0285 |
| 2012 |    6 | June      |     4011 |    139 |            0.0347 |
| 2012 |    7 | July      |     4180 |    170 |            0.0407 |
| 2012 |    8 | August    |     6090 |    229 |            0.0376 |
| 2012 |    9 | September |     6581 |    284 |            0.0432 |
| 2012 |   10 | October   |     8066 |    363 |            0.0450 |

## Requirement 6

### For the gsearch lander test, estimate the revenue that test generated.

(Hint: Look at the increase in CVR from the test (Jun 19 – Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value)

### Solution

First find out the first pageview_id for '/lander-1' i.e. id of first pageview when test page was launched, the subquery inside **WHERE** clause will do this.

Then get the first pageview_id & pageview_url for each website_session_id after the launch of test '/lander-1' and where pageview_url either '/home' or '/lander-1'.

~~~~mysql
CREATE TEMPORARY TABLE gsearch_nonbrand_firstPV
SELECT 
	website_sessions.website_session_id AS website_session_id,
	MIN(website_pageviews.website_pageview_id) AS first_pv,
    website_pageviews.pageview_url AS pv_url
FROM website_pageviews
	LEFT JOIN website_sessions
		ON website_pageviews.website_session_id = website_sessions.website_session_id
WHERE website_pageviews.website_pageview_id >= (
    -- Find pageview_id when '/lander-1' was launched
	SELECT
		MIN(website_pageview_id) AS first_pv
	FROM website_pageviews
	WHERE pageview_url = "/lander-1"
) 
AND website_sessions.created_at < '2012-07-28' 
AND website_sessions.utm_source = "gsearch" 
AND website_sessions.utm_campaign = "nonbrand" 
AND website_pageviews.pageview_url IN ('/home', '/lander-1')
GROUP BY
	website_sessions.website_session_id
;
~~~~

*Output :-*

~~~~mysql
SELECT * FROM gsearch_nonbrand_firstPV LIMIT 5;
~~~~

| website_session_id | first_pv | pv_url    |
|--------------------|----------|-----------|
|              11683 |    23504 | /lander-1 |
|              11684 |    23505 | /home     |
|              11685 |    23506 | /lander-1 |
|              11686 |    23507 | /lander-1 |
|              11687 |    23509 | /home     |

Get the order_id for each website_sesson_id in temporary table **gsearch_nonbrand_firstPV**

~~~~mysql
CREATE TEMPORARY TABLE gsearch_nonbrand_firstPV_w_orderId
SELECT 
	gsearch_nonbrand_firstPV.website_session_id AS website_session_id,
    gsearch_nonbrand_firstPV.pv_url AS pv_url,
    orders.order_id AS order_id
FROM gsearch_nonbrand_firstPV
	LEFT JOIN orders
		ON gsearch_nonbrand_firstPV.website_session_id = orders.website_session_id
;
~~~~

*Output :-*

~~~~mysql
SELECT * FROM gsearch_nonbrand_firstPV_w_orderId LIMIT 5;
~~~~

| website_session_id | pv_url    | order_id |
|--------------------|-----------|----------|
|              11683 | /lander-1 |     NULL |
|              11684 | /home     |     NULL |
|              11685 | /lander-1 |     NULL |
|              11686 | /lander-1 |     NULL |
|              11687 | /home     |     NULL |

Get the number of sessions, orders and CVR for each pageview_url from temporary table **gsearch_nonbrand_firstPV_w_orderId**

~~~~mysql
SELECT
	pv_url, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    COUNT(DISTINCT order_id) AS orders,
    COUNT(DISTINCT order_id)/COUNT(DISTINCT website_session_id) AS conv_rate
FROM gsearch_nonbrand_firstPV_w_orderId
GROUP BY pv_url
;
~~~~

*Output :-*

| pv_url    | sessions | orders | conv_rate |
|-----------|----------|--------|-----------|
| /home     |     2234 |     71 |    0.0318 |
| /lander-1 |     2287 |     94 |    0.0411 |

~~~~mysql
SELECT 
	COUNT(website_session_id) AS sessions_since_test
FROM website_sessions
WHERE created_at < '2012-11-27'
	AND website_session_id > (
		--  most recent pageview for gsearch nonbrand where the traffic was sent to /home
		SELECT 
			MAX(website_sessions.website_session_id) AS most_recent_gsearch_nonbrand_home_pageview 
		FROM website_sessions 
			LEFT JOIN website_pageviews 
				ON website_pageviews.website_session_id = website_sessions.website_session_id
		WHERE utm_source = 'gsearch'
			AND utm_campaign = 'nonbrand'
			AND pageview_url = '/home'
			AND website_sessions.created_at < '2012-11-27'
    )
	AND utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand'
;
~~~~

*Output :-*

| sessions_since_test |
|---------------------|
|               22024 |

## Requirement 7

### For the landing page test analyzed previously, show a full conversion funnel from each of the two pages to orders.

Use the time period as last time (Jun 19 – Jul 28).

### Solution

Check clicks on each pages for each session id

~~~~mysql
CREATE TEMPORARY TABLE pageview_level_clk
SELECT 
	website_sessions.website_session_id AS website_session_id, 
    website_pageviews.pageview_url AS pageview_url,
	CASE WHEN website_pageviews.pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
    CASE WHEN website_pageviews.pageview_url = '/lander-1' THEN 1 ELSE 0 END AS test_lander,
    CASE WHEN website_pageviews.pageview_url = '/products' THEN 1 ELSE 0 END AS products,
    CASE WHEN website_pageviews.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy,
    CASE WHEN website_pageviews.pageview_url = '/cart' THEN 1 ELSE 0 END AS cart,
    CASE WHEN website_pageviews.pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping,
    CASE WHEN website_pageviews.pageview_url = '/billing' THEN 1 ELSE 0 END AS billing,
    CASE WHEN website_pageviews.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou
FROM website_sessions
	LEFT JOIN website_pageviews
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch' 
	AND website_sessions.utm_campaign = 'nonbrand' 
    AND website_sessions.created_at < '2012-07-28'
    AND website_sessions.created_at > '2012-06-19'
ORDER BY 
	website_sessions.website_session_id,
    website_pageviews.created_at
;
~~~~

*Output :-*

~~~~mysql
SELECT * FROM pageview_level_clk LIMIT 5;
~~~~

| website_session_id | pageview_url           | homepage | test_lander | products | mrfuzzy | cart | shipping | billing | thankyou |
|--------------------|------------------------|----------|-------------|----------|---------|------|----------|---------|----------|
|              11618 | /home                  |        1 |           0 |        0 |       0 |    0 |        0 |       0 |        0 |
|              11618 | /products              |        0 |           0 |        1 |       0 |    0 |        0 |       0 |        0 |
|              11618 | /the-original-mr-fuzzy |        0 |           0 |        0 |       1 |    0 |        0 |       0 |        0 |
|              11618 | /cart                  |        0 |           0 |        0 |       0 |    1 |        0 |       0 |        0 |
|              11618 | /shipping              |        0 |           0 |        0 |       0 |    0 |        1 |       0 |        0 |

~~~~mysql
CREATE TEMPORARY TABLE session_level_clk
SELECT 
	website_session_id,
	MAX(homepage) AS homepage_clk,
    MAX(test_lander) AS test_lander_clk,
    MAX(products) AS products_clk,
    MAX(mrfuzzy) AS mrfuzzy_clk,
    MAX(cart) AS cart_clk,
    MAX(shipping) AS shipping_clk,
    MAX(billing) AS billing_clk,
    MAX(thankyou) AS thankyou_clk
FROM pageview_level_clk
GROUP BY website_session_id
;
~~~~

*Output :-*

~~~~mysql
SELECT * FROM session_level_clk LIMIT 5;
~~~~

| website_session_id | homepage_clk | test_lander_clk | products_clk | mrfuzzy_clk | cart_clk | shipping_clk | billing_clk | thankyou_clk |
|--------------------|--------------|-----------------|--------------|-------------|----------|--------------|-------------|--------------|
|              11618 |            1 |               0 |            1 |           1 |        1 |            1 |           1 |            1 |
|              11619 |            1 |               0 |            0 |           0 |        0 |            0 |           0 |            0 |
|              11620 |            1 |               0 |            1 |           1 |        1 |            1 |           0 |            0 |
|              11621 |            1 |               0 |            1 |           0 |        0 |            0 |           0 |            0 |
|              11622 |            1 |               0 |            0 |           0 |        0 |            0 |           0 |            0 |

~~~~mysql
SELECT 
	CASE 
		WHEN homepage_clk = 1 THEN 'saw_homepage'
		WHEN test_lander_clk = 1 THEN 'saw_test_lander'
		ELSE 'error' 
    END AS segment,
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN products_clk = 1 THEN website_session_id ELSE NULL END) AS to_products,
    COUNT(DISTINCT CASE WHEN mrfuzzy_clk = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN cart_clk = 1 THEN website_session_id ELSE NULL END) AS to_cart,
    COUNT(DISTINCT CASE WHEN shipping_clk = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
    COUNT(DISTINCT CASE WHEN billing_clk = 1 THEN website_session_id ELSE NULL END) AS to_billing,
    COUNT(DISTINCT CASE WHEN thankyou_clk = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM session_level_clk
GROUP BY segment
;
~~~~

*Output :-*

| segment         | sessions | to_products | to_mrfuzzy | to_cart | to_shipping | to_billing | to_thankyou |
|-----------------|----------|-------------|------------|---------|-------------|------------|-------------|
| saw_homepage    |     2288 |         964 |        699 |     304 |         205 |        173 |          75 |
| saw_test_lander |     2287 |        1073 |        767 |     345 |         229 |        195 |          94 |

## Requirement 8

### Quantify the impact of our billing test, as well. Analyze the lift generated from the test (Sep 10 – Nov 10), in terms of revenue per billing page session, and then pull the number of billing page sessions for the past month to understand monthly impact.

### Solution

Get the billing page version and price for each session and order and then ratio of sum of price and count of session id as revenue per serssion for each billing page. 

~~~~mysql
SELECT
	billing_version_seen, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    SUM(price_usd)/COUNT(DISTINCT website_session_id) AS revenue_per_billing_page_seen
 FROM( 
		-- this subquery provides billing page version and order price of each session and order respectively
		SELECT 
			website_pageviews.website_session_id, 
			website_pageviews.pageview_url AS billing_version_seen, 
			orders.order_id, 
			orders.price_usd
		FROM website_pageviews 
			LEFT JOIN orders
				ON orders.website_session_id = website_pageviews.website_session_id
		WHERE website_pageviews.created_at > '2012-09-10' -- prescribed in assignment
			AND website_pageviews.created_at < '2012-11-10' -- prescribed in assignment
			AND website_pageviews.pageview_url IN ('/billing','/billing-2')
	) AS billing_pageviews_and_order_data
GROUP BY billing_version_seen
;
~~~~

| billing_version_seen | sessions | revenue_per_billing_page_seen |
|----------------------|----------|-------------------------------|
| /billing             |      656 |                     23.013689 |
| /billing-2           |      649 |                     31.349661 |

Count session id for all billing page versions for last month.

~~~~mysql
SELECT 
	COUNT(website_session_id) AS billing_sessions_past_month
FROM website_pageviews 
WHERE website_pageviews.pageview_url IN ('/billing','/billing-2') 
	AND created_at BETWEEN '2012-10-27' AND '2012-11-27' -- past month
;
~~~~

| billing_sessions_past_month |
|-----------------------------|
|                        1091 |
