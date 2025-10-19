
![Data Analytics Intership](https://img.shields.io/badge/Data_Analytics-Intership-orange)
[![SQL](https://img.shields.io/badge/SQL-PostgreSQL-blue)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Completed-success)]()

# 🎵 Chinook SQL Analysis

The **Chinook Database** is a sample relational database designed for learning and demonstrating SQL and database management concepts. It models a **digital music store**, including data on artists, albums, tracks, customers, employees, and invoices, making it ideal for practicing analytical SQL queries and exploring relational design.

---

## 📘 Overview

**Database Purpose:**  
Simulates a digital music store to explore queries related to sales, customer behavior, and revenue patterns.  

**Schema Highlights:**  
- **Tables:** Artist, Album, Track, Customer, Invoice, InvoiceLine, Employee, Genre  
- **Relationships:** Artist → Album → Track → InvoiceLine → Invoice → Customer  
- **ERD:** Commonly visualized to understand entity connections.

---

## 🗂 Database Structure

## 🧩 Database Schema Overview

| Table Name            | Description |
|------------------------|-------------|
| **Artist**             | Stores artist details. |
| **Album**              | Contains albums linked to artists. |
| **Track**              | Includes track details such as genre, composer, and price. |
| **Genre**              | Defines music genres. |
| **MediaType**          | Lists file types and formats. |
| **Customer**           | Contains customer information including name, company, and location. |
| **Invoice**            | Records sales transactions and totals. |
| **InvoiceLine**        | Details individual tracks purchased per invoice. |
| **Employee**           | Contains store employees and reporting hierarchy. |
| **Playlist / PlaylistTrack** | Defines playlist structure and the tracks linked to each playlist. |



---

## ⚙️ Setup Instructions

1. Download the Chinook database in your preferred format:  
   - `SQLite`, `MySQL`, `PostgreSQL`, `SQL Server`, or `Oracle`
2. Load it into your SQL environment (e.g., **DB Browser for SQLite**, **pgAdmin**, or **MySQL Workbench**)
3. Run queries directly to explore sales, customers, and music trends.

---
## ERD Diagram

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/ERD%20chinook.png) 

---

## 🧠 SQL Analysis

Below are key insights drawn from analytical SQL queries.  

---
 1. Top Selling Tracks (by Total Revenue)

```sql
SELECT 
    t.Name AS track_name,
    a.Title AS album_title,
    ar.Name AS artist_name,
    SUM(il.Quantity * il.UnitPrice) AS total_revenue
FROM invoiceline AS il
JOIN track AS t ON il.TrackId = t.TrackId
JOIN album AS a ON t.AlbumId = a.AlbumId
JOIN artist AS ar ON a.ArtistId = ar.ArtistId
GROUP BY t.TrackId, t.Name, a.Title, ar.Name
ORDER BY total_revenue DESC
LIMIT 5;

```
![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/1.%20Top%20Selling%20Tracks%20(by%20total%20revenue).png)

### Insight: The Office series consistently rank among the top sellers, each generating $3.98 in revenue, suggesting a strong fan base and stable demand across multiple albums. Heroes and Aquaman have individual high-performing tracks, suggesting that their appeal is limited to specific releases. 


---
 2. Top Selling Tracks (by Units Sold)
```sql
SELECT 
    t.Name AS track_name,
    a.Title AS album_title,
    ar.Name AS artist_name,
    SUM(il.Quantity) AS total_units_sold
FROM invoiceline il
JOIN track AS t ON il.TrackId = t.TrackId
JOIN album AS a ON t.AlbumId = a.AlbumId
JOIN artist AS ar ON a.ArtistId = ar.ArtistId
GROUP BY t.TrackId, t.Name, a.Title, ar.Name
ORDER BY total_units_sold DESC
LIMIT 5;
```

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/2.%20Top%20Selling%20Tracks%20(by%20units%20sold).png?raw=true)

### Insight: The top-selling tracks by units sold show an equal distribution, with each leading track selling two units. The list features a diverse mix of international and regional artists including Eric Clapton, Kiss, and Raul Seixas showing Chinook’s wide-range of audience. Hence suggesting consistent interest across multiple genres rather than dominance by a single artist or album.



---
 3. Revenue by Region
```sql
SELECT 
    i.billingcountry AS region, 
    SUM(i.total) AS revenue 
FROM Invoice AS i
JOIN customer AS c ON i.customerid = c.customerid
GROUP BY i.billingcountry
ORDER BY revenue DESC;
```

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/3.%20Revenue%20per%20region.png?raw=true)

