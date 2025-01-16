# Music Store Data Analysis Project
=====================================

## Project Overview

The Music Store Data Analysis Project is designed to leverage SQL to analyze the performance and customer interactions of a fictional music store. This project employs a structured approach to querying data from the store's database, allowing for insights into various aspects such as employee hierarchy, invoice generation, customer spending patterns, and genre popularity across different countries.

This project consists of three main question sets categorized by difficulty (Easy, Moderate, and Advanced), each focusing on specific analyses to extract meaningful information from the music store dataset. The dataset includes various tables such as `employee`, `customer`, `invoice`, `track`, `album`, and `genre`, allowing for a rich set of queries.

## Key Objectives

1. **Employee Analysis**: Identify the senior most employees based on their job titles to understand the corporate hierarchy.
  
2. **Sales Insights**: Analyze which countries generate the most sales and identify the top invoicing cities to target them for promotional events.
  
3. **Customer Engagement**: Discover the best customers based on total spending and analyze customer data segmented by music genre preferences.
  
4. **Artist and Genre Popularity**: Determine which rock artists are most popular and uncover trends in genre preferences across different geographical regions.

5. **Track Performance**: Analyze the lengths of songs in relation to the average song length to determine standout tracks.

## Table of Contents

### Question Set 1: Easy

1. [Who is the senior most employee based on job title?](#q1-who-is-the-senior-most-employee-based-on-job-title)
2. [Which countries have the most Invoices?](#q2-which-countries-have-the-most-invoices)
3. [What are top 3 values of total invoice?](#q3-what-are-top-3-values-of-total-invoice)
4. [Which city has the best customers?](#q4-which-city-has-the-best-customers)
5. [Who is the best customer?](#q5-who-is-the-best-customer)

### Question Set 2: Moderate

1. [Email and Genre of Rock Music listeners](#q6-write-query-to-return-the-email-first-name-last-name--genre-of-all-rock-music-listeners)
2. [Top 10 Rock Bands based on number of tracks](#q7-lets-invite-the-artists-who-have-written-the-most-rock-music-in-our-dataset)
3. [Tracks longer than average song length](#q8-return-all-the-track-names-that-have-a-song-length-longer-than-the-average-song-length)

### Question Set 3: Advanced

1. [Amount spent by each customer on artists](#q9-find-how-much-amount-spent-by-each-customer-on-artists)
2. [Most popular music genre by country](#q10-we-want-to-find-out-the-most-popular-music-genre-for-each-country)
3. [Top customer spending per country](#q11-write-a-query-that-determines-the-customer-that-has-spent-the-most-on-music-for-each-country)

## SQL Queries

### Question Set 1: Easy

#### q1. Who is the senior most employee based on job title?
```sql
SELECT title, last_name, first_name 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```
* This query retrieves the title and names of the employee who holds the highest position in the hierarchy based on job title levels.

#### q2. Which countries have the most Invoices?
```sql
SELECT COUNT(*) AS c, billing_country 
FROM invoice
GROUP BY billing_country
ORDER BY c DESC;
```
* This query counts all invoices per country to identify where the highest number of transactions are generated.

#### q3. What are top 3 values of total invoice?
```sql
SELECT total 
FROM invoice
ORDER BY total DESC
LIMIT 3;
```
* This retrieves the three highest invoice totals, allowing insights into the largest sales transactions.

#### q4. Which city has the best customers?
```sql
SELECT billing_city, SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```
* This query identifies the city that has generated the most revenue based on invoice totals, which can help in deciding locations for promotional events.

#### q5. Who is the best customer?
```sql
SELECT customer.customer_id, first_name, last_name, SUM(total) AS total_spending
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
GROUP BY customer.customer_id
ORDER BY total_spending DESC
LIMIT 1;
```
* This query identifies the customer who has spent the most money at the store, highlighting the most valuable customer.

### Question Set 2: Moderate

#### q6. Email and Genre of Rock Music listeners
```sql
SELECT DISTINCT email, first_name, last_name
FROM customer
JOIN invoice ON customer.customer_id = invoice.customer_id
JOIN invoiceline ON invoice.invoice_id = invoiceline.invoice_id
WHERE track_id IN (
    SELECT track_id FROM track
    JOIN genre ON track.genre_id = genre.genre_id
    WHERE genre.name LIKE 'Rock'
)
ORDER BY email;
```
* This query retrieves the details of customers who have purchased rock music, providing insight into the demographics of rock listeners.

#### q7. Top 10 Rock Bands based on number of tracks
```sql
SELECT artist.artist_id, artist.name, COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album ON album.album_id = track.album_id
JOIN artist ON artist.artist_id = album.artist_id
JOIN genre ON genre.genre_id = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```
* This query identifies the rock artists who have produced the most tracks in the dataset, allowing for targeted promotions to fans of these artists.

#### q8. Tracks longer than average song length
```sql
SELECT name, milliseconds
FROM track
WHERE milliseconds > (
    SELECT AVG(milliseconds) AS avg_track_length
    FROM track
)
ORDER BY milliseconds DESC;
```
* This retrieves all tracks that exceed the average song length, highlighting potentially noteworthy songs.

### Question Set 3: Advanced

#### q9. Amount spent by each customer on artists
```sql
WITH best_selling_artist AS (
    SELECT artist.artist_id AS artist_id, artist.name AS artist_name, SUM(invoice_line.unit_price*invoice_line.quantity) AS total_sales
    FROM invoice_line
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN album ON album.album_id = track.album_id
    JOIN artist ON artist.artist_id = album.artist_id
    GROUP BY 1
    ORDER BY 3 DESC
    LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY 1, 2, 3, 4
ORDER BY 5 DESC;
```
* This query calculates how much each customer has spent on the best-selling artist, allowing targeted marketing towards high-spending customers.

#### q10. Most popular music genre by country
```sql
WITH popular_genre AS (
    SELECT COUNT(invoice_line.quantity) AS purchases, customer.country, genre.name, genre.genre_id,
        ROW_NUMBER() OVER (PARTITION BY customer.country ORDER BY COUNT(invoice_line.quantity) DESC) AS RowNo
    FROM invoice_line
    JOIN invoice ON invoice.invoice_id = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track ON track.track_id = invoice_line.track_id
    JOIN genre ON genre.genre_id = track.genre_id
    GROUP BY 2, 3, 4
    ORDER BY 2 ASC, 1 DESC
)
SELECT * FROM popular_genre WHERE RowNo <= 1;
```
* This query identifies the most popular music genre for each country, providing insights on market preferences globally.

#### q11. Top customer spending per country
```sql
WITH Customer_with_country AS (
    SELECT customer.customer_id, first_name, last_name, billing_country, SUM(total) AS total_spending,
        ROW_NUMBER() OVER (PARTITION BY billing_country ORDER BY SUM(total) DESC) AS RowNo
    FROM invoice
    JOIN customer ON customer.customer_id = invoice.customer_id
    GROUP BY 1, 2, 3, 4
    ORDER BY 4 ASC, 5 DESC
)
SELECT * FROM Customer_with_country WHERE RowNo <= 1;
```
* This query returns the top spending customer for each country, offering insights into customer value on a geographical basis.

## Conclusion

The SQL queries implemented in this project provide a foundational analysis of the music store's dataset, allowing stakeholders to understand the dynamics of sales, customer behavior, and market trends. This comprehensive analysis can guide future promotional activities, inventory decisions, and customer engagement strategies, ultimately leading to greater profitability and customer satisfaction.
```
