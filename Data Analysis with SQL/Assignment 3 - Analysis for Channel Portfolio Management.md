# Assignment 1: Analysis for Channel Portfolio Management

![image.png](attachment:image.png)

## *Test Skill* - Analyzing Channel Porfolio

### Task 1 - Second paid search channel "bsearch" launched around August 22, pull weekly trending session volume since then and compare to gsearch nonbrand.

### Solution: 

~~~~mysql
# weekly trended session vol
# date > '2012-08-22' and < '2012-11-29'
# separate sessions for gsearch and bsearch
# utm_campaign = 'nonbrand'


SELECT 
    MIN(DATE(created_at)) AS week_start_date, -- as week_start_date
    COUNT(DISTINCT(website_session_id)) AS total_sessions, -- as total_sessions
    COUNT(DISTINCT(CASE WHEN utm_source="gsearch" THEN website_session_id else NULL END)) AS gsearch, -- as gsearch_sessions
    COUNT(DISTINCT(CASE WHEN utm_source="bsearch" THEN website_session_id ELSE NULL END)) AS bsearch -- as bsearch_sessions
FROM website_sessions
WHERE created_at > '2012-08-22'
    AND created_at < '2012-11-29'
    AND utm_campaign = 'nonbrand'
GROUP BY YEARWEEK(created_at)
;
~~~~

*Outcome:-*

| week_start_date | total_sessions | gsearch | bsearch |
|-----------------|----------------|---------|---------|
| 2012-08-22      |            857 |     647 |     210 |
| 2012-08-26      |           1393 |    1053 |     340 |
| 2012-09-02      |           1212 |     918 |     294 |
| 2012-09-09      |           1282 |     961 |     321 |
| 2012-09-16      |           1513 |    1146 |     367 |
| 2012-09-23      |           1373 |    1053 |     320 |
| 2012-09-30      |           1332 |    1006 |     326 |
| 2012-10-07      |           1320 |     998 |     322 |
| 2012-10-14      |           1662 |    1240 |     422 |
| 2012-10-21      |           1736 |    1317 |     419 |
| 2012-10-28      |           1589 |    1198 |     391 |
| 2012-11-04      |           1780 |    1345 |     435 |
| 2012-11-11      |           1677 |    1250 |     427 |
| 2012-11-18      |           4601 |    3499 |    1102 |
| 2012-11-25      |           2991 |    2243 |     748 |

## *Test Skill* - Comparing Channel Characterstics

### Task 2 - Pull sessions data for bsearch and gsearch nonbrand campaign by device type and aggreate data since August 22, 2012.

### Solution: 

~~~~mysql
# created at > '2012-08-22' & < '2012-11-30'
# utm_campaign = "nonbrand"
# sessions by device types where device_type = "mobile"
# calculte percentage of sessions by mobile / total sessions
# aggregate by utm_source => index

SELECT
    utm_source,
    COUNT(DISTINCT(website_session_id)) AS total_sessions,
    COUNT(DISTINCT(CASE WHEN device_type="mobile" THEN website_session_id ELSE NULL END)) AS mobile_sessions,
    ( COUNT(DISTINCT(CASE WHEN device_type="mobile" THEN website_session_id ELSE NULL END)) / 
        COUNT(DISTINCT(website_session_id))
        ) AS pct_mobile_session
FROM website_sessions
WHERE created_at > '2012-08-22'
    AND created_at < '2012-11-29'
    AND utm_campaign = "nonbrand"
GROUP BY utm_source
;
~~~~

*Outcome:-*

| utm_source | total_sessions | mobile_sessions | pct_mobile_session |
|------------|----------------|-----------------|--------------------|
| bsearch    |           6444 |             554 |             0.0860 |
| gsearch    |          19874 |            4864 |             0.2447 |

### *Test Skill* - Cross Channel Bid Optimization

### Task 3 - Pull nonbrand conversion rates from session to order for gsearch and bsearch, and slice the data by device type. Analyze data from August 22, 2012 to September 18, 2012.

