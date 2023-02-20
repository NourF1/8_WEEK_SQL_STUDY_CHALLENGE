--1 How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

      SELECT 
                date_trunc('week',"registration_date") + 4 AS registration_week,
                COUNT("runner_id") AS runner_signup
      FROM runners
      GROUP BY date_trunc('week',"registration_date")+4;

--2 What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

      select "runner_id",
      avg(DATEDIFF(minute, "order_time", "pickup_time")) AS pickup_minutes
      from customer_orders
      join runner_orders on runner_orders."order_id"=customer_orders."order_id" 
      where "pickup_time" <> 'null'
      group by "runner_id"

--3 Is there any relationship between the number of pizzas and how long the order takes to prepare?

      with cte as (
      select customer_orders."order_id", max(DATEDIFF(minute, "order_time", "pickup_time"))AS prep_time,count("pizza_id") as No_of_pizzas
      from customer_orders
      join runner_orders on runner_orders."order_id"=customer_orders."order_id" 
      where "pickup_time" <> 'null'
      group by CUSTOMER_ORDERS."order_id")
      select No_of_pizzas , avg(prep_time) 
      from cte
      group by No_of_pizzas

--4 What was the average distance travelled for each customer?
      
      with cte as(
      select replace("distance",'km','') as dist ,"order_id"
      from runner_orders
      where "distance"<> 'null')
      select "customer_id",avg(dist) from cte
      join customer_orders on cte."order_id"=customer_orders."order_id"
      group by "customer_id"
      order by "customer_id"

--5 What was the difference between the longest and shortest delivery times for all orders?

      SELECT max(regexp_replace("duration",'[^0-9]',''))-min(regexp_replace("duration",'[^0-9]','')) as max_diff_deliev_time
      FROM runner_orders
      WHERE "pickup_time" <> 'null'

--6 What was the average speed for each runner for each delivery and do you notice any trend for these values?

      WITH cte as (
      SELECT "order_id","runner_id",replace("distance",'km','') as dist ,regexp_replace("duration",'[^0-9]','') as time       FROM runner_orders
      WHERE dist <> 'null' and time<>'null')
      SELECT "order_id","runner_id",dist/time as average_speed
      FROM cte
      ORDER BY "runner_id"

--7 What is the successful delivery percentage for each runner?

      WITH cte as (      SELECT "runner_id", count("order_id") as IP,
      SUM(
      CASE
      WHEN "duration"='null' THEN 0 
      ELSE 1
      END) AS OP
      FROM runner_orders
      GROUP BY "runner_id")
      SELECT "runner_id",(OP/IP)*100 as effeciency_percent       
      FROM cte



      
      
