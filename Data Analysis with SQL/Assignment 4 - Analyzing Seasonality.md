# Assignment 4 - Business Patterns & Seasonality

## *Test Skill* - Analyzing Seasonality

### Task 1 - Pull monthly and weekly data for the year 2012 to look for the sessions and orders volume patterns.

### Solution: 

~~~~mysql
SELECT
    YEAR(DATE(website_sessions.created_at)) AS yr,
    MONTHNAME(DATE(website_sessions.created_at)) AS mnth,
    COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
    COUNT(DISTINCT(orders.order_id)) AS orders
FROM website_sessions
    LEFT JOIN orders
        ON website_sessions.website_session_id = orders.website_session_id
WHERE YEAR(DATE(website_sessions.created_at)) = '2012'
GROUP BY
    YEAR(DATE(website_sessions.created_at)),
    MONTH(DATE(website_sessions.created_at))
;
~~~~

*Output:-* Yearly & Monthly data

| yr   | mnth      | sessions | orders |
|------|-----------|----------|--------|
| 2012 | March     |     1835 |     59 |
| 2012 | April     |     3713 |    100 |
| 2012 | May       |     3717 |    106 |
| 2012 | June      |     4011 |    139 |
| 2012 | July      |     4180 |    170 |
| 2012 | August    |     6090 |    229 |
| 2012 | September |     6581 |    284 |
| 2012 | October   |     8066 |    363 |
| 2012 | November  |    14062 |    617 |
| 2012 | December  |    10110 |    514 |

~~~~mysql
SELECT
    MIN(DATE(website_sessions.created_at)) AS week_start_date,
    COUNT(DISTINCT(website_sessions.website_session_id)) AS sessions,
    COUNT(DISTINCT(orders.order_id)) AS orders
FROM website_sessions
    LEFT JOIN orders
        ON website_sessions.website_session_id = orders.website_session_id
WHERE YEAR(DATE(website_sessions.created_at)) = '2012'
GROUP BY
    YEARWEEK(website_sessions.created_at)
;
~~~~

*Output:-* Weekly data

| week_start_date | sessions | orders |
|-----------------|----------|--------|
| 2012-03-19      |      861 |     25 |
| 2012-03-25      |      974 |     34 |
| 2012-04-01      |     1188 |     28 |
| 2012-04-08      |     1034 |     29 |
| 2012-04-15      |      700 |     22 |
| 2012-04-22      |      654 |     19 |
| 2012-04-29      |      771 |     19 |
| 2012-05-06      |      780 |     16 |
| 2012-05-13      |      725 |     24 |
| 2012-05-20      |      954 |     25 |
| 2012-05-27      |      875 |     33 |
| 2012-06-03      |      920 |     33 |
| 2012-06-10      |      995 |     31 |
| 2012-06-17      |      963 |     35 |
| 2012-06-24      |      882 |     31 |
| 2012-07-01      |      905 |     31 |
| 2012-07-08      |      916 |     36 |
| 2012-07-15      |      986 |     47 |
| 2012-07-22      |      951 |     41 |
| 2012-07-29      |     1171 |     56 |
| 2012-08-05      |     1223 |     47 |
| 2012-08-12      |     1182 |     40 |
| 2012-08-19      |     1518 |     55 |
| 2012-08-26      |     1590 |     49 |
| 2012-09-02      |     1408 |     55 |
| 2012-09-09      |     1493 |     73 |
| 2012-09-16      |     1772 |     76 |
| 2012-09-23      |     1623 |     69 |
| 2012-09-30      |     1570 |     67 |
| 2012-10-07      |     1617 |     77 |
| 2012-10-14      |     1939 |     92 |
| 2012-10-21      |     2048 |     96 |
| 2012-10-28      |     1914 |     81 |
| 2012-11-04      |     2090 |     89 |
| 2012-11-11      |     1968 |    101 |
| 2012-11-18      |     5126 |    223 |
| 2012-11-25      |     4193 |    179 |
| 2012-12-02      |     2695 |    143 |
| 2012-12-09      |     2481 |    128 |
| 2012-12-16      |     2727 |    133 |
| 2012-12-23      |     1715 |     75 |
| 2012-12-30      |      268 |     18 |

## *Test Skill* - Analyzing Business Pattern

### Task 2 - Analyze the website session volume, by hour of day and by day week. Avoid the holiday time period and use the date range of September 15, 2012 - November 15, 2012.

### Solution: 

