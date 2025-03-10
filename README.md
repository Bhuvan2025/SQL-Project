# SQL Project--Pizza Sales

Use Pizza_sales;

# Retrieve the total number of orders placed.

SELECT 
    COUNT(order_id) AS Count
FROM
    order_details;

# Calculate the total revenue generated from pizza sales.

SELECT 
    ROUND(SUM(quantity * price), 2) AS Total_Revenue
FROM
    order_details
        INNER JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id;
    
# Identify the highest-priced pizza.

SELECT 
    name, price
FROM
    pizza_types
        INNER JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY price DESC
LIMIT 1;

# Identify the most common pizza size ordered.

SELECT 
    size, SUM(quantity) AS Quanity_ordered
FROM
    order_details
        INNER JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id
GROUP BY size
ORDER BY 2 DESC;

# List the top 5 most ordered pizza types along with their quantities.

SELECT 
    name, SUM(quantity) AS Quantity_Ordered
FROM
    order_details
        INNER JOIN
    (SELECT 
        name, pizza_id
    FROM
        pizza_types
    INNER JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id) AS P1 ON order_details.pizza_id = P1.pizza_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

# Join the necessary tables to find the total quantity of each pizza category ordered.

SELECT 
    category, SUM(quantity) AS Total_Quanity
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id
        JOIN
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY 1
ORDER BY 2 DESC;

# Determine the distribution of orders by hour of the day.

SELECT 
    HOUR(time) AS Hour, COUNT(order_id) AS Orders_placed
FROM
    orders
GROUP BY 1
ORDER BY 1;

# Join relevant tables to find the category-wise distribution of pizzas.

SELECT 
    category, COUNT(pizza_type_id) as No_of_pizzas
FROM
    pizza_types
GROUP BY category;

# Group the orders by date and calculate the average number of pizzas ordered per day.
SELECT 
    ROUND(AVG(Qty)) as Avg_pizzas_per_day
FROM
    (SELECT 
        date, SUM(quantity) AS Qty
    FROM
        order_details
    JOIN orders ON order_details.order_id = orders.order_id
    GROUP BY 1) AS Table1;

# Determine the top 3 most ordered pizza types based on revenue.

SELECT 
    name, SUM(quantity * price) AS Total_revenue
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id
        JOIN
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;

# Calculate the percentage contribution of each pizza type to total revenue.

SELECT 
    category,
    ROUND((SUM(quantity * price) / (SELECT 
                    SUM(quantity * price)
                FROM
                    order_details
                        JOIN
                    pizzas ON order_details.pizza_id = pizzas.pizza_id)) * 100,
            2) AS Revenue_Percent
FROM
    order_details
        JOIN
    pizzas ON order_details.pizza_id = pizzas.pizza_id
        JOIN
    pizza_types ON pizzas.pizza_type_id = pizza_types.pizza_type_id
GROUP BY 1
ORDER BY 2 DESC;


# Analyze the cumulative revenue generated over time.
SELECT 
	date, 
    ROUND(SUM(revenue) OVER(order by date), 2) as Cumulative_revenue
FROM
	(SELECT 
		date, 
		SUM(quantity*price) as revenue
	FROM
		order_details 
			JOIN
		orders ON order_details.order_id = orders.order_id
			JOIN
		pizzas ON order_details.pizza_id = pizzas.pizza_id
	GROUP BY 1
	ORDER BY 1) AS sales;
    
    

# Determine the top 3 most ordered pizza types based on revenue for each pizza category.
SELECT
	category, 
    name, ROUND(Revenue, 2) AS Total_Revenue
FROM
	(SELECT
		category, 
		name, 
		Revenue, 
				RANK() OVER(PARTITION BY 
				category 
                ORDER BY Revenue DESC) AS Ranking 
    FROM
		(SELECT
			category, 
            name, 
            SUM(quantity*price) AS Revenue
		FROM
			pizza_types 
				JOIN 
			pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
				JOIN
			order_details ON pizzas.pizza_id = order_details.pizza_id
			GROUP BY 1,2) AS Table1) AS Table2
WHERE Ranking <=3;