### Insight: The regional revenue analysis shows that USA dominates Chinook’s sales, with ($523.06) and Canada ($303.96). Europe follows with steady performance from France and Germany, while emerging markets like Brazil and India show potential for expansion. This insight highlights the importance of geographic segmentation in identifying growth opportunities and shaping future business strategies.



---
4. Monthly Revenue Trend
```sql
SELECT 
    DATE_TRUNC('month', i.invoicedate) AS month,
    COUNT(DISTINCT i.invoiceid) AS num_invoices,
    SUM(il.unitprice * il.quantity) AS revenue
FROM invoice AS i
JOIN invoiceline AS il ON i.invoiceid = il.invoiceid
GROUP BY DATE_TRUNC('month', i.invoicedate)
ORDER BY month DESC;
```

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/4.%20Monthly%20Revenue.png?raw=true)

### Insight: The monthly revenue trend shows a steady increase throughout the year, with notable peaks in June and November, possibly driven by seasonal sales. Revenue dips slightly in the early months, suggesting slower customer activity post-holiday. Overall, the upward trend indicates consistent customer engagement and healthy business growth over time.



---
 5. Listeners Who Love Jazz, Rock, and Pop
```sql
WITH genre_tracks AS (
    SELECT trackid, g.name AS genre
    FROM track t
    JOIN genre g ON g.genreid = t.genreid
    WHERE g.name IN ('Jazz', 'Rock', 'Pop')
)
SELECT 
    (c.firstname || ' ' || c.lastname) AS customer_name, 
    c.email, c.country, gt.genre
FROM invoiceline il
JOIN invoice i    ON i.invoiceid = il.invoiceid
JOIN customer c   ON c.customerid = i.customerid
JOIN genre_tracks gt ON gt.trackid = il.trackid;
```

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/5.%20Name%20emailid%20country%20genre.png?raw=true) 


### Insight: The analysis of listeners who love Jazz, Rock, and Pop highlights distinct audience preferences within Chinook’s customer base. Rock emerges as the dominant genre with total tracks of 1297 and sold tracks nearly 835, followed by Pop, indicating mainstream appeal. Jazz, while smaller in audience size, attracts a dedicated listener segment. Understanding these genre preferences can guide personalized marketing strategies, improving playlist curation, and optimize inventory for high-demand genres.



---
6. Top Customers by Genre
```sql
SELECT customer_name, email, country, genre, rank
FROM (
    SELECT (c.firstname || ' ' || c.lastname) AS customer_name,
           c.email, c.country,  g.name AS genre,
           RANK() OVER (PARTITION BY g.name ORDER BY COUNT(*) DESC) AS rank
    FROM invoiceline il
    JOIN track t      ON t.trackid = il.trackid
    JOIN genre g      ON g.genreid = t.genreid
    JOIN invoice i    ON i.invoiceid = il.invoiceid
    JOIN customer c   ON c.customerid = i.customerid
    WHERE g.name IN ('Jazz', 'Rock', 'Pop')
    GROUP BY c.firstname, c.lastname, c.email, c.country, g.name
) ranked
WHERE rank <= 5;
```
![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/6.%20Top%20Customers%20by%20Genres.png?raw=true)


### Insight: Jazz is the most popular genre, with most customers spending 0.99, while a few high-value customers (spending 1.98) stand out. Customers come from diverse countries, indicating international reach. Prioritizing higher-spending customers could enhance loyalty, and targeting the Jazz genre may be effective for marketing strategies. 




---
 7. Top Customers by Total Spend
```sql
SELECT 
    customer_name, email, country, genre, total_spent,
    rank
FROM (
    SELECT 
        (c.firstname || ' ' || c.lastname) AS customer_name,
        c.email, c.country, g.name AS genre,
        ROUND(SUM(il.unitprice * il.quantity), 2) AS total_spent,
        RANK() OVER (PARTITION BY c.country ORDER BY SUM(il.unitprice * il.quantity) DESC) AS rank
    FROM invoiceline il
    JOIN track AS t      ON t.trackid = il.trackid
    JOIN genre AS g      ON g.genreid = t.genreid
    JOIN invoice AS i    ON i.invoiceid = il.invoiceid
    JOIN customer AS c   ON c.customerid = i.customerid
    WHERE g.name IN ('Jazz', 'Rock', 'Pop')
    GROUP BY c.firstname, c.lastname, c.email, c.country, g.name
) ranked
WHERE rank <= 5
ORDER BY total_spent, rank
LIMIT 10;
```

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/7.%20Top%20customers%20by%20Total%20spend.png?raw=true) 

