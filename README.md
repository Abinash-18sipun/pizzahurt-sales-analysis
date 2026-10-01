# pizzahurt-sales-analysis
create database pizzahut;
create table orders(
order_id int not null,
order_date date not null,
order_time  time not null,
primary key(order_id) );

create table order_details(
order_details_id int not null,
order_id int not null,
pizza_id  text not null,
quantity int not null,
primary key(order_details_id) );

select * from order_details;
select * from orders;
select * from pizza_types;
select * from pizzas;
-- Retrieve the total number of orders placed--
select count(order_id) as Total_orders from orders;

-- Calculate the total revenue generated from pizza sales --
SELECT 
    ROUND(SUM(o.quantity * p.price), 2) AS total_sales
FROM
    order_details AS o
        JOIN
    pizzas AS p ON p.pizza_id = o.pizza_id;
    
-- Identify the highest priced pizzza--
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;

-- determine the distribution of orders by hour of the day--
SELECT 
    HOUR(order_time) AS order_hour, COUNT(order_id) order_count
FROM
    orders
GROUP BY order_hour
ORDER BY order_hour ASC;

-- category wise distribution of pizzas--
select category, count(pizza_type_id) as total_pizzas from pizza_types group by category;

-- Identify the most common pizza size ordered--
SELECT 
    pizzas.size,
    COUNT(order_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC;

-- list the top 5 most orderd pizza types along with their quantites --
SELECT 
    pizza_types.name, SUM(order_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
-- join necessary tabels to find the total number of each pizza caterory orderd--
SELECT 
    pizza_types.category, SUM(order_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;

-- Group the orders by date and calculate the average number of pizzas ordered per day--
SELECT 
    ROUND(AVG(QUANTITY), 0) AS order_quantity
FROM
    (SELECT 
        orders.order_date, SUM(order_details.quantity) AS quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS order_quantity ;
    
-- Determine the top 3 most ordered pizza types based on revenue-
SELECT 
    pizza_types.name,
    SUM(order_details.quantity * pizzas.price) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizzas.pizza_type_id = pizza_types.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;

-- Calculate the percentage contribution of each pizza type to total revenue
SELECT 
    pizza_types.category,
    ROUND(
        SUM(order_details.quantity * pizzas.price) /
        (
            SELECT SUM(od.quantity * p.price)
            FROM order_details od
            JOIN pizzas p
                ON p.pizza_id = od.pizza_id
        ) * 100,
        2
    ) AS revenue
FROM pizza_types
JOIN pizzas
    ON pizza_types.pizza_type_id = pizzas.pizza_type_id
JOIN order_details
    ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;

-- Analyze the cumulative revenue generated over time.
select order_date , round(sum(revenue) over (order by order_date) , 2) as cumulative_revenue 
from
(select orders.order_date , sum(pizzas.price * order_details.quantity) as revenue
from orders
join order_details
on orders.order_id = order_details.order_id 
join pizzas
on pizzas.pizza_id = order_details.pizza_id
group by orders.order_date
) as sales;

-- Determine the top 3 most ordered pizza types based on revenue for each pizza category.
Select category as Category , name as Name, round(revenue, 2) as Revenue
from
(Select category , name , revenue ,
rank() over(partition by category order by revenue desc) as rn
from
(select pizza_types.category , pizza_types.name ,
sum(pizzas.price * order_details.quantity) as revenue
from pizza_types
join pizzas
on pizzas.pizza_type_id = pizza_types.pizza_type_id
join order_details
on pizzas.pizza_id = order_details.pizza_id 
group by pizza_types.category , pizza_types.name ) as a) as b
where rn <=3 ;













