# Music-Store-Data-Analysis-Project


-- Question 1 :- Who is the senior most employee based on job title.
select * from employee order by levels desc limit 1;

-- Question 2 :- Which countries have the most invoices.
select billing_country,count(invoice_id) as Number_of_Invoices from invoice 
group by billing_country 
order by Number_of_Invoices 
desc limit 1;

-- Question 3:- What are top three values of total invoice.
select total from invoice
order by total 
desc limit 3;

-- Question 4:- Which city has the best customers? we would like to throw a promotional music festival in the city we made the most money. write a query that returns one city that has the highest sum of invoices totals. 
select billing_city,sum(total) as Total_Invoices from invoice
group by billing_city 
order by Total_invoices 
desc limit 1;

-- Question 5:- Who is the best customer? The customer who has spent the most money will be declared the best customer. Write a query that returns the person who has spent the most money.
select c.customer_id,c.first_name,c.last_name,sum(i.total) as Total from customer c
join invoice i on c.customer_id=i.customer_id
group by c.customer_id
order by Total 
desc limit 1;

-- Question 6:- Write a query to return email,first name,last name, genre of all Rock music listerners. Return your list ordered by alphabetically by email starting with A.
select  DISTINCT email,first_name,last_name
from customer
join invoice on customer.customer_id=invoice.customer_id
join invoice_line on invoice.invoice_id=invoice_line.invoice_id
where track_id in (
	select track_id from track
	join genre on track.genre_id=genre.genre_id
	where genre.name like 'Rock'
)
order by email;

-- Question 7:- Let's invite the artist who have written the most rock music in our dataset. Write a query that returns the name and total track count of the top 10 rock bands.
select artist.artist_id,artist.name,count(artist.artist_id) as Total_no_of_songs
from track
join album on track.album_id=album.album_id
join artist on artist.artist_id=album.artist_id
join genre on genre.genre_id=track.genre_id
where genre.name like 'Rock'
group by artist.artist_id
order by Total_no_of_songs
desc limit 10;

-- Question 8:- Return all the track names that have a song length longer than average song length. Return Name and milliseconds for each track. Order by song length with longest song at the first.
select Name,Milliseconds from track 
where Milliseconds > 
(select avg(milliseconds) as Avg_Track_Record from track)
order by Milliseconds desc;

-- Question 9:- Find how much amount spent by each customer on artists. Write a query to return customer name,artist name and total spent.

with best_selling_artist as (
select artist.artist_id as artist_id,artist.name as Artist_Name,
sum(invoice_line.unit_price*invoice_line.quantity) as Total_Sales
from invoice_line
join track on invoice_line.track_id=track.track_id
join album on track.album_id=album.album_id
join artist on album.artist_id=artist.artist_id
group by 1
order by 3 DESC
limit 1
)

select c.customer_id,c.first_name,c.last_name,bsa.artist_name,
sum(il.unit_price*il.quantity) as Amount_Spent
from invoice i
join customer c on i.customer_id=c.customer_id
join invoice_line il on i.invoice_id=il.invoice_id
join track t on t.track_id=il.track_id
join album alb on alb.album_id=t.album_id
join best_selling _artist bsa on bsa=alb.artist_id
group by 1,2,3,4
order by 5 desc;


-- Question 10:- We want to find out the most popular music genre for each country. We determine the most poopular genre as the genre with the highest amount of purchases.
--              Write a query that returns each country along with the top genre. For countries where the maximum number of purchases is shared return all genre.

with recursive sales_per_country as (
select count(*) as purchases_per_genre,customer.country,genre.name,genre.genre_id
from invoice_line
join invoice on invoice.invoice_id=invoice_line.invoice_id
join customer on customer.customer_id=invoice.customer_id
join track on track.track_id=invoice_line.track_id
join genre on genre.genre_id=track.genre_id
group by 2,3,4
order by 2
),
max_genre_per_country as (
select max(purchases_per_genre) as max_genre_number,country 
from sales_per_country
group by 2
order by 2 desc
)

select sales_per_country.*
from sales_per_country
join max_genre_per_country on sales_per_country.country=max_genre_per_country.country
where sales_per_country.purchases_per_genre=max_genre_per_country.max_genre_number
order by purchases_per_genre desc;


-- Question 11:- Write a query that determines the customer that has spent the most on the music for each country.
--               Write a query that returns the country along with top customer na dhow much he has spent.
--		 For countries where the top spent is shared, provide all customers who spent this amount.


with recursive customer_with_country as (
select customer.customer_id,first_name,last_name,billing_country,sum(total) as total_spending
from invoice
join customer on customer.customer_id=invoice.customer_id
group by 1,2,3,4
order by 1,5 desc
),
country_max_spending as (
select billing_country,max(total_spending) as max_spending
from customer_with_country
group by billing_country
)

select cc.billing_country,cc.total_spending,cc.first_name,cc.last_name,cc.customer_id
from customer_with_country cc
join country_max_spending cs on cc.billing_country=cs.billing_country
where cc.total_spending=cs.max_spending
order by 1;

