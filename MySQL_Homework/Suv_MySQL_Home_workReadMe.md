-- MY SQL Homework
-- Suvrangshu Ghosh
-- NOTES:
-- Please use Sakila database schema 
-- ----------------------------------------------------------------------
-- Modification History :
--
--
-- 
-- ----------------------------------------------------------------------
use sakila;


-- 1a. Display the first and last names of all actors from the table actor.
select first_name, last_name from actor;

-- 1b. Display the first and last name of each actor in a single column in upper case letters. 
-- Name the column Actor Name.

select upper(concat(first_name, ' ', last_name)) from actor;

-- 2a. You need to find the ID number, first name, and last name of an actor,
-- of whom you know only the first name, "Joe." What is one query would you use to obtain this information?

select actor_ID, first_name , last_name from actor
     where first_name = 'Joe';

-- 2b. Find all actors whose last name contain the letters GEN:
select * from actor 
	where last_name like '%GEN%';

-- 2c. Find all actors whose last names contain the letters LI. 
-- This time, order the rows by last name and first name, in that order:
select last_name, first_name from actor 
	where last_name like '%LI%';

-- 2d. Using IN, display the country_id and country columns of the following countries: 
-- Afghanistan, Bangladesh, and China:

select country_id, country from country
	where country in ('Afghanistan', 'Bangladesh', 'CHina');

-- 3a. You want to keep a description of each actor. 
-- You don't think you will be performing queries on a description, 
-- so create a column in the table actor named description and 
-- use the data type BLOB (Make sure to research the type BLOB, 
-- as the difference between it and VARCHAR are significant).

ALTER TABLE actor
ADD COLUMN description BLOB AFTER last_name;


-- 3b. Very quickly you realize that entering descriptions for each actor is too much effort. 
-- Delete the description column.
ALTER TABLE actor
	drop column description;

-- 4a. List the last names of actors, as well as how many actors have that last name.
select last_name, count(*) from actor 
	group by last_name;

-- 4b. List last names of actors and the number of actors who have that last name, 
-- but only for names that are shared by at least two actors
select last_name, count(*) from actor 
	group by last_name
    having count(*) > 1;

-- 4c. The actor HARPO WILLIAMS was accidentally entered in the actor table as GROUCHO WILLIAMS. 
-- Write a query to fix the record.

-- first looking for row with first_name = 'GROUCHO' and last_name = 'WILLIAMS'

select * from actor
	where last_name = 'WILLIAMS';
-- execute the update command
UPDATE actor 
	SET first_name = 'HARPO' 
    where first_name = 'GROUCHO' and last_name = 'WILLIAMS';

-- 4d. Perhaps we were too hasty in changing GROUCHO to HARPO. 
-- It turns out that GROUCHO was the correct name after all! 
-- In a single query, if the first name of the actor is currently HARPO, change it to GROUCHO.

UPDATE actor 
	SET first_name = 'GROUCHO' 
    where first_name = 'HARPO' and last_name = 'WILLIAMS';


-- 5a. You cannot locate the schema of the address table. Which query would you use to re-create it?

show create table address;

-- checking data in staff table

select * from staff; 

-- 6a. Use JOIN to display the first and last names, 
-- as well as the address, of each staff member. Use the tables staff and address:
select 
	first_name, last_name, address, address2, district, city,postal_code
from 
    staff S
inner join 
    address A 
on 
    S.address_id = A.address_id
inner join city C
on 
	A.city_id = C.city_id;

-- 6b. Use JOIN to display the total amount rung up by each staff member in August of 2005. 
-- Use tables staff and payment.

   
select 
	S1.staff_id,
    S1.first_name,
    S1.last_name,
    month(payment_date) as 'Month',
    year(payment_date) as 'Year',
    sum(amount)
from payment P1

INNER JOIN 
	staff S1
on 
	P1.staff_id = S1.staff_id
where
	month(payment_date) = '8'and year(payment_date) = '2005'
group by 
	P1.staff_id order by amount desc;

    
-- 6c. List each film and the number of actors who are listed for that film. 
-- Use tables film_actor and film. Use inner join.
        
select  FA.film_id,F.title, count(*) as 'Total Actors' 
from film_actor FA
inner join 
	film F
on 
	FA.film_id = F.film_id
group by  film_id;

-- 6d. How many copies of the film Hunchback Impossible exist in the inventory system?

select title, count(*) as 'Inventory Total'
from inventory i
inner join
	film f
on
	f.film_id = i.film_id
where
	f.title = 'Hunchback Impossible'
group by i.film_id;

-- 6e. Using the tables payment and customer and the JOIN command, 
-- list the total paid by each customer. List the customers alphabetically by last name:
    
select 
	#C1.customer_id,
    C1.first_name,
    C1.last_name,
    sum(amount) as 'Total Amount Paid'
