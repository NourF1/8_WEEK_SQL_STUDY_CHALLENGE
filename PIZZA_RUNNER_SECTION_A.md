                                                                     --Case Study #2: PIZZA RUNNER A



-- 1. How many pizzas were ordered?
     
     SELECT COUNT(order_id) AS pizzas_ordered 
     FROM pizza_runner.customer_orders;

-- 2. How many unique customer orders were made?

     SELECT COUNT(DISTINCT order_id) 
     FROM pizza_runner.customer_orders;

-- 3. How many successful orders were delivered by each runner?

     SELECT runner_id, COUNT(*) 
     FROM pizza_runner.runner_orders 
     WHERE pickup_time != 'null' 
     GROUP BY runner_id 
     ORDER BY runner_id;

-- 4. How many of each type of pizza was delivered?

     SELECT pizza_name , COUNT(co.pizza_id) 
     FROM pizza_runner.customer_orders AS co
     JOIN pizza_runner.runner_orders AS ro ON ro.order_id=co.order_id
     JOIN pizza_runner.pizza_names AS pn ON pn.pizza_id=co.pizza_id
     WHERE pickup_time != 'null'
     GROUP BY pizza_name;
     
-- 5. How many Vegetarian and Meatlovers were ordered by each customer?
     
     SELECT customer_id, pizza_name, COUNT(pizza_name) 
     FROM pizza_runner.customer_orders AS co
     JOIN pizza_runner.pizza_names AS pn ON pn.pizza_id=co.pizza_id
     GROUP BY pizza_name, co.customer_id 
     ORDER BY co.customer_id;
 
 -- 6. What was the maximum number of pizzas delivered in a single order?
 
     SELECT co.order_id, COUNT(pizza_id) 
     FROM pizza_runner.customer_orders AS co
     JOIN pizza_runner.runner_orders AS ro ON ro.order_id=co.order_id
     WHERE pickup_time != 'null'
     GROUP BY co.order_id
     ORDER BY COUNT(pizza_id) DESC
     LIMIT 1;
     
-- 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

     SELECT customer_id, 
     SUM(CASE WHEN (exclusions != 'null' AND LENGTH(exclusions) <> 0) OR (extras != 'null' AND LENGTH(extras) <> 0) THEN 1 ELSE 0 END) AS changes,
     SUM(CASE WHEN (exclusions = 'null' OR LENGTH(exclusions) = 0) AND (extras = 'null' OR LENGTH(extras) = 0) THEN 1 ELSE 0 END) AS no_changes
     FROM pizza_runner.customer_orders AS co
     JOIN pizza_runner.runner_orders AS ro ON ro.order_id=co.order_id
     WHERE pickup_time != 'null'
     GROUP BY co.customer_id
     ORDER BY co.customer_id;

-- 8. How many pizzas were delivered that had both exclusions and extras?

     SELECT COUNT(pizza_id) 
     FROM pizza_runner.customer_orders AS co
     JOIN pizza_runner.runner_orders AS ro ON ro.order_id=co.order_id
     WHERE pickup_time != 'null' AND (exclusions != 'null' AND LENGTH(exclusions) <> 0) AND (extras != 'null' AND LENGTH(extras) <> 0);

-- 9. What was the total volume of pizzas ordered for each hour of the day?

     SELECT COUNT(pizza_id), DATE_PART('hour',order_time) AS hour 
     FROM pizza_runner.customer_orders AS co
     JOIN pizza_runner.runner_orders AS ro ON ro.order_id=co.order_id
     WHERE pickup_time != 'null'
     GROUP BY DATE_PART('hour',order_time) 
     ORDER BY hour;

-- 10. What was the volume of orders for each day of the week?

     SELECT COUNT(pizza_id),
    TO_CHAR(order_time,'day') as DOW
    FROM pizza_runner.customer_orders as co
      join pizza_runner.runner_orders as ro on ro.order_id=co.order_id
    GROUP BY to_char(order_time,'day') 
    order by DOW;
