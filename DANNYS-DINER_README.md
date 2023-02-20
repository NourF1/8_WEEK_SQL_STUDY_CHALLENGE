# 8_WEEK_SQL_STUDY_CHALLENGE


                                         --Case Study #1: Danny's Diner

--1 What is the total amount each customer spent at the restaurant?

     SELECT
     "customer_id",
     SUM("price") as total_paid
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id" 
     GROUP BY "customer_id";


--2 How many days has each customer visited the restaurant?

     SELECT  "customer_id",
     COUNT(distinct"order_date") as visited_days_no
     FROM sales
     GROUP BY "customer_id";


--3 What was the first item FROM the menu purchased by each customer?
     WITH cte  as (SELECT
     "customer_id",
     "product_name",
      rank() over( partition by  s."customer_id" order by s."order_date")as rnk 
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id")
     SELECT "customer_id",
     "product_name"
     FROM cte 
     WHERE rnk = 1;


---4 What is the most purchased item on the menu and how many times was it purchased by all customers?

      SELECT "product_name",
     COUNT(s."product_id")as n0_purchased
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     GROUP BY s."product_id","product_name"
     order by n0_purchased DESC
     LIMIT 1;


--5 Which item was the most popular for each customer?

     WITH cte as (SELECT s."customer_id","product_name", COUNT(s."product_id")as n0_purchased, rank () over(partition by "customer_id"order by n0_purchased DESC) as rnk
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     GROUP BY s."product_id","product_name", s."customer_id"
     order by s."customer_id",n0_purchased DESC)
     SELECT "customer_id",
     "product_name"
     FROM cte 
     WHERE rnk=1
     GROUP BY "customer_id","product_name";


--6 Which item was purchased first by the customer after they became a member?

     WITH cte as (SELECT s."customer_id","product_name","order_date",rank() over(partition by s."customer_id" order by "order_date") as rnk
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     JOIN members as mem on mem."customer_id"=s."customer_id"
      WHERE "order_date" >= "JOIN_date")
     SELECT "customer_id","product_name"
     FROM cte 
     WHERE rnk = 1
     GROUP BY "customer_id","product_name";


--7 Which item was purchased just before the customer became a member?

     WITH cte as (SELECT s."customer_id","product_name","order_date",rank() over(partition by s."customer_id" order by "order_date"DESC) as rnk
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     JOIN members as mem on mem."customer_id"=s."customer_id"
     WHERE "order_date" < "JOIN_date")
     SELECT "customer_id",
     "product_name"
     FROM cte 
     WHERE rnk = 1
     GROUP BY "customer_id","product_name";


--8 What is the total items and amount spent for each member before they became a member?

     SELECT s."customer_id",
      COUNT(s."product_id")no_of_products,
      SUM("price") total_spent
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     JOIN members as mem on mem."customer_id"=s."customer_id"
     WHERE "order_date" < "JOIN_date"
     GROUP BY  s."customer_id";


--9 If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

     SELECT "customer_id",
     SUM(CASE
     WHEN "product_name"='sushi' then "price" * 10 * 2
     ELSE "price" * 10 
     END) as points
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     GROUP BY "customer_id";


--10 In the first week after a customer JOINs the program (including their JOIN date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the END of January?


     SELECT s."customer_id",
     SUM(CASE
     WHEN "order_date"between "JOIN_date" and dateadd('day' ,6,"JOIN_date") then "price" * 10 * 2
     WHEN "product_name"='sushi' then "price" * 10 * 2
     ELSE "price" * 10 
     END) as points
     FROM sales as s
     JOIN menu as m on m."product_id"=s."product_id"
     JOIN members as mem on mem."customer_id"=s."customer_id"
     WHERE "order_date"<= '2021-01-31'
     GROUP BY s."customer_id";