from 
	payment P1
INNER JOIN 
	customer C1
on 
	P1.customer_id = C1.customer_id
group by 
	P1.customer_id 
order by 
   C1.last_name;
 	
-- 7a. The music of Queen and Kris Kristofferson have seen an unlikely resurgence. 
-- As an unintended consequence, films starting with the letters K and Q have also soared in popularity. 
-- Use subqueries to display the titles of movies starting with the letters K and Q whose language is English.


select title, description
from 
	film
where 
	title like 'K%' or title like 'Q%'
and 
language_id = 
(
	select language_id
    from language
    where 
    upper(name) = 'ENGLISH'
);


-- 7b. Use subqueries to display all actors who appear in the film Alone Trip.
select first_name, last_name
from 
	actor
where
	actor_id in
(
	select actor_id
    from film_actor
    where
    film_id in 
       (
		select film_id from film 
		where title = 'Alone Trip'
        )
);


-- 7c. You want to run an email marketing campaign in Canada, 
-- for which you will need the names and email addresses of all Canadian customers. 
-- Use joins to retrieve this information


select first_name, last_name, email, A1.address, C2.city, C3.country
from 
	customer C1
inner join 
	address A1
on
  A1.address_id = C1.address_id
inner join 
	city C2
on 
 C2.city_id = A1.city_id
 inner join 
	country C3
on 
	C3.country_id = C2.country_id
where 
	upper(C3.country) = "CANADA";

-- 7d. Sales have been lagging among young families, 
-- and you wish to target all family movies for a promotion. 
-- Identify all movies categorized as family films.

 
 select F1.film_id, title, description, rating, C1.name as 'Category'
 from 
	film F1
inner join 
	film_category FC
on
	(F1.film_id = FC.film_id)
inner join
	category C1
on 
	C1.category_id = FC.category_id and C1.name = 'Family';

    
-- 7e. Display the most frequently rented movies in descending order.
-- NOTE : Using Limit 6000 to see all records 

-- --------------------------------------------
    
select title, count(*) as Rental_count
from 
	rental R
join 
	inventory I 
on
	R.inventory_id = I.inventory_id 
join 
	film F 
on
	I.film_id = f.film_id
group by 
    f.title order by Rental_count  desc limit 6000;
	    
-- 7f. Write a query to display how much business, in dollars, each store brought in.

select S.store_id, concat('$', format(sum(amount),2)) as "Total Sales $" 
from
	payment P 
join 
	staff S 
on
	P.staff_id = S.staff_id
join
	store ST 
on 
	S.store_id = ST.store_id
group by 
	P.staff_id;

-- 7g. Write a query to display for each store its store ID, city, and country.

select ST.store_id, city, country

from
	store ST 
join
	address A 
on 
	ST.address_id = A.address_id
join
	city C
on
	A.city_id = C.city_id
join
	country CNTRY
on
	C.country_id = CNTRY.country_id;
    
-- 7h. List the top five genres in gross revenue in descending order. 
-- (Hint: you may need to use the following tables: category, film_category, inventory, payment, and rental.)

-- concat('$', format(sum(price), 2)) - formating to Dollar sign 

select cat.name, concat('$', format(sum(amount),2)) as Gross_revenu 

from 
	payment P 
join 
	rental R 
on
	P.rental_id = R.rental_id
join
	inventory I 
on 
	R.inventory_id = I.inventory_id
join
	film_category FC
on 
	I.film_id = FC.film_id
join 
	category CAT
on
	FC.category_id = CAT.category_id
group by CAT.name order by Gross_revenu desc limit 5 offset 0;


-- 8a. In your new role as an executive, you would like to have an easy way of viewing the 
-- Top five genres by gross revenue. Use the solution from the problem above to create a view. 
-- If you haven't solved 7h, you can substitute another query to create a view.

CREATE VIEW `Top_5_Gen_by_grossView` 
AS
select cat.name, concat('$', format(sum(amount),2)) as Gross_revenu 
from 
	payment P 
join 
	rental R 
on
	P.rental_id = R.rental_id
join
	inventory I 
on 
	R.inventory_id = I.inventory_id
join
	film_category FC
on 
	I.film_id = FC.film_id
join 
	category CAT
on
	FC.category_id = CAT.category_id
group by CAT.name order by Gross_revenu desc limit 5 offset 0;


-- running the view 
select * from Top_5_Gen_by_grossView;

-- 8b. How would you display the view that you created in 8a?

desc Top_5_Gen_by_grossView;

SHOW CREATE VIEW Top_5_Gen_by_grossView;

select view_definition 
from information_schema.views
where table_name = 'Top_5_Gen_by_grossView';

-- 8c. You find that you no longer need the view top_five_genres. Write a query to delete it.

DROP VIEW Top_5_Gen_by_grossView;

-- -----------------------------------------------------------