### Solution: 

~~~~mysql
SELECT
    website_sessions.device_type,
    website_sessions.utm_source,
    COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
    COUNT(DISTINCT(orders.order_id)) AS orders,
    (COUNT(DISTINCT(orders.order_id)) /
    COUNT(DISTINCT(website_sessions.website_session_id))
    ) AS cvr
FROM website_sessions
    LEFT JOIN orders
        ON website_sessions.website_session_id = orders.website_session_id
WHERE website_sessions.created_at > '2012-08-22'
    AND website_sessions.created_at < '2012-09-18'
    AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY 1, 2
;
~~~~

*Outcome:-*

| device_type | utm_source | sessions | orders | cvr    |
|-------------|------------|----------|--------|--------|
| desktop     | bsearch    |     1112 |     42 | 0.0378 |
| desktop     | gsearch    |     2834 |    132 | 0.0466 |
| mobile      | bsearch    |      124 |      1 | 0.0081 |
| mobile      | gsearch    |      955 |     11 | 0.0115 |

## *Test Skill* - Analyzing channel Portfolio Trends

### Task 4 - Pull weekly session volumne for gsearch and bsearch nonbrand, broken down by device, since November 4th and include a comparision metric to show bsearch as percentage of gsearch for each device type.

### Solution: 

~~~~mysql
# created_at > '2012-11-04' & < '2012-12-22'
# utm_campaign = "nonbrand"
# find week_start_date
# count of session & order id for each gsearch & bsearch aggregated by device type
# percentage of bsearch to gsearch for each device type

SELECT
    MIN(DATE(created_at)) AS week_start_date,
    COUNT(DISTINCT(CASE WHEN (utm_source="bsearch" AND device_type="desktop") THEN website_session_id ELSE NULL END)) AS bdesk_session,
    COUNT(DISTINCT(CASE WHEN (utm_source="gsearch" AND device_type="desktop") THEN website_session_id ELSE NULL END)) AS gdesk_session,
    (COUNT(DISTINCT(CASE WHEN (utm_source="bsearch" AND device_type="desktop") THEN website_session_id ELSE NULL END)) / 
        COUNT(DISTINCT(CASE WHEN (utm_source="gsearch" AND device_type="desktop") THEN website_session_id ELSE NULL END))
    ) AS b2g_desk_pct, 
    COUNT(DISTINCT(CASE WHEN (utm_source="bsearch" AND device_type="mobile") THEN website_session_id ELSE NULL END)) AS bmob_session,
    COUNT(DISTINCT(CASE WHEN (utm_source="gsearch" AND device_type="mobile") THEN website_session_id ELSE NULL END)) AS gmob_session,
    (COUNT(DISTINCT(CASE WHEN (utm_source="bsearch" AND device_type="mobile") THEN website_session_id ELSE NULL END)) / 
        COUNT(DISTINCT(CASE WHEN (utm_source="gsearch" AND device_type="mobile") THEN website_session_id ELSE NULL END))
    ) AS b2g_mob_pct
FROM website_sessions
WHERE created_at > '2012-11-04'
    AND created_at < '2012-12-22'
    AND utm_campaign = "nonbrand"
GROUP BY YEARWEEK(created_at)
;
~~~~

*Outcome:-*

| week_start_date | bdesk_session | gdesk_session | b2g_desk_pct | bmob_session | gmob_session | b2g_mob_pct |
|-----------------|---------------|---------------|--------------|--------------|--------------|-------------|
| 2012-11-04      |           404 |          1018 |       0.3969 |           31 |          327 |      0.0948 |
| 2012-11-11      |           394 |           960 |       0.4104 |           33 |          290 |      0.1138 |
| 2012-11-18      |          1013 |          2652 |       0.3820 |           89 |          847 |      0.1051 |
| 2012-11-25      |           846 |          2068 |       0.4091 |           61 |          698 |      0.0874 |
| 2012-12-02      |           507 |          1320 |       0.3841 |           29 |          390 |      0.0744 |
| 2012-12-09      |           294 |          1260 |       0.2333 |           47 |          428 |      0.1098 |
| 2012-12-16      |           336 |          1206 |       0.2786 |           40 |          355 |      0.1127 |

