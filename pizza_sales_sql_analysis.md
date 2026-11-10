# Pizza Sales Data Analysis Project

## Project Overview
This project involves a comprehensive analysis of pizza sales data using SQL. The goal is to extract actionable business insights from a relational database containing order details, pizza types, and pricing information. The analysis is structured into Basic, Intermediate, and Advanced queries to explore sales volume, revenue generation, and categorical performance.

## Database Structure
The project assumes a relational schema with the following tables:
* `orders`: Contains `order_id`, `order_date`, `order_time`.
* `order_details`: Contains `order_detail_id`, `order_id`, `pizza_id`, `quantity`.
* `pizzas`: Contains `pizza_id`, `pizza_type_id`, `size`, `price`.
* `pizza_types`: Contains `pizza_type_id`, `name`, `category`, `ingredients`.

---

## Part 1: Basic Analysis

**1. Retrieve the total number of orders placed.**
```sql
SELECT COUNT(order_id) AS total_orders 
FROM orders;
```

**2. Calculate the total revenue generated from pizza sales.**
```sql
SELECT ROUND(SUM(od.quantity * p.price), 2) AS total_revenue
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id;
```

**3. Identify the highest-priced pizza.**
```sql
SELECT pt.name, p.price 
FROM pizzas p
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
ORDER BY p.price DESC 
LIMIT 1;
```

**4. Identify the most common pizza size ordered.**
```sql
SELECT p.size, COUNT(od.order_detail_id) AS order_count
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
GROUP BY p.size
ORDER BY order_count DESC 
LIMIT 1;
```

**5. List the top 5 most ordered pizza types along with their quantities.**
```sql
SELECT pt.name, SUM(od.quantity) AS total_quantity
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY total_quantity DESC 
LIMIT 5;
```

---

## Part 2: Intermediate Analysis

**1. Find the total quantity of each pizza category ordered.**
```sql
SELECT pt.category, SUM(od.quantity) AS total_quantity
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY total_quantity DESC;
```

**2. Determine the distribution of orders by hour of the day.**
```sql
SELECT HOUR(order_time) AS order_hour, COUNT(order_id) AS total_orders
FROM orders
GROUP BY HOUR(order_time)
ORDER BY order_hour;
```

**3. Find the category-wise distribution of pizzas.**
```sql
SELECT category, COUNT(pizza_type_id) AS pizza_count
FROM pizza_types
GROUP BY category;
```

**4. Calculate the average number of pizzas ordered per day.**
```sql
SELECT ROUND(AVG(daily_pizzas), 0) AS avg_pizzas_per_day
FROM (
    SELECT o.order_date, SUM(od.quantity) AS daily_pizzas
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.order_date
) AS daily_orders_summary;
```

**5. Determine the top 3 most ordered pizza types based on revenue.**
```sql
SELECT pt.name, ROUND(SUM(od.quantity * p.price), 2) AS total_revenue
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.name
ORDER BY total_revenue DESC 
LIMIT 3;
```

---

## Part 3: Advanced Analysis

**1. Calculate the percentage contribution of each pizza type to total revenue.**
```sql
SELECT pt.category, 
       ROUND(SUM(od.quantity * p.price) / 
       (SELECT SUM(od2.quantity * p2.price) 
        FROM order_details od2 
        JOIN pizzas p2 ON od2.pizza_id = p2.pizza_id) * 100, 2) AS revenue_percentage
FROM order_details od
JOIN pizzas p ON od.pizza_id = p.pizza_id
JOIN pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY revenue_percentage DESC;
```

**2. Analyze the cumulative revenue generated over time.**
```sql
WITH daily_revenue AS (
    SELECT o.order_date, SUM(od.quantity * p.price) AS revenue
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    JOIN pizzas p ON od.pizza_id = p.pizza_id
    GROUP BY o.order_date
)
SELECT order_date, 
       revenue, 
       ROUND(SUM(revenue) OVER(ORDER BY order_date), 2) AS cumulative_revenue
FROM daily_revenue;
```

**3. Determine the top 3 most ordered pizza types based on revenue for each pizza category.**
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
```