~~~~mysql
SELECT
    hr,
    ROUND(AVG(CASE WHEN wkday = 0 THEN sessions ELSE NULL END), 1) AS mon,
    ROUND(AVG(CASE WHEN wkday = 1 THEN sessions ELSE NULL END), 1) AS tue,
    ROUND(AVG(CASE WHEN wkday = 2 THEN sessions ELSE NULL END), 1) AS wed,
    ROUND(AVG(CASE WHEN wkday = 3 THEN sessions ELSE NULL END), 1) AS thru,
    ROUND(AVG(CASE WHEN wkday = 4 THEN sessions ELSE NULL END), 1) AS fri,
    ROUND(AVG(CASE WHEN wkday = 5 THEN sessions ELSE NULL END), 1) AS sat,
    ROUND(AVG(CASE WHEN wkday = 6 THEN sessions ELSE NULL END), 1) AS sun
FROM (
    -- This subquery will return sessions by date-hour-weekday attrition
    SELECT 
        DATE(created_at) AS created_at,
        HOUR(created_at) AS hr,
        WEEKDAY(created_at) AS wkday,
        COUNT(DISTINCT(website_session_id)) AS sessions
    FROM website_sessions
    WHERE created_at > '2012-09-15'
        AND created_at < '2012-11-15'
    GROUP BY 1, 2, 3
) AS hr_wk_sessions
GROUP BY 1
ORDER BY 1 ASC
;
~~~~

| hr   | mon  | tue  | wed  | thru | fri  | sat  | sun  |
|------|------|------|------|------|------|------|------|
|    0 |  7.3 | 17.9 | 25.2 | 21.8 | 18.0 | 21.3 |  8.4 |
|    1 |  6.6 | 19.8 | 18.8 | 25.6 | 21.1 | 19.7 |  7.9 |
|    2 |  7.7 | 21.0 | 19.0 | 24.8 | 20.0 | 19.4 |  7.6 |
|    3 |  6.3 | 19.3 | 20.2 | 20.6 | 21.0 | 17.3 |  5.8 |
|    4 |  7.4 | 17.4 | 15.8 | 17.6 | 17.0 | 12.0 |  6.0 |
|    5 |  6.8 | 12.7 | 15.0 | 13.5 | 13.8 | 12.0 |  7.2 |
|    6 |  7.6 | 11.1 | 13.8 | 13.9 | 10.8 | 10.7 |  6.2 |
|    7 |  8.3 | 10.3 | 12.0 | 12.3 | 10.6 |  9.7 |  4.6 |
|    8 | 11.0 | 10.6 | 12.0 | 10.3 | 10.0 |  6.8 |  6.2 |
|    9 |  9.1 |  8.1 |  8.6 | 10.1 | 11.5 |  6.1 |  5.2 |
|   10 |  8.9 |  8.6 |  8.0 |  8.0 | 10.0 |  6.0 |  5.0 |
|   11 |  7.2 |  7.9 |  5.9 |  6.5 |  6.8 |  4.7 |  4.0 |
|   12 |  5.4 |  4.6 |  3.9 |  6.3 |  5.4 |  5.0 |  3.0 |
|   13 |  6.1 |  4.8 |  4.6 |  5.3 |  3.8 |  3.4 |  3.0 |
|   14 |  6.3 |  4.1 |  6.2 |  3.6 |  5.0 |  3.8 |  2.6 |
|   15 |  5.0 |  6.3 |  5.1 |  5.0 |  5.3 |  3.3 |  3.7 |
|   16 |  5.0 |  6.1 |  4.8 |  5.3 |  5.5 |  4.2 |  3.1 |
|   17 |  6.6 |  6.9 |  6.9 | 10.0 |  6.9 |  4.4 |  4.0 |
|   18 | 10.2 | 10.1 |  9.2 | 11.5 |  8.0 |  4.9 |  4.3 |
|   19 | 14.7 | 13.1 | 16.0 | 19.1 | 14.8 |  5.9 |  4.8 |
|   20 | 16.9 | 17.7 | 20.2 | 17.5 | 19.3 |  8.2 |  5.3 |
|   21 | 19.7 | 16.8 | 23.6 | 21.4 | 17.6 |  7.1 |  8.0 |
|   22 | 19.0 | 22.1 | 24.1 | 21.6 | 22.0 |  8.2 |  6.0 |
|   23 | 19.7 | 20.7 | 19.9 | 24.6 | 19.3 |  8.1 |  8.1 |
