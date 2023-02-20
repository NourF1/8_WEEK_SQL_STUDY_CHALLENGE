                                                                       
                                                             Case Study #2: PIZZA RUNNER C
                                                              C. Ingredient Optimisation


--1 What are the standard ingredients for each pizza?

      WITH cte as (SELECT DISTINCT
      "pizza_id",
      top.value as "topping_id"
      FROM pizza_recipes
      LEFT JOIN lateral split_to_table("toppings",',') as top
      JOIN pizza_toppings as pt on pt."topping_id"="topping_id"
      ORDER BY "pizza_id")
      SELECT cte."pizza_id",
      "pizza_name",
      listagg(DISTINCT "topping_name",',')as toppings
      FROM cte 
      JOIN pizza_toppings as ptop on ptop."topping_id"=cte."topping_id" 
      JOIN pizza_names as pn on pn."pizza_id"=cte."pizza_id"
      GROUP BY cte."pizza_id",PN."pizza_name"
      ORDER BY cte."pizza_id";

--2 What was the most commonly added extra?

      SELECT
      ext.value as xtras_id ,count (xtras_id) as Num,"topping_name"       FROM customer_orders
      LEFT JOIN lateral split_to_table("extras",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=xtras_id
      WHERE value <> 'null' and value <> ''
      GROUP BY xtras_id,"topping_name"
      ORDER BY num desc
      limit 1;


--3 What was the most common exclusion?

      SELECT
      exc.value as exclusive_id ,count (exclusive_id) as Num,"topping_name"       FROM customer_orders
      LEFT JOIN lateral split_to_table("exclusions",',') as exc  
      JOIN pizza_toppings as pt on pt."topping_id"=exclusive_id
      WHERE value <> 'null' and value <> ''
      GROUP BY exclusive_id,"topping_name"
      ORDER BY num desc
      limit 1;


/*4Generate an order item for each record in the customers_orders table in the format of one of the following:
.Meat Lovers
.Meat Lovers - Exclude Beef
.Meat Lovers - Extra Bacon*/

      WITH XT as (
      SELECT
      co."order_id" ,
      co."pizza_id",
      co."extras",
      "topping_id",
      listagg(DISTINCT"topping_name",',') as xtras
      FROM customer_orders as co
      LEFT JOIN lateral split_to_table("extras",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=ext.value
      WHERE value <> 'null' and value <> ''
      GROUP BY co."order_id" ,co."pizza_id",CO."extras","topping_id"),
      XC as (SELECT
      co."order_id" ,
      co."pizza_id",
      co."exclusions",
      "topping_id",
      listagg(DISTINCT"topping_name",',') as exclu
      FROM customer_orders as co
      LEFT JOIN lateral split_to_table("exclusions",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=ext.value
      WHERE value <> 'null' and value <> ''
      GROUP BY co."order_id" ,co."pizza_id",co."exclusions","topping_id"
      )
      SELECT
      co."order_id",
      CONCAT(CASE
       WHEN pn."pizza_name"='Meatlovers' THEN 'Meat Lovers'
       ELSE pn."pizza_name"
      END,
      COALESCE(' - Extra '|| xtras,''),
      COALESCE(' - Exclude '|| exclu,'')) as Order_in_detail
      FROM customer_orders as co
      LEFT JOIN xt on co."order_id"=xt."order_id" and co."pizza_id"=xt."pizza_id" and co."extras"=xt."extras"
      LEFT JOIN xc on co."order_id"=xc."order_id" and co."pizza_id"=xc."pizza_id" and co."exclusions"=xc."exclusions"
      JOIN pizza_names as pn on pn."pizza_id"=co."pizza_id"
      ORDER BY co."order_id",Order_in_detail

/*--5 Generate an alphabetically ordered comma separated ingredient list for each pizza order       FROM the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"*/

      WITH top as (SELECT
      co."order_id",co."pizza_id",pt."topping_id",pt."topping_name" as toppings        FROM customer_orders as co 
      JOIN pizza_recipes as pr on pr."pizza_id"=co."pizza_id"
      LEFT JOIN lateral split_to_table("toppings",',') as top
      JOIN pizza_toppings as pt on pt."topping_id"=top.value
      ),
      XT as (
      SELECT
      co."order_id" ,
      co."pizza_id",
      co."extras",
      "topping_id" as add,
      listagg(DISTINCT"topping_name",',') as xtras
      FROM customer_orders as co
      LEFT JOIN lateral split_to_table("extras",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=ext.value
      WHERE value <> 'null' and value <> ''
      GROUP BY co."order_id" ,co."pizza_id",CO."extras","topping_id"),
      XC as (SELECT
      co."order_id" ,
      co."pizza_id",
      co."exclusions",
      listagg(DISTINCT"topping_name",',') as exclu,"topping_id" as rem
      FROM customer_orders as co
      LEFT JOIN lateral split_to_table("exclusions",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=ext.value
      WHERE value <> 'null' and value <> ''
      GROUP BY co."order_id" ,co."pizza_id",co."exclusions","topping_id"
      ),
      orders_w_ext_exc as (
      SELECT 
      top."order_id",
      top."topping_id",
      top."pizza_id",
      toppings
      FROM top  
      LEFT JOIN xc on top."order_id"=xc."order_id" and top."pizza_id"=xc."pizza_id" and top."topping_id"=xc.rem
      WHERE xc.rem is null

      UNION ALL

      SELECT 
      "order_id",
      "pizza_id",
      add,
      xtras
      FROM xt),
      ing_overall as (
          SELECT 
      oxc."order_id",
      oxc."pizza_id",
      oxc.toppings,
      "pizza_name",
      count("topping_id") as N
      FROM orders_w_ext_exc as oxc
      JOIN pizza_names as pn on pn."pizza_id"=oxc."pizza_id"
      GROUP BY oxc."order_id",oxc.toppings,"pizza_name",oxc."pizza_id"
      ORDER BY oxc."order_id",oxc.toppings,"pizza_name",oxc."pizza_id"
      ),
      final as (
      SELECT 
      "order_id",
      "pizza_name",
      listagg(DISTINCT CASE
      WHEN n>1  THEN n || 'x' || toppings
      ELSE toppings
      END,',') as ing
      FROM ing_overall
      GROUP BY "order_id","pizza_name")
      SELECT 
      "order_id",
      "pizza_name" || ' : ' || ing as order_ing
      FROM final
      ORDER BY "order_id"


--6 What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

      WITH top as (
          SELECT
      co."order_id",
      co."pizza_id",
      pt."topping_id",
      pt."topping_name" as toppings        
      FROM customer_orders as co 
      JOIN pizza_recipes as pr on pr."pizza_id"=co."pizza_id"
      LEFT JOIN lateral split_to_table("toppings",',') as top
      JOIN pizza_toppings as pt on pt."topping_id"=top.value
      ),
      XT as (
      SELECT
      co."order_id" ,
      co."pizza_id",
      co."extras",
      "topping_id" as add,
      listagg(DISTINCT"topping_name",',') as xtras
      FROM customer_orders as co
      LEFT JOIN lateral split_to_table("extras",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=ext.value
      WHERE ext.value <> 'null' and  length(ext.value)>0
      GROUP BY co."order_id" ,co."pizza_id",CO."extras","topping_id"),
      XC as (      SELECT
      co."order_id" ,
      co."pizza_id",
      co."exclusions",
      listagg(DISTINCT"topping_name",',') as exclu,"topping_id" as rem
      FROM customer_orders as co
      LEFT JOIN lateral split_to_table("exclusions",',') as ext 
      JOIN pizza_toppings as pt on pt."topping_id"=ext.value
      WHERE ext.value <> 'null' and length(ext.value)>0
      GROUP BY co."order_id" ,co."pizza_id",co."exclusions","topping_id"
      ),
      orders_w_ext_exc as (
      SELECT 
      top."order_id",
      top."topping_id",
      top."pizza_id",
      toppings
      FROM top  
      LEFT JOIN xc on top."order_id"=xc."order_id" and top."pizza_id"=xc."pizza_id" and top."topping_id"=xc.rem
      WHERE xc.rem is null

      UNION ALL

      SELECT 
      "order_id",
      "pizza_id",
      add,
      xtras
      FROM xt)
      SELECT 
      oxc.toppings,
      count("topping_id") as N
      FROM orders_w_ext_exc as oxc
      JOIN runner_orders as ro on ro."order_id"= oxc."order_id"
      WHERE ro."pickup_time" <>'null' and "pickup_time" <> ''
      GROUP BY oxc.toppings
      ORDER BY N desc , TOPPINGS
