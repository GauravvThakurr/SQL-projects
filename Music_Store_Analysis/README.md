# Music Store SQL Analysis

## Database and Tools
* Postgre SQL
* PgAdmin4
  
## Schema diagram
![Er diagram music](https://github.com/GauravvThakurr/SQL_projects/assets/141028751/fbcdfbd9-a103-4316-b51d-7d6aff279743)


````sql
Q1: Who is the senior most employee based on job title?

select first_name,last_name, levels from employee
order by levels desc
limit 1;

Q2: Which countries have the most Invoices?

select billing_country, count(*) as count_of_invoice from invoice
group by billing_country
order by count(*) desc
limit 1;

Q3: What are top 3 values of total invoice
 
 select total from invoice
 order by total desc
 limit 3;
 
 Q4: Which city has the best customers? We would like to throw a
promotional Music Festival in the city we made the most money. Write a
query that returns one city that has the highest sum of invoice totals.
Return both the city name & sum of all invoice totals

select billing_city, sum(total) as invoice_total from invoice
group by billing_city
order by  sum(total) desc
limit 1;

Q5: Who is the best customer? The customer who has spent the most
money will be declared the best customer. Write a query that return
the person who has spent the most money.

select c.customer_id, c.first_name, c.last_name, sum(i.total) as total from customer c
join invoice i on  c.customer_id = i.customer_id
group by  c.customer_id
order by total desc
limit 1;

Q6 : Write query to return the email, first name, last name, & Genre
of all Rock Music listeners. Return your list ordered alphabetically
by email starting with A

SELECT distinct c.email, c.first_name, c.last_name
FROM customer c
join invoice i on i.customer_id = c.customer_id
join invoice_line li on li.invoice_id = i.invoice_id 
where li.track_id in ( 
		select t.track_id from track t
		join genre g on g.genre_id = t.genre_id
		where g.name = 'Rock' 
)
order by c.email 

Q7: Lets invite the artists who have written the most rock music in
our dataset. Write a query that returns the Artist name and total
track count of the top 10 rock bands

SELECT a.artist_id, a.name, COUNT(*) AS number_of_songs
FROM track t
JOIN album al ON al.album_id = t.album_id
JOIN artist a ON a.artist_id = al.artist_id
JOIN genre g ON g.genre_id = t.genre_id
WHERE g.name = 'Rock'
GROUP BY a.artist_id, a.name
ORDER BY number_of_songs DESC
LIMIT 10;

Q8: Return all the track names that have a song length longer than
the average song length. Return the Name and Milliseconds for
each track. Order by the song length with the longest songs listed
first.

select name, milliseconds from track 
where milliseconds > (select avg(milliseconds) from track)
order by milliseconds desc

Q9: Find how much amount spent by each customer on best selling artists? Write a
query to return customer name, artist name and total spent

WITH cte AS (
	SELECT a.artist_id AS artist_id, a.name AS artist_name, 
	SUM(il.unit_price * il.quantity) AS total_sales
	FROM invoice_line il
	JOIN track t ON t.track_id = il.track_id
	JOIN album al ON al.album_id = t.album_id
	JOIN artist a ON a.artist_id = al.artist_id
	GROUP BY a.artist_id
	ORDER BY total_sales DESC
	LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, cte.artist_name, 
SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album al on al.album_id = t.album_id
JOIN cte ON cte.artist_id = al.artist_id
GROUP BY  c.customer_id, c.first_name, c.last_name, cte.artist_name
ORDER BY amount_spent DESC;

Q10: We want to find out the most popular music Genre for each country.
( We determine the most popular genre as the genre with the highest
amount of purchases. Write a query that returns each country along with
the top Genre. For countries where the maximum number of purchases
is shared return all Genres.


WITH cte AS 
(
    SELECT c.country, g.name, COUNT(*) AS purchases, 
	ROW_NUMBER() OVER(PARTITION BY c.country ORDER BY COUNT(*) DESC) AS RN
    FROM invoice_line il
	JOIN invoice i ON i.invoice_id = il.invoice_id
	JOIN customer c ON c.customer_id = i.customer_id
	JOIN track t ON t.track_id = il.track_id
	JOIN genre g ON g.genre_id = t.genre_id
	GROUP BY c.country, g.name
	ORDER BY c.country,  COUNT(*) DESC
)
SELECT * FROM cte WHERE RN = 1


 
Q11: Write a query that determines the customer that has spent the most
on music for each country. Write a query that returns the country along
with the top customer and how much they spent. For countries where
the top amount spent is shared, provide all customers who spent this
amount
 
 WITH cte AS (
		SELECT c.customer_id, c.first_name,c.last_name,i.billing_country,SUM(i.total) AS total_spending,
	    ROW_NUMBER() OVER(PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RN
		FROM invoice i
		JOIN customer c  ON c.customer_id = i.customer_id
		GROUP BY c.customer_id, c.first_name,c.last_name,i.billing_country
		ORDER BY total_spending DESC )
 
SELECT * FROM cte WHERE RN = 1
 
````