## *Test Skill* - Analyzing Direct Traffic

### Task 5 - Pull organic search, direct type in, and paid brand search sessions by month, and show those sesions a % of paid search nonbrand.

### Solution: 

~~~~mysql
# created at < '2012-12-23'
# get month from created_at
# sessions by nonbrand, brand, direct, organic
# percentatge of brand, direct, organic to nonbrand
# aggregate by month

SELECT
    YEAR(created_at) AS yr,
    MONTH(created_at) AS mnth,
    COUNT(DISTINCT(CASE WHEN utm_campaign='nonbrand' THEN website_session_id ELSE NULL END)) AS nonbrand, 
    COUNT(DISTINCT(CASE WHEN utm_campaign='brand' THEN website_session_id ELSE NULL END)) AS brand,
    (COUNT(DISTINCT(CASE WHEN utm_campaign='brand' THEN website_session_id ELSE NULL END))  /
        COUNT(DISTINCT(CASE WHEN utm_campaign='nonbrand' THEN website_session_id ELSE NULL END))
    ) AS b2nb_rate,
    COUNT(DISTINCT(CASE WHEN utm_campaign IS NULL AND http_referer IS NOT NULL THEN website_session_id ELSE NULL END)) AS direct,
    (COUNT(DISTINCT(CASE WHEN utm_campaign IS NULL AND http_referer IS NOT NULL THEN website_session_id ELSE NULL END)) /
        COUNT(DISTINCT(CASE WHEN utm_campaign='nonbrand' THEN website_session_id ELSE NULL END))
    ) AS d2nb_rate,
    COUNT(DISTINCT(CASE WHEN utm_campaign IS NULL AND http_referer IS NULL THEN website_session_id ELSE NULL END)) AS organic,
    (COUNT(DISTINCT(CASE WHEN utm_campaign IS NULL AND http_referer IS NULL THEN website_session_id ELSE NULL END)) /
        COUNT(DISTINCT(CASE WHEN utm_campaign='nonbrand' THEN website_session_id ELSE NULL END))
    ) AS o2nb_rate
FROM website_sessions
WHERE created_at < '2012-12-23'
GROUP BY 1, 2
;
~~~~

*Outcome:-*

| yr   | mnth | nonbrand | brand | b2nb_rate | direct | d2nb_rate | organic | o2nb_rate |
|------|------|----------|-------|-----------|--------|-----------|---------|-----------|
| 2012 |    3 |     1808 |    10 |    0.0055 |      8 |    0.0044 |       9 |    0.0050 |
| 2012 |    4 |     3497 |    72 |    0.0206 |     75 |    0.0214 |      69 |    0.0197 |
| 2012 |    5 |     3283 |   139 |    0.0423 |    147 |    0.0448 |     148 |    0.0451 |
| 2012 |    6 |     3477 |   165 |    0.0475 |    196 |    0.0564 |     173 |    0.0498 |
| 2012 |    7 |     3599 |   196 |    0.0545 |    202 |    0.0561 |     183 |    0.0508 |
| 2012 |    8 |     5315 |   260 |    0.0489 |    266 |    0.0500 |     249 |    0.0468 |
| 2012 |    9 |     5617 |   343 |    0.0611 |    333 |    0.0593 |     288 |    0.0513 |
| 2012 |   10 |     6788 |   421 |    0.0620 |    420 |    0.0619 |     437 |    0.0644 |
| 2012 |   11 |    12309 |   558 |    0.0453 |    628 |    0.0510 |     567 |    0.0461 |
| 2012 |   12 |     6673 |   473 |    0.0709 |    491 |    0.0736 |     490 |    0.0734 |