### Insight: Highest spenders are from diverse countries like France for Jazz, Brazil for Rock, showing a global reach




---
 8. Most and Least Popular Genres
```sql
WITH temp AS 
(SELECT g.name AS genre, COUNT(1) AS no_of_songs,
        RANK() OVER(ORDER BY COUNT(1) DESC) AS rnk
    FROM invoiceline il
    JOIN track AS t ON t.trackid = il.trackid
    JOIN genre AS g ON g.genreid = t.genreid
    GROUP BY g.name
    ORDER BY 2 DESC
),
max_rank AS 
( SELECT MAX(rnk) AS max_rnk FROM temp )
SELECT 
    genre, no_of_songs,
    CASE 
        WHEN rnk = 1 THEN 'Most Popular' 
        ELSE 'Least Popular' 
    END AS popular_flag
FROM temp
INNER JOIN max_rank ON rnk = max_rnk OR rnk = 1;
```

![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/8.%20Most%20and%20Least%20Popular%20Genres.png?raw=true) 

### Insight: Rock is the most popular genre with sold tracks nearly 835.




---
 9. Employee Supporting Most Customers
```sql

SELECT employee_name, title AS designation
FROM (
    SELECT (e.firstname||' '||e.lastname) AS employee_name, e.title,
           COUNT(1) AS no_of_customers,
           RANK() OVER(ORDER BY COUNT(1) DESC) AS rnk
    FROM Customer c
    JOIN employee e ON e.employeeid = c.supportrepid
    GROUP BY e.firstname, e.lastname, e.title
) x
WHERE x.rnk = 1;
```
![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/9.%20Employee%20who%20has%20supported%20the%20most%20no%20of%20customers.png?raw=true) 

### Insight: Jane Peacock has the highest customers of 21. 






---
10. Business Summary
```sql
WITH line_revenue AS 
(
  SELECT il.invoiceid, (il.unitprice * il.quantity) AS line_total
  FROM invoiceline AS il
)
SELECT
  (SELECT COUNT(*) FROM customer)  AS customers,
  (SELECT COUNT(*) FROM invoice) AS invoices,
  (SELECT ROUND(AVG(total),2) FROM invoice) AS avg_invoice_value,
  (SELECT ROUND(SUM(lr.line_total),2) FROM line_revenue lr) AS total_revenue,
  (SELECT SUM(quantity) FROM invoiceline) AS total_tracks_sold,
  (SELECT COUNT(DISTINCT country) FROM customer) AS countries_served;
```
![Dashboard Preview](https://github.com/Jeseenacodes/Internship/blob/main/Task%203/10.%20Business%20Summary.png?raw=true) 

### Insight: The Business serves 59 customers, with 412 invoices issued in total. The average invoice value is 5.65 indicating a low average transaction size. The total revenue from these customers is $2328.60. A total of 2240 tracks have been sold where customers from 24 countries have purchased indicating broad international presence. 


---
## Recommednations 

1. Target localized promotions in top regions like USA & Canada and invest in emerging markets like Brazil & India to expand the customer base.
2. Promote Rock collections while offering premium loyalty programs for Jazz listeners to enhance retention and spend.
3. Introduce discount/ loyalty programs during low-activity months like Jan-March to maintain sales.
4. Highlight top-perming tracks to boost engagement and invest in franchise-based albums.
5. Introduce bundle discounts or subscription models to increase average transaction value.
6. Reward frequent buyers with loyalty benefits or access to exclusive content.
7. Group customers by pattern of spending to curate special offers.
8. Involve if partnerships with international artists to enhance sales in underperforming regions.  

## Conclusion 

The insights from this analysis highlight key opportunities for Chinook to strengthen its performance. By applying the recommended strategies, the store can address current challenges, enhance customer engagement, and drive growth in sales and market reach.








