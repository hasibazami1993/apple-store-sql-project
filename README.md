
# ![Apple Logo](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/Apple_Changsha_RetailTeamMembers_09012021_big.jpg.slideshow-xlarge_2x.jpg) Apple Retail Sales SQL Project - Analyzing Millions of Sales Rows

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)


## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.
```
   SELECT 
	country,
	COUNT(store_id) as total_stores
FROM stores
GROUP BY 1
ORDER BY 2 DESC;
```
2. Calculate the total number of units sold by each store.
```
SELECT 
	s.store_id,
	st.store_name,
	SUM(s.quantity) as total_unit_sold
FROM sales as s
JOIN
stores as st
ON st.store_id = s.store_id
GROUP BY 1, 2
ORDER BY 3 DESC;
```
3. Identify how many sales occurred in December 2023.
```
SELECT
COUNT(sale_id) as total_sales
FROM sales
WHERE TO_CHAR(sale_date, 'MM-YYYY') = '12-2023'
```
4. Determine how many stores have never had a warranty claim filed.
```
SELECT COUNT(*) FROM stores
WHERE store_id NOT IN (
SELECT 
	DISTINCT store_id
FROM sales as s
RIGHT JOIN warranty as w
ON s.sale_id = w.sale_id
)
;
```
5. Calculate the percentage of warranty claims marked as "Warranty Void".
```
SELECT 
	ROUND(COUNT(claim_id)/(SELECT COUNT(*) FROM warranty)::numeric * 100,2) as warranty_void_pct
FROM warranty
WHERE repair_status = 'Warranty Void';
```
6. Identify which store had the highest total units sold in the last year.
```
SELECT 
	s.store_id,
	st.store_name,
	SUM(s.quantity)
FROM sales as s
JOIN stores as st
ON s.store_id = st.store_id
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 YEAR')
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 1;
```
7. Count the number of unique products sold in the last year.
```
SELECT
	COUNT(DISTINCT product_id)
FROM sales
WHERE sale_date >= (CURRENT_DATE - INTERVAL '1 YEAR')
```
8. Find the average price of products in each category.
```
SELECT
	category_name,
	ROUND(AVG(price::numeric),2) as average_price
FROM
(SELECT
	* 
FROM category c LEFT JOIN products p
ON c.category_id = p.category_id
) j
GROUP BY 1
ORDER BY 2 DESC
;

```
9. How many warranty claims were filed in 2020?
```
SELECT
	COUNT(DISTINCT CASE
		WHEN TO_CHAR(claim_date,'YYYY') = '2020' THEN claim_id END) AS num_of_claims
	FROM warranty;
```
10. For each store, identify the best-selling day based on highest quantity sold.
```
SELECT
	*
	FROM
(SELECT
	s.store_name,
	TO_CHAR(sa.sale_date,'Day') as day_name,
	SUM(sa.quantity) as total_sold,
	DENSE_RANK() OVER(PARTITION BY s.store_name ORDER BY SUM(sa.quantity) DESC) as rank
	FROM stores s JOIN sales sa
	ON s.store_id = sa.store_id
	GROUP BY 1,2
	ORDER BY 3 DESC) as rank
	WHERE rank = 1;
```

### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.
```
WITH product_rank AS (
SELECT 
	st.country,
	p.product_name,
	SUM(s.quantity) as total_qty_sold,
	RANK() OVER(PARTITION BY st.country ORDER BY SUM(s.quantity)) as rank
FROM sales as s
JOIN
stores as st
ON s.store_id = st.store_id
JOIN
products as p
ON s.product_id = p.product_id
GROUP BY 1,2
)
SELECT
	*
	FROM product_rank
	WHERE rank=1
;
```
13. Calculate how many warranty claims were filed within 180 days of a product sale.
```
SELECT 
COUNT(DISTINCT CASE WHEN w.claim_date - s.sale_date <= 180 THEN w.claim_id END) AS claim_count
FROM warranty as w
LEFT JOIN sales as s
ON w.sale_id = s.sale_id;
```
15. Determine how many warranty claims were filed for products launched in the last two years.
```
SELECT
	p.product_name,
	COUNT(w.claim_id) as no_claim,
	COUNT(s.sale_id)
FROM warranty as w
RIGHT JOIN sales as s
	ON w.sale_id = s.sale_id
LEFT JOIN products as p
	ON s.product_id = p.product_id
WHERE p.launch_date >= CURRENT_DATE - INTERVAL '2 YEARS'
GROUP BY 1
HAVING COUNT(w.claim_id) > 0;

```
17. List the months in the last three years where sales exceeded 5,000 units in the USA.
```
SELECT 
	TO_CHAR(sale_date, 'MM-YYYY') as month,
	SUM(s.quantity) AS total_units_sold
FROM sales as s
JOIN stores as st
ON s.store_id = st.store_id
WHERE st.country = 'USA'
AND s.sale_date >= CURRENT_DATE - INTERVAL '3 YEARS'
GROUP BY 1
HAVING SUM(s.quantity) > 5000;
```
19. Identify the product category with the most warranty claims filed in the last two years.
```
SELECT 
	c.category_name,
	COUNT(w.claim_id) as total_claims
FROM warranty as w
LEFT JOIN
sales as s
ON w.sale_id = s.sale_id
JOIN 
products as p
ON p.product_id = s.product_id
JOIN 
category as c
ON c.category_id = p.category_id
WHERE
	w.claim_date >= CURRENT_DATE - INTERVAL '2 YEAR'
GROUP BY 1
ORDER BY COUNT(w.claim_id) DESC;
```

### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```
SELECT 
	country,
	total_units_sold,
	total_claims,
	COALESCE(total_claims::numeric/total_units_sold::numeric * 100, 0) as risk
FROM
(SELECT 
	st.country,
	SUM(s.quantity) as total_units_sold,
	COUNT(w.claim_id) as total_claims
FROM sales as s
JOIN stores as st
ON s.store_id = st.store_id
LEFT JOIN
warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1)
ORDER BY 4 DESC
;
```
18. Analyze the year-by-year growth ratio for each store.
```
WITH yearly_sales AS
(
SELECT
	s.store_id,
	st.store_name,
	EXTRACT(YEAR FROM sale_date) AS year,
	SUM(s.quantity * p.price) as total_sales
FROM sales as s
JOIN products as p
ON s.product_id = p.product_id
JOIN stores as st
ON st.store_id = s.store_id
GROUP BY 1,2,3
ORDER BY 2,3
),
growth_ratio AS (
SELECT
	store_name,
	year,
	LAG(total_sales,1) OVER(PARTITION BY store_name ORDER BY year ASC) AS last_year_sale,
	total_sales as current_year_sale
FROM yearly_sales
)
SELECT 
	store_name,
	year,
	last_year_sale,
	current_year_sale,
	ROUND((current_year_sale - last_year_sale)::numeric/last_year_sale::numeric * 100,3) as growth_ratio
FROM growth_ratio
WHERE last_year_sale IS NOT NULL
AND YEAR <> EXTRACT(YEAR FROM CURRENT_DATE)
;
```
20. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```
SELECT
	CASE
		WHEN p.price < 500 THEN 'Less Expensive Product'
		WHEN p.price BETWEEN 500 AND 1000 THEN 'Mid Range Product'
		ELSE 'Expensive Product'
	END as price_segment,
	COUNT(w.claim_id) as total_claims
FROM warranty as w
LEFT JOIN 
sales as s
ON w.sale_id = s.sale_id
JOIN
products as p
ON p.product_id = s.product_id
WHERE claim_date >= CURRENT_DATE - INTERVAL '5 YEAR'
GROUP BY 1;
```
22. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```
WITH paid_repair AS (
SELECT
	s.store_id,
	COUNT(w.claim_id) as paid_repaired
FROM sales as s
RIGHT JOIN warranty as w
ON w.sale_id = s.sale_id
WHERE w.repair_status = 'Paid Repaired'
GROUP BY 1
),
total_repaired
AS
(SELECT
	s.store_id,
	COUNT(w.claim_id) as total_repaired
FROM sales as s
RIGHT JOIN warranty as w
ON w.sale_id = s.sale_id
GROUP BY 1
)
SELECT
	tr.store_id,
	st.store_name,
	pr.paid_repaired,
	tr.total_repaired,
	ROUND(pr.paid_repaired::numeric/tr.total_repaired::numeric * 100,2) as percentage_paid_repair
FROM paid_repair as pr
JOIN
total_repaired tr
ON pr.store_id = tr.store_id
JOIN stores as st
ON tr.store_id = st.store_id
;

```
24. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```
WITH monthly_sales AS (
SELECT
	store_id,
	EXTRACT(YEAR FROM sale_date) as year,
	TO_CHAR(sale_date, 'Month') as month,
	SUM(p.price * s.quantity) as total_revenue
FROM sales as s
JOIN 
products as p
ON s.product_id = p.product_id
GROUP BY 1,2,3
ORDER BY 1, 2,3
)
SELECT
	store_id,
	month,
	year,
	total_revenue,
	SUM(total_revenue) OVER (PARTITION BY store_id ORDER BY year, month) AS running_total
FROM monthly_sales
```


## Project Focus


- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.


---
