# Retail Sales Performance Analytics (SQL)

## About the Project
I built this SQL project to analyze a retail dataset (pizza sales) and answer practical business questions. The goal was to practice writing queries that mirror real-world data requests, progressing from basic metrics to more complex revenue modeling. 

I wrote these queries to track order volume, pricing trends, and category performance across four linked tables. 

## Tech Stack & Skills Used
* **Tools:** SQL (MySQL)
* **Techniques:** Multi-table `JOIN`s, `GROUP BY` aggregations, Subqueries, Date/Time functions
* **Advanced SQL:** Common Table Expressions (CTEs), Window Functions (`RANK() OVER`, `SUM() OVER`)

## The Database
The data is spread across four relational tables:
1. **`orders`**: Tracks individual orders (date and time).
2. **`order_details`**: Line-item details for each order (what was ordered and how many).
3. **`pizzas`**: Pricing and sizing info for specific pizza IDs.
4. **`pizza_types`**: High-level categorization (name, category, ingredients).

## What I Analyzed 

I structured the project into three phases based on complexity:

### 1. Basic Metrics
* Found the total number of orders and overall revenue.
* Identified the most common pizza sizes and the highest-priced items.
* Pulled the top 5 best-selling pizzas by raw quantity.

### 2. Intermediate Trends
* Mapped out order volume by hour to find peak business times.
* Grouped sales by category to see which types of pizza sell the most.
* Calculated the average number of pizzas ordered per day using subqueries.

### 3. Advanced Revenue Modeling
* **Revenue Share:** Calculated the exact percentage each pizza category contributes to total revenue.
* **Running Totals:** Used the `SUM() OVER` window function to track cumulative rolling revenue over time.
* **Category Rankings:** Wrote CTEs combined with `RANK() OVER(PARTITION BY...)` to isolate the top 3 revenue-generating pizzas within each specific category.

## Sample Code
Here is a look at the query I wrote to rank the top 3 pizzas by revenue, partitioned by category. I used CTEs to keep the logic clean and avoid messy nested subqueries.

```sql
WITH category_revenue AS (
    SELECT pt.category, pt.name, SUM(od.quantity * p.price) AS revenue
    FROM order_details od
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
    GROUP BY pt.category, pt.name
),
ranked_revenue AS (
    SELECT category, name, revenue,
           RANK() OVER(PARTITION BY category ORDER BY revenue DESC) AS revenue_rank
    FROM category_revenue
)
SELECT category, name, revenue
FROM ranked_revenue
WHERE revenue_rank <= 3;
