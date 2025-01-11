
# Music Store Database Analysis Using SQL

## Objective:
Analyzed a music store's transactional database to extract insights related to customer behavior, sales trends, and artist performance for strategic decision-making.

---

## Queries and Insights

### **1. Senior-most Employee Identification**
**Query:**
```sql
SELECT * 
FROM employee
ORDER BY levels DESC
LIMIT 1;
```
**Insight:** Identified the employee with the highest job level, useful for hierarchical planning.

---

### **2. Top Countries by Invoice Count**
**Query:**
```sql
SELECT COUNT(invoice_id) AS invoice_count, billing_country  
FROM invoice
GROUP BY billing_country
ORDER BY invoice_count DESC
LIMIT 5;
```
**Insight:** Determined the top 5 countries generating the most invoices, guiding global market strategies.

---

### **3. Top Invoice Totals**
**Query:**
```sql
SELECT total 
FROM invoice
ORDER BY total DESC
LIMIT 5;
```
**Insight:** Retrieved the highest invoice amounts, highlighting high-value transactions.

---

### **4. Top Revenue-Generating City**
**Query:**
```sql
SELECT billing_city, SUM(total) AS total_revenue 
FROM invoice 
GROUP BY billing_city
ORDER BY total_revenue DESC 
LIMIT 1;
```
**Insight:** Identified the city generating the highest revenue to plan promotional events effectively.

---

### **5. Best Customer Identification**
**Query:**
```sql
SELECT c.customer_id, c.first_name, c.last_name, SUM(i.total) AS total_spent 
FROM invoice i
JOIN customer c USING(customer_id)
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC 
LIMIT 1;
```
**Insight:** Determined the customer who spent the most, enabling targeted loyalty campaigns.

---

### **6. Rock Music Listeners**
**Query:**
```sql
SELECT DISTINCT(c.email), c.first_name, c.last_name, g.name 
FROM customer c
JOIN invoice USING(customer_id)
JOIN invoice_line USING(invoice_id)
JOIN track USING(track_id)
JOIN genre g USING(genre_id)
WHERE g.name LIKE 'Rock'
ORDER BY c.email ASC;
```
**Insight:** Listed customers preferring Rock music, aiding genre-specific marketing.

---

### **7. Top Rock Artists**
**Query:**
```sql
SELECT a.name, COUNT(t.track_id) AS track_count 
FROM artist a
JOIN album alb USING(artist_id)
JOIN track t ON t.album_id = alb.album_id
JOIN genre g USING(genre_id)
WHERE g.name = 'Rock'
GROUP BY a.name
ORDER BY track_count DESC
LIMIT 10;
```
**Insight:** Highlighted top Rock artists by track count for strategic promotions.

---

### **8. Tracks Longer than Average Length**
**Query:**
```sql
SELECT name, milliseconds 
FROM track 
WHERE milliseconds > (SELECT AVG(milliseconds) FROM track)
ORDER BY milliseconds DESC;
```
**Insight:** Identified tracks longer than average for playlist creation.

---

### **9. Customer Spending on Artists**
**Query:**
```sql
WITH best_selling_artist AS (
  SELECT artist.artist_id, artist.name AS artist_name, 
         SUM(invoice_line.unit_price * invoice_line.quantity) AS total_sales
  FROM invoice_line
  JOIN track ON track.track_id = invoice_line.track_id
  JOIN album ON album.album_id = track.album_id
  JOIN artist ON artist.artist_id = album.artist_id
  GROUP BY artist.artist_id, artist.name
  ORDER BY total_sales DESC
  LIMIT 1
)
SELECT c.customer_id, c.first_name, c.last_name, bsa.artist_name, 
       SUM(il.unit_price * il.quantity) AS amount_spent
FROM invoice i
JOIN customer c ON c.customer_id = i.customer_id
JOIN invoice_line il ON il.invoice_id = i.invoice_id
JOIN track t ON t.track_id = il.track_id
JOIN album alb ON alb.album_id = t.album_id
JOIN best_selling_artist bsa ON bsa.artist_id = alb.artist_id
GROUP BY c.customer_id, c.first_name, c.last_name, bsa.artist_name
ORDER BY amount_spent DESC;
```
**Insight:** Tracked customer spending on specific artists to refine marketing efforts.

---

### **10. Most Popular Genre by Country**
**Query:**
```sql
WITH most_popular_genre AS (
  SELECT g.genre_id, g.name, c.country, COUNT(i.quantity) AS purchases,
         ROW_NUMBER() OVER(PARTITION BY c.country ORDER BY COUNT(i.quantity) DESC) AS rownum
  FROM genre g
  JOIN track USING (genre_id)
  JOIN invoice_line i USING(track_id)
  JOIN invoice USING(invoice_id)
  JOIN customer c USING(customer_id)
  GROUP BY g.genre_id, g.name, c.country
)
SELECT * FROM most_popular_genre 
WHERE rownum = 1;
```
**Insight:** Found the most purchased genre in each country to understand regional preferences.

---

### **11. Top Customers by Country**
**Query:**
```sql
WITH top_customer AS (
  SELECT c.customer_id, CONCAT(c.first_name, ' ', c.last_name) AS customer_name, 
         i.billing_country, SUM(i.total) AS amount_spent,
         ROW_NUMBER() OVER(PARTITION BY i.billing_country ORDER BY SUM(i.total) DESC) AS rownum
  FROM invoice i
  JOIN customer c USING(customer_id)
  GROUP BY c.customer_id, customer_name, i.billing_country
)
SELECT * FROM top_customer 
WHERE rownum = 1;
```
**Insight:** Identified the highest spenders in each country for targeted outreach.

---

## Technical Skills Demonstrated
- **SQL Techniques:** Proficiency in joins, window functions, CTEs, and aggregations.
- **Data Insights:** Derived actionable insights for marketing and business decisions.
- **Performance Optimization:** Designed efficient queries for large datasets.

---

## Outcome
The analysis provided critical insights into customer behavior, regional preferences, and artist performance, enabling data-driven decisions for promotions, inventory management, and targeted campaigns.
