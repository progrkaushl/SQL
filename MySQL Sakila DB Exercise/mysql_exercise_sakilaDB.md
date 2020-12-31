# MYSQL - EXERCISE
### Submitted By - KAUSHLENDRA


Database used in this exercise is [SAKILA](https://dev.mysql.com/doc/sakila/en/) sample database.

-----------------------------------------------

## 1. SELECT STATEMENT

#### 1a. Select all columns from the actor table.
SOLUTION:
~~~sql
SELECT * FROM actor;
~~~

#### 1b. Select only the last_name column from the actor table.
SOLUTION:
~~~sql
SELECT `last_name` FROM actor;
~~~

#### 1c. Select only the following columns from the film table.
| COLUMN NAME 		|  Note		                    |
|-------------------|-------------------------------|
| title				| Exists in film table          |
| description		| Exists in film table          |
| rental_duration	| Exists in film table          |
| rental_rate		| Exists in film table          |
| total_rental_cost	| rental_duration * rental_rate |
SOLUTION:
~~~sql
SELECT `title`,
	`description`,
	`rental_duration`,
	`rental_rate`,
	(`rental_duration` * `rental_rate`) AS "total_rental_cost"
FROM film;
~~~


-----------------------------------------------

## 2. DISTINCT OPERATOR

#### 2a. Select all distinct (different) last names from the actor table.
SOLUTION:
~~~sql
SELECT DISTINCT `last_name`  FROM actor;
~~~

#### 2b. Select all distinct (different) postal codes from the address table.
SOLUTION:
~~~sql
SELECT DISTINCT `postal_code` FROM address;
~~~

#### 2c. Select all distinct (different) ratings from the film table.
SOLUTION:
~~~sql
SELECT DISTINCT `rating` FROM film;
~~~


-----------------------------------------------

## 3. WHERE CLAUSE

#### 3a. Select the title, description, rating, movie length columns from the films table that last 3 hours or longer.
SOLUTION:
~~~sql
SELECT `title`, `description`, `rating`, `length` AS "movie length" 
FROM film 
WHERE `length` >= 180;
~~~

#### 3b. Select the payment id, amount, and payment date columns from the payments table for payments made on or after 05/27/2005.
SOLUTION:
~~~sql
SELECT `payment_id`, `amount`, `payment_date` 
FROM payment 
WHERE `payment_date` >= "2005-05-27 00:00:00";
~~~


#### 3c. Select the primary key, amount, and payment date columns from the payment table for payments made on 05/27/2005.
SOLUTION:
~~~sql
/*
SELECT `payment_id`, `amount`, `payment_date` FROM payment WHERE `payment_date` = "2005-05-27";
This query will be wrong as payment_date is a datetime and there is no exact record of "2005-05-27"
*/

/* BETWEEN, LIKE and DATE methods can be used in this case, DATE method is more useful as it parse date from datetime column and match it with condition */
SELECT `payment_id`, `amount`, `payment_date` FROM payment WHERE `payment_date` BETWEEN "2005-05-27 00:00:00" AND "2005-05-28 00:00:00";
SELECT `payment_id`, `amount`, `payment_date` FROM payment WHERE `payment_date` LIKE '2005-05-27%';
SELECT `payment_id`, `amount`, `payment_date` FROM payment WHERE DATE(`payment_date`) = "2005-05-27";
~~~


#### 3d. Select all columns from the customer table for rows that have a last names beginning with S and a first names ending with N.
SOLUTION:
~~~sql
SELECT * 
FROM customer 
WHERE `last_name` LIKE 's%' AND `first_name` LIKE '%n';
~~~


#### 3e. Select all columns from the customer table for rows where the customer is inactive or has a last name beginning with "M".
SOLUTION:
~~~sql
SELECT * 
FROM customer 
WHERE `active` = "false"  OR `last_name` LIKE 'M%';
~~~


#### 3f. Select all columns from the category table for rows where the primary key is greater than 4 and the name field begins with either C, S or T.
SOLUTION:
~~~sql
SELECT * 
FROM category 
WHERE `category_id` > 4 AND 
	`name` LIKE 'C%' OR `name` LIKE 'S%' OR `name` LIKE 'T%';
~~~


#### 3g. Select all columns minus the password column from the staff table for rows that contain a password.
SOLUTION:
~~~sql
SELECT `store_id`, `first_name`, `last_name`, `address_id`, `picture`, `email`, `store_id`, `active`, `username`, `last_update` 
FROM staff 
WHERE `password` IS NOT NULL;
~~~


#### 3h. Select all columns minus the password column from the staff table for rows that do not contain a password.
SOLUTION:
~~~sql
SELECT `store_id`, `first_name`, `last_name`, `address_id`, `picture`, `email`, `store_id`, `active`, `username`, `last_update` 
FROM staff 
WHERE `password` IS NULL;
~~~



-----------------------------------------------

## 4. IN OPERATOR

#### 4a. Select the phone and district columns from the address table for addresses in California, England, Taipei, or West Java.
SOLUTION:
~~~sql
SELECT `phone`, `district` 
FROM address 
WHERE `district` IN ("California", "England", "Taipei", "West Java") 
ORDER BY `district`;
~~~

#### 4b. Select the payment id, amount, and payment date columns from the payment table for payments made on 05/25/2005, 05/27/2005, and 05/29/2005.
Use the IN operator and the DATE function, instead of the AND operator as in previous exercises.
SOLUTION:
~~~sql
SELECT `payment_id`, `amount`, `payment_date` 
FROM payment 
WHERE DATE(`payment_date`) IN ("2005-05-25","2005-05-27","2005-05-29") 
ORDER BY `payment_date`;
~~~
#### 4c. Select all columns from the film table for films rated G, PG-13 or NC-17.
SOLUTION:
~~~sql
/* There are two methods to do this, either use IN an select ratings required or use NOT IN and exclude ratings not required. */

/* Using IN Operator */
SELECT * FROM film WHERE `rating` IN ("G","PG-13","NC-17") ORDER BY `title`;

/* Using NOT IN Operator */
SELECT * FROM film WHERE `rating` NOT IN ("PG","R") ORDER BY `title`;
~~~

---------------------------------------------------------


## 5. BETWEEN OPERATOR

#### 5a. Select all columns from the payment table for payments made between midnight 05/25/2005 and 1 second before midnight 05/26/2005.
SOLUTION:
~~~sql
/* There are two methods for solving this, either use exact datetime or just use date with BETWEEN clause. */

/* Using DATE */
SELECT * 
FROM payment 
WHERE `payment_date` BETWEEN "2005-05-25" AND "2005-05-26" 
ORDER BY `payment_date`;

/* Using DATETIME */
SELECT * 
FROM payment 
WHERE `payment_date` BETWEEN "2005-05-25 00:00:00" AND "2005-05-25 23:59:59" 
ORDER BY `payment_date`;
~~~

#### 5b. Select the following columns from the film table for films where the length of the description is between 100 and 120.
|COLUMN NAME		|Note							|
|-------------------|-------------------------------|
|title				|Exists in film table.			|
|description		|Exists in film table.			|
|release_year		|Exists in film table.			|
|total_rental_cost	|rental_duration * rental_rate	|
SOLUTION:
~~~sql
SELECT `title`, `description`, `release_year`, (`rental_duration` * `rental_rate`) AS "total_rental_cost" 
FROM film 
WHERE `length` BETWEEN 100 AND 120 
ORDER BY `title`;
~~~


---------------------------------------------------------


## 6. LIKE OPERATOR

#### 6a. Select the following columns from the film table for rows where the description begins with "A Thoughtful".
###### Title, Description, Release Year
SOLUTION:
~~~sql
SELECT `title`, `description`, `release_year` 
FROM film 
WHERE `description` LIKE "A Thoughtful%" 
ORDER BY `title`;
~~~


#### 6b. Select the following columns from the film table for rows where the description ends with the word "Boat".
###### Title, Description, Rental Duration
SOLUTION:
~~~sql
SELECT `title`, `description`, `rental_duration` 
FROM film 
WHERE `description` LIKE '%Boat' 
ORDER BY `title`;
~~~


#### 6c. Select the following columns from the film table where the description contains the word "Database" and the length of the film is greater than 3 hours.
###### Title, Length, Description, Rental Rate
SOLUTION:
~~~sql
SELECT `title`, `length`, `description`, `rental_rate`
FROM film 
WHERE `description` LIKE '%database%' AND `length` > 180
ORDER BY `title`;
~~~


---------------------------------------------------------


## 7. LIMIT OPERATOR

#### 7a. Select all columns from the payment table and only include the first 20 rows.
SOLUTION:
~~~sql
SELECT * FROM payment LIMIT 20;
~~~

#### 7b. Select the payment date and amount columns from the payment table for rows where the payment amount is greater than 5, and only select rows whose zero-based index in the result set is between 1000-2000.
SOLUTION:
~~~sql
SELECT `payment_date`, `amount` 
FROM payment
WHERE `amount` > 5
LIMIT 1000 OFFSET 2000;
~~~

#### 7c. Select all columns from the customer table, limiting results to those where the zero-based index is between 101-200.
SOLUTION:
~~~sql
SELECT * FROM customer LIMIT 100 OFFSET 200; 
~~~


---------------------------------------------------------


## 8. ORDER BY STATEMENT

#### 8a. Select all columns from the film table and order rows by the length field in ascending order.
SOLUTION:
~~~sql
SELECT *
FROM film
ORDER BY `length` ASC;
~~~

#### 8b. Select all distinct ratings from the film table ordered by rating in descending order.
SOLUTION:
~~~sql
SELECT DISTINCT `rating`
FROM film
ORDER BY `rating` DESC;
~~~

#### 8c. Select the payment date and amount columns from the payment table for the first 20 payments ordered by payment amount in descending order.
SOLUTION:
~~~sql
SELECT `payment_date`, `amount`
FROM payment
ORDER BY `payment`
LIMIT 20;
~~~

#### 8d. Select the title, description, special features, length, and rental duration columns from the film table for the first 10 films with behind the scenes footage under 2 hours in length and a rental duration between 5 and 7 days, ordered by length in descending order.
SOLUTION:
~~~sql
SELECT `title`, `description`, `special_features`, `length`, `rental_duration`
FROM film
WHERE `special_features` = "Behind the Scenes" AND 
	`length` < 160 AND 
	`rental_duration` BETWEEN 5 AND 7
ORDER BY `length` DESC
LIMIT 10;
~~~


---------------------------------------------------------


## 9. JOINS

#### 9a. Select customer first_name/last_name and actor first_name/last_name columns from performing a /left join/  between the customer and actor column on the last_name column in each table. 
(i.e. `customer.last_name = actor.last_name`)
Label customer first_name/last_name columns as customer_first_name/customer_last_name
Label actor first_name/last_name columns in a similar fashion.
###### Returns correct number of records: 599
SOLUTION:
~~~sql
SELECT customer.first_name AS "customer_first_name",
	customer.last_name AS "customer_last_name",
	actor.first_name AS "actor_first_name",
	actor.last_name AS "actor_last_name"
FROM customer
LEFT JOIN actor ON 
	customer.first_name = actor.first_name AND 
	customer.last_name = actor.last_name;
~~~


#### 9b. Select the customer first_name/last_name and actor first_name/last_name columns from performing a /right join between the customer and actor column on the last_name column in each table. 
(i.e. `customer.last_name = actor.last_name`)
###### Returns correct number of records: 200
SOLUTION:
~~~sql
SELECT customer.first_name AS "customer_first_name",
	customer.last_name AS "customer_last_name",
	actor.first_name AS "actor_first_name",
	actor.last_name AS "actor_last_name"
FROM customer
RIGHT JOIN actor ON 
	customer.first_name = actor.first_name AND 
	customer.last_name = actor.last_name;
~~~


### 9c. Select the customer first_name/last_name and actor first_name/last_name columns from performing an inner join between the customer and actor column on the last_name column in each table. 
(i.e. `customer.last_name = actor.last_name`)
###### Returns correct number of records: 43
SOLUTION:
~~~sql
SELECT customer.first_name AS "customer_first_name",
	customer.last_name AS "customer_last_name",
	actor.first_name AS "actor_first_name",
	actor.last_name AS "actor_last_name"
FROM customer
INNER JOIN actor ON 
	customer.first_name = actor.first_name AND 
	customer.last_name = actor.last_name;
~~~

### 9d. Select the city name and country name columns from the city table, performing a left join with the country table to get the country name column.
###### Returns correct records: 600 
SOLUTION:
~~~sql
SELECT city.city AS "city", country.country AS "country"
FROM city
LEFT JOIN country ON
	city.country_id = country.country_id
ORDER BY country;
~~~


### 9e. Select the title, description, release year, and language name columns from the film table, performing a left join with the language table to get the "language" column.
Label the language.name column as "language" (e.g. `select language.name as language`)
###### Returns correct number of rows: 1000
SOLUTION:
~~~sql
SELECT film.title AS "title", 
	film.description AS "description", 
	film.release_year AS "release year", 
	language.name AS "langauge"
FROM film
LEFT JOIN language ON
	film.language_id = language.language_id	
ORDER BY langauge;
~~~

### 9f. Select the first_name, last_name, address, address2, city name, district, and postal code columns from the staff table, performing 2 left joins with the address table then the city table to get the address and city related columns.
###### Returns correct number of rows: 2
SOLUTION:
~~~sql
SELECT staff.first_name AS "first name",
	staff.last_name AS "last name",
	address.address AS "primary address",
	address.address2 AS "secondary address",
	address.district AS "district",
	address.postal_code AS "postal code",
	city.city AS "city"
FROM staff
LEFT JOIN address ON 
	staff.address_id = address.address_id
LEFT JOIN city ON 
	address.city_id = city.city_id
;
~~~

## END OF EXERCISE
# THANK YOU