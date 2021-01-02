# Assignment 2 - Website Performance Analysis

![img](https://i.imgur.com/RQ5G8BD.png)

## *Test Skill -* Identifying Top Website Pages 

### Task 1 - Pull the most viewed website pages, ranked by pageviews volumne. Use 2012-06-09 as baseline.

### Solution :

- Use *created_at* to filter data by date
- Distinct count the *website_pageview_id* as sessions
- Group by the sessions using *pageview_url*

~~~~mysql
SELECT 
    pageview_url,
    COUNT(DISTINCT(website_pageview_id)) AS pvs,
-- Calculate the percentage for each row based on column total    
    (
        COUNT(DISTINCT(website_pageview_id)) / 
        (SELECT
             COUNT(website_pageview_id)
         FROM website_pageviews 
         WHERE created_at < '2012-06-09')
    ) AS pvs_pct
FROM website_pageviews
WHERE created_at < '2012-06-09'
GROUP BY pageview_url
ORDER BY pvs_pct DESC;
~~~~

*Output :-*

| pageview_url              | pvs      | pvs_pct     |
|---------------------------|----------|-------------|
| /home                     |    10403 |      0.4983 |
| /products                 |     4239 |      0.2031 |
| /the-original-mr-fuzzy    |     3037 |      0.1455 |
| /cart                     |     1306 |      0.0626 |
| /shipping                 |      869 |      0.0416 |
| /billing                  |      716 |      0.0343 |
| /thank-you-for-your-order |      306 |      0.0147 |

**/home** has highest sessions ~50% of total sessions, followed by **/products** page with ~20% sessions.

## *Test Skill -* Identifying Top Entry Pages 

### Task 2 - Pull all the entry pages and rank them on entry volume.

### Solution :

First we will created a **Temporary Table** for each *website_session_id* with minimum value of *website_pageview_id* because MIN(website_pageview_id) = entry page.

This will give us data for each session id and their landing / entry page.

It will just create a temporary table and doesn't return any data. To see the data we have to use `SELECT * FROM <temporary table>`.

~~~~mysql
-- Drop the table if it exists before creating temporary table
DROP TABLE IF EXISTS entry_pages;
~~~~

~~~~mysql
-- Create temporary table
CREATE TEMPORARY TABLE entry_pages

-- All the data returned from below SELECT statement will be channeled into the temporary table
SELECT 
    website_session_id,
    MIN(website_pageview_id) AS entry_page_id
FROM website_pageviews
WHERE created_at < '2012-06-12'
GROUP BY website_session_id;
~~~~

Now we will do a left join of **website_pageviews** table and our temporary table **entry_pages** and we will count pageview id (*entry_page_id*) from temporary table grouped by *pageview_url* from website_pageviews table.

~~~~sql
SELECT 
    pageview_url,
    COUNT(entry_pages.entry_page_id) AS entry_pv
FROM website_pageviews
LEFT JOIN entry_pages
    ON entry_pages.entry_page_id = website_pageviews.website_pageview_id
WHERE created_at < '2012-06-12'
GROUP BY pageview_url
ORDER BY entry_pv DESC;
~~~~

*Output :-*

| pageview_url              | entry_pv           |
|---------------------------|--------------------|
| /home                     |              10714 |
| /shipping                 |                  0 |
| /billing                  |                  0 |
| /the-original-mr-fuzzy    |                  0 |
| /products                 |                  0 |
| /cart                     |                  0 |
| /thank-you-for-your-order |                  0 |

All the pageview are from home page.

![img](https://i.imgur.com/WSoP51x.png)

## *Test Skill -* Calculating Bouce Rate

### Task 3 - Pull the bounce rates for traffic landing on the homepage. Report should include *sessions*, *bounced sessions* and *% of sessions which bounced*.

### Solution :

**STEP 1** - Create temporary table consisting first *website_pageview_id* as **min_pv_id** for each *website_session_id*

~~~~mysql
CREATE TEMPORARY TABLE first_pageviews_tb
SELECT 
    website_session_id AS session_id,
    MIN(website_pageview_id) AS min_pv_id
FROM website_pageviews
WHERE created_at < '2012-06-14'
GROUP BY website_session_id
;

-- Lets look into first 5 results from the table
SELECT * FROM first_pageviews_tb LIMIT 5;
~~~~

| website_session_id | min_pv_id |
|--------------------|-----------|
|                  1 |         1 |
|                  2 |         2 |
|                  3 |         3 |
|                  4 |         4 |
|                  5 |         5 |

**STEP 2** - Create temporary table for *session_id* with first pageview id (min_pv_id) and corresponding landing page url as **landing_page**. Filter out the *pageview_url* and keep only **/home** as landing page.

~~~~mysql
CREATE TEMPORARY TABLE session_w_landing_page_tb

-- for each session if from the *first_pageviews* table, get the landing page url as '/home'
-- left join first_pageiews and website_pageviews using pageview id
SELECT 
    first_pageviews_tb.session_id AS session_id,
    website_pageviews.pageview_url AS landing_page
FROM first_pageviews_tb
LEFT JOIN website_pageviews
    ON first_pageviews_tb.min_pv_id = website_pageviews.website_pageview_id
WHERE website_pageviews.pageview_url = '/home'
;

-- Lets look into first 5 results from the table
SELECT * FROM session_w_landing_page_tb LIMIT 5;
~~~~

| session_id | landing_page |
|------------|--------------|
|          1 | /home        |
|          2 | /home        |
|          3 | /home        |
|          4 | /home        |
|          5 | /home        |

**STEP 3** - Create temporary table for *session_id* which has only 1 pageview count for *landing_page*, which means users came to that landing page and didn't visit any other page during that session, hence user bounced.

~~~~mysql
CREATE TEMPORARY TABLE bounced_sessions_tb

-- bounced_session_id -> session_id for which landing_page has count of website_pageview_id = 1
SELECT 
    session_w_landing_page_tb.session_id AS bounced_session_id,
    session_w_landing_page_tb.landing_page AS bounced_landing_page,
    COUNT(DISTINCT(website_pageviews.website_pageview_id)) AS pv_count
FROM session_w_landing_page_tb
LEFT JOIN website_pageviews
    ON session_w_landing_page_tb.session_id = website_pageviews.website_session_id
GROUP BY 
    bounced_session_id,
    bounced_landing_page
HAVING pv_count = 1    
;

-- Lets look into first 5 results from the table
SELECT * FROM bounced_sessions_tb LIMIT 5;
~~~~

| bounced_session_id | bounced_landing_page | pv_count |
|--------------------|----------------------|----------|
|                  1 | /home                |        1 |
|                  2 | /home                |        1 |
|                  3 | /home                |        1 |
|                  4 | /home                |        1 |
|                  5 | /home                |        1 |

**STEP 4** - Count distinct values of *session_id* as *sessions* from **session_w_landing_page** and *bounced_session_id* as *bounced_sessions* from **bounced_sessions_tb**

~~~~mysql
SELECT 
    COUNT(DISTINCT(session_w_landing_page_tb.session_id)) AS sessions,
    COUNT(DISTINCT(bounced_sessions_tb.bounced_session_id)) AS bounced_sessions,
    COUNT(DISTINCT(bounced_sessions_tb.bounced_session_id)) / COUNT(DISTINCT(session_w_landing_page_tb.session_id)) AS bounce_rate
FROM session_w_landing_page_tb
LEFT JOIN bounced_sessions_tb
    ON session_w_landing_page_tb.session_id = bounced_sessions_tb.bounced_session_id
;
~~~~~

| sessions | bounced_sessions | bounce_rate |
|----------|------------------|-------------|
|    10965 |             6488 |      0.5917 |

Homepage has near around 60% bounce rate which is quite high.

## *Test Skill -* Analyzing Landing Page Tests

### Task 4 :

### Solution :

First let's find out the first date and first pageview id for **/lander-1** page url in the database

~~~~mysql
SELECT
    pageview_url,
    MIN(DATE(created_at)) AS launch_date,
    MIN(website_pageview_id) AS first_pv_id
FROM website_pageviews
WHERE pageview_url = '/lander-1'
    AND created_at IS NOT NULL
;
~~~~

| pageview_url | launch_date | first_pv_id |
|--------------|-------------|-------------|
| /lander-1    | 2012-06-19  |       23504 |

Create temporary table for session id with corresponding first pageview id

~~~~mysql
CREATE TEMPORARY TABLE first_pageviews_tb
SELECT 
    website_pageviews.website_session_id AS session_id,
    MIN(website_pageviews.website_pageview_id) AS min_pv_id
FROM website_pageviews
INNER JOIN website_sessions
    ON website_sessions.website_session_id = website_pageviews.website_session_id
    AND website_sessions.created_at < '2012-07-28'
    AND website_pageviews.website_pageview_id > 23504
    AND website_sessions.utm_source = 'gsearch'
    AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY website_pageviews.website_session_id
;
~~~~

Create temporary table for sessions based on first pageview id and their corresponding landing page url

~~~~mysql
CREATE TEMPORARY TABLE session_w_landing_page_tb

-- for each session if from the *first_pageviews* table, get the landing page url as '/home'
-- left join first_pageiews and website_pageviews using pageview id
SELECT 
    first_pageviews_tb.session_id AS session_id,
    website_pageviews.pageview_url AS landing_page
FROM first_pageviews_tb
LEFT JOIN website_pageviews
    ON first_pageviews_tb.min_pv_id = website_pageviews.website_pageview_id
-- check for both '/home' and '/lander-1'
WHERE website_pageviews.pageview_url IN ('/home', '/lander-1')
;
~~~~

Create temporary table for sessions with landing page and their PV = 1

~~~~mysql
CREATE TEMPORARY TABLE bounced_sessions_tb

-- bounced_session_id -> session_id for which landing_page has count of website_pageview_id = 1
SELECT 
    session_w_landing_page_tb.session_id AS bounced_session_id,
    session_w_landing_page_tb.landing_page AS bounced_landing_page,
    COUNT(DISTINCT(website_pageviews.website_pageview_id)) AS pv_count
FROM session_w_landing_page_tb
LEFT JOIN website_pageviews
    ON session_w_landing_page_tb.session_id = website_pageviews.website_session_id
GROUP BY 
    bounced_session_id,
    bounced_landing_page
HAVING pv_count = 1    
;
~~~~

Group by data based on *landing_page* and get distinct count of session_id, bounced_sesson_id and their ratio as bounce rate.

~~~~mysql
SELECT
    session_w_landing_page_tb.landing_page AS test_lander,
    COUNT(DISTINCT(session_w_landing_page_tb.session_id)) AS sessions,
    COUNT(DISTINCT(bounced_sessions_tb.bounced_session_id)) AS bounced_sessions,
    COUNT(DISTINCT(bounced_sessions_tb.bounced_session_id)) / COUNT(DISTINCT(session_w_landing_page_tb.session_id)) AS bounce_rate
FROM session_w_landing_page_tb
LEFT JOIN bounced_sessions_tb
    ON session_w_landing_page_tb.session_id = bounced_sessions_tb.bounced_session_id
GROUP BY test_lander
;
~~~~~

*Final Output :-*

| lander    | sessions | bounced_sessions | bounce_rate |
|-----------|----------|------------------|-------------|
| /home     |     2234 |             1304 |      0.5837 |
| /lander-1 |     2286 |             1213 |      0.5306 |

**/home** has bounce rate of around 58% while **/lander-1** has bounce rate of 53%. /lander-1 is more effective.

## *Test Skill -* Landing Page Trend Analysis

### Task 5 - Get the data for volume of paid search nonbrand traffic landing on /home and /lander-1, trended weekly since 2012-06-01. Also get the overall paid search bounce rate trended weekly.

### Solution :

**STEP 1** - Get min pageview id for each session id and count of total pageview id

~~~~mysql
CREATE TEMPORARY TABLE first_pageview_tb

SELECT 
	website_sessions.website_session_id AS session_id,
	MIN(website_pageviews.website_pageview_id) AS min_pv_id,
	COUNT(website_pageviews.website_pageview_id) AS pv_count
FROM website_sessions
	LEFT JOIN website_pageviews
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.created_at > '2012-06-01'
	AND website_sessions.created_at < '2012-08-31'
	AND website_sessions.utm_source = 'gsearch'
	AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY 
	session_id
~~~~

~~~~mysql
-- Lets look into first 5 results from the table
SELECT * FROM first_pageview_tb LIMIT 5;
~~~~

| session_id | min_pv_id | pv_count |
|------------|-----------|----------|
|       9266 |     18412 |        1 |
|       9267 |     18413 |        2 |
|       9268 |     18414 |        1 |
|       9269 |     18416 |        2 |
|       9270 |     18418 |        6 |

**STEP 2** - Get pageview_url and created_at for each session_id

~~~~mysql
CREATE TEMPORARY TABLE session_landing_page_date
SELECT 
	first_pageview_tb.session_id,
	first_pageview_tb.min_pv_id,
	first_pageview_tb.pv_count,
	MIN(website_pageviews.created_at) AS min_create_date,
	website_pageviews.pageview_url AS landing_page
FROM first_pageview_tb
	LEFT JOIN website_pageviews
		ON first_pageview_tb.min_pv_id = website_pageviews.website_pageview_id
GROUP BY 
	first_pageview_tb.session_id
;
~~~~

~~~~mysql
-- Lets look into first 5 results from the table
SELECT * FROM session_landing_page_date LIMIT 5;
~~~~

| session_id | min_pv_id | pv_count | min_create_date     | landing_page |
|------------|-----------|----------|---------------------|--------------|
|       9266 |     18412 |        1 | 2012-06-01 00:03:00 | /home        |
|       9267 |     18413 |        2 | 2012-06-01 00:07:31 | /home        |
|       9268 |     18414 |        1 | 2012-06-01 00:09:16 | /home        |
|       9269 |     18416 |        2 | 2012-06-01 00:11:17 | /home        |
|       9270 |     18418 |        6 | 2012-06-01 00:16:42 | /home        |

**STEP 3** 
- Distinct count of session_id as total sessions
- Distinct count of session_id where pv_count = 1 as bounced sessions
- Distinct count of session_id where landing_page = '/home' as home sessions
- Distinct count of session_id where landing_page = '/lander-1' as lander-1 sessions

~~~~mysql
SELECT 
	DATE(session_landing_page_date.min_create_date) AS week_start_date,
-- this gives count of session id as the total sessions
/*	COUNT(DISTINCT(session_landing_page_date.session_id)) AS total_sessions, */
-- this gives count of min_pv_id which have pv_coun = 1 as bounced sessions
/*	COUNT(DISTINCT(CASE WHEN session_landing_page_date.pv_count = 1 
				THEN session_landing_page_date.session_id 
				ELSE NULL END)) AS bounced_sessions, */
	(COUNT(DISTINCT(CASE WHEN session_landing_page_date.pv_count = 1 
                    THEN session_landing_page_date.session_id 
                    ELSE NULL END)) /
	COUNT(DISTINCT(session_landing_page_date.session_id))
    ) AS bounce_rate,
-- this gives count of session id where landing page = '/home' as home session
	COUNT(DISTINCT(CASE WHEN session_landing_page_date.landing_page = '/home' 
                   THEN session_landing_page_date.session_id 
                   ELSE NULL END)) AS home_sessions,
-- this gives count of session id where landing page = '/lander-1' as lander-1 session
	COUNT(DISTINCT(CASE WHEN session_landing_page_date.landing_page = '/lander-1' 
                   THEN session_landing_page_date.session_id 
                   ELSE NULL END)) AS lander1_sessions
FROM session_landing_page_date
GROUP BY 
	YEARWEEK(session_landing_page_date.min_create_date)
;
~~~~

*Final Output :-*

| week_start_date | bounce_rate | home_sessions | lander1_sessions |
|-----------------|-------------|---------------|------------------|
| 2012-06-01      |      0.6000 |           215 |                0 |
| 2012-06-03      |      0.5842 |           796 |                0 |
| 2012-06-10      |      0.6151 |           873 |                0 |
| 2012-06-17      |      0.5639 |           505 |              332 |
| 2012-06-24      |      0.5833 |           365 |              391 |
| 2012-07-01      |      0.5792 |           402 |              387 |
| 2012-07-08      |      0.5686 |           386 |              409 |
| 2012-07-15      |      0.5406 |           425 |              424 |
| 2012-07-22      |      0.5076 |           400 |              392 |
| 2012-07-29      |      0.4985 |            50 |              977 |
| 2012-08-05      |      0.5487 |             0 |             1077 |
| 2012-08-12      |      0.5050 |             0 |              996 |
| 2012-08-19      |      0.5044 |             0 |             1021 |
| 2012-08-26      |      0.5503 |             0 |              776 |

After the end of 2012-07-29 all traffic is coming from **/lander-1**

![img](https://i.imgur.com/miWT8fi.png)

## *Test Skill -* Building Conversion Funnel

### Task 6 - Build a full conversion funnel, analyizing how many customers make it to each step. Start with '/lander-1' and build a funnel all the way to '/thank-you-for-your-order' page. Use data since 2012-08-05.

### Solution :

Conditions given:
1. utm_source = gsearch
2. landing page = /lander-1
3. ending page = /thank-you
4. start date > 2012-08-05
5. end_date < 2012-09-05 

First lets check the pages we have for the given duration excluding **/home** page url.

~~~~mysql
SELECT
	website_pageviews.pageview_url AS page_url,
	COUNT(DISTINCT(website_pageviews.website_pageview_id)) AS pv_count	
FROM website_pageviews
WHERE created_at > '2012-08-05'
	AND created_at < '2012-09-05'
	AND website_pageviews.pageview_url NOT LIKE '/home'
GROUP BY 
	page_url
ORDER BY pv_count DESC
;
~~~~

| page_url                  | pv_count |
|---------------------------|----------|
| /lander-1                 |     5194 |
| /products                 |     2899 |
| /the-original-mr-fuzzy    |     2116 |
| /cart                     |      924 |
| /shipping                 |      619 |
| /billing                  |      484 |
| /thank-you-for-your-order |      207 |

1. create a sub query for seasion id with 1 value for each page_url if viewed during same session as a separate column
2. extract maximum value for each page url and group by session id

~~~~mysql
CREATE TEMPORARY TABLE session_page_level_flags_tb
SELECT 
	session_id,
	MAX(lander1_click_pv) AS lander1_click_pv_max,
	MAX(proucts_click_pv) AS proucts_click_pv_max,
	MAX(mr_fuzzy_click_pv) AS mr_fuzzy_click_pv_max,
	MAX(cart_click_pv) AS cart_click_pv_max,
	MAX(shipping_click_pv) AS shipping_click_pv_max,
	MAX(billing_click_pv) AS billing_click_pv_max,
	MAX(thank_you_click_pv) AS thank_you_click_pv_max
FROM(
	SELECT 
		website_sessions.website_session_id AS session_id,
		(CASE WHEN website_pageviews.pageview_url = '/lander-1' THEN 1 ELSE 0 END) AS lander1_click_pv,
		(CASE WHEN website_pageviews.pageview_url = '/products' THEN 1 ELSE 0 END) AS proucts_click_pv,
		(CASE WHEN website_pageviews.pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END) AS mr_fuzzy_click_pv,
		(CASE WHEN website_pageviews.pageview_url = '/cart' THEN 1 ELSE 0 END) AS cart_click_pv,
		(CASE WHEN website_pageviews.pageview_url = '/shipping' THEN 1 ELSE 0 END) AS shipping_click_pv,
		(CASE WHEN website_pageviews.pageview_url = '/billing' THEN 1 ELSE 0 END) AS billing_click_pv,
		(CASE WHEN website_pageviews.pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END) AS thank_you_click_pv
	FROM website_sessions
		LEFT JOIN website_pageviews
			ON website_sessions.website_session_id = website_pageviews.website_session_id
	WHERE website_sessions.created_at > '2012-08-05'
		AND website_sessions.created_at < '2012-09-05'
		AND website_sessions.utm_source = 'gsearch'
		AND website_sessions.utm_campaign = 'nonbrand'
) AS pageurl_level
GROUP BY 
	session_id
;
~~~~

~~~~mysql
-- Lets look into first 5 results from the table
SELECT * FROM session_page_level_flags_tb LIMIT 5;
~~~~

| session_id | lander1_click_pv_max | proucts_click_pv_max | mr_fuzzy_click_pv_max | cart_click_pv_max | shipping_click_pv_max | billing_click_pv_max | thank_you_click_pv_max |
|------------|----------------------|----------------------|-----------------------|-------------------|-----------------------|----------------------|------------------------|
|      18207 |                    1 |                    1 |                     0 |                 0 |                     0 |                    0 |                      0 |
|      18208 |                    1 |                    1 |                     0 |                 0 |                     0 |                    0 |                      0 |
|      18209 |                    1 |                    0 |                     0 |                 0 |                     0 |                    0 |                      0 |
|      18210 |                    1 |                    0 |                     0 |                 0 |                     0 |                    0 |                      0 |
|      18211 |                    1 |                    1 |                     1 |                 1 |                     0 |                    0 |                      0 |

Count session id each page url column if value is 1, this will give total pageview for each pages

~~~~mysql
CREATE TEMPORARY TABLE sessions_page_level_clk_pv_funnel_tb
SELECT 
	COUNT(CASE WHEN lander1_click_pv_max = 1 THEN session_id ELSE NULL END) AS lander1_total_clk_pv,
	COUNT(CASE WHEN proucts_click_pv_max = 1 THEN session_id ELSE NULL END) AS products_total_clk_pv,
	COUNT(CASE WHEN mr_fuzzy_click_pv_max = 1 THEN session_id ELSE NULL END) AS mr_fuzzy_total_clk_pv, 
	COUNT(CASE WHEN cart_click_pv_max = 1 THEN session_id ELSE NULL END) AS cart_total_clk_pv, 
	COUNT(CASE WHEN shipping_click_pv_max = 1 THEN session_id ELSE NULL END) AS shipping_total_clk_pv, 
	COUNT(CASE WHEN billing_click_pv_max = 1 THEN session_id ELSE NULL END) AS billing_total_clk_pv, 
	COUNT(CASE WHEN thank_you_click_pv_max = 1 THEN session_id ELSE NULL END) AS thank_you_total_clk_pv  
FROM session_page_level_flags_tb
;
~~~~

~~~~mysql
-- Lets look into outcome of this temporary table
SELECT * FROM sessions_page_level_clk_pv_funnel_tb LIMIT 5;
~~~~

| lander1_total_clk_pv | products_total_clk_pv | mr_fuzzy_total_clk_pv | cart_total_clk_pv | shipping_total_clk_pv | billing_total_clk_pv | thank_you_total_clk_pv |
|----------------------|-----------------------|-----------------------|-------------------|-----------------------|----------------------|------------------------|
|                 4441 |                  2090 |                  1549 |               673 |                   446 |                  354 |                    155 |

Find ratio of each page total pageviews to previous page total pageview, this will give Click Through Rate for each page.

~~~~mysql
SELECT 
	products_total_clk_pv / lander1_total_clk_pv AS lander1_ctr,
	mr_fuzzy_total_clk_pv / products_total_clk_pv AS products_ctr,
	cart_total_clk_pv / mr_fuzzy_total_clk_pv AS mar_fuzzy_ctr,
	shipping_total_clk_pv / cart_total_clk_pv AS cart_ctr, 
	billing_total_clk_pv / shipping_total_clk_pv AS shipping_ctr, 
	thank_you_total_clk_pv / billing_total_clk_pv AS billing_ctr
FROM sessions_page_level_clk_pv_funnel_tb
;
~~~~

*Final Output :-*

| lander1_ctr | products_ctr | mar_fuzzy_ctr | cart_ctr | shipping_ctr | billing_ctr |
|-------------|--------------|---------------|----------|--------------|-------------|
|      0.4706 |       0.7411 |        0.4345 |   0.6627 |       0.7937 |      0.4379 |

CTR for **/lander-1**, **/the-original-mr-fuzzy** and **/billing** is much lower.

## *Test Skill -* Analyzing Conversion Funnel Tests

### Task 7 : Check the sessions data for both versions of billing page '/billing' and '/billing-2' along with the conversion rate from billing page to order placement.

### Solution :

Conditions given:
- landing page = /lander-1
- ending page = /thank-you
- test page = /billing-2
- start date > based on test page launch date (2012-09-10)
- end_date < 2012-11-12 


First let's find out the first date and first pageview id for **/lander-1** page url in the database

~~~~mysql
SELECT
    pageview_url,
    MIN(DATE(created_at)) AS launch_date,
    MIN(website_pageview_id) AS first_pv_id
FROM website_pageviews
WHERE pageview_url = '/billing-2'
    AND created_at IS NOT NULL
;
~~~~

| pageview_url | launch_date | first_pv_id |
|--------------|-------------|-------------|
| /billing-2   | 2012-09-10  |       53550 |

We will create a temporary table for sessions id having page visit for different versions of billing and then after doing left join with orders table we will fetch respective order_id for each of the rows.

~~~~mysql
CREATE TEMPORARY TABLE billing_page_ver_orders_tb
SELECT 
	website_pageviews.website_session_id AS session_id,
	website_pageviews.pageview_url AS page_versions,
	orders.order_id AS orders_id
FROM website_pageviews
	LEFT JOIN orders
		ON website_pageviews.website_session_id = orders.website_session_id
WHERE website_pageviews.created_at > '2012-09-10'
	AND website_pageviews.created_at < '2012-11-12'
	AND website_pageviews.pageview_url IN ("/billing", "/billing-2")
;
~~~~

~~~~mysql
-- Lets look into first 5 results from the table
SELECT * FROM billing_page_ver_orders_tb LIMIT 5;
~~~~

| session_id | page_versions | orders_id |
|------------|---------------|-----------|
|      25268 | /billing      |      NULL |
|      25278 | /billing      |       869 |
|      25306 | /billing      |       870 |
|      25325 | /billing-2    |       871 |
|      25343 | /billing      |       872 |

Count distinct session id as sessions and order id as orders and group by page versions. This will give us sessions & orders for both verions of billing page along with it we can also calculate billing page to order conversion rate.

~~~~mysql
SELECT 
	page_versions,
	COUNT(DISTINCT(session_id)) AS sessions,
	COUNT(orders_id) AS sales_order,
	( COUNT(orders_id) / COUNT(DISTINCT(session_id)) ) AS billing_to_order_cvr
FROM billing_page_ver_orders_tb
GROUP BY 
	page_versions
;
~~~~

*Final Output :-*

| page_versions | sessions | sales_order | billing_to_order_cvr |
|---------------|----------|-------------|----------------------|
| /billing      |      669 |         307 |               0.4589 |
| /billing-2    |      662 |         413 |               0.6239 |

New version of billing page has higher conversion rate  than previous one. Kudos!

_____________
## End of Assignment - Website Performance Analysis