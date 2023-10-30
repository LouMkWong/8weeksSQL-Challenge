**Schema (PostgreSQL v13)**

---
**Query #4**

    WITH pizzatopping_cte AS
    
    (
      SELECT
      	pr.pizza_id,
      	pr.toppings,
      	pn.pizza_name,
      	pt.topping_name
      
      FROM pizza_recipes_cln as pr
      	JOIN pizza_runner.pizza_names as pn
      	ON pr.pizza_id = pn.pizza_id
      	JOIN pizza_runner.pizza_toppings as pt
      	ON pr.toppings = pt.topping_id
    )
    
    SELECT
    	pizza_name,
        array_agg(topping_name) as pizza_toppings
        
    FROM pizzatopping_cte
    
    GROUP BY pizza_name;

| pizza_name | pizza_toppings                                                 |
| ---------- | -------------------------------------------------------------- |
| Meatlovers | Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami |
| Vegetarian | Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce          |

---
**Query #5**

    WITH extras_cte as
    
    (
    SELECT
    	order_id,
        unnest(string_to_array(extras, ',')) :: INT as extras
      	
    FROM customer_orders_cln 
    )
    
    SELECT
    	
        pt.topping_name as most_ordered_as_extra,
        COUNT(pt.topping_name) as number_of_time_ordered_as_extra
        
    FROM extras_cte as e
        JOIN pizza_runner.pizza_toppings as pt
        ON e.extras = pt.topping_id
        
    GROUP BY pt.topping_name
    
    ORDER BY 2 DESC
    
    LIMIT 1;

| most_ordered_as_extra | number_of_time_ordered_as_extra |
| --------------------- | ------------------------------- |
| Bacon                 | 4                               |

---
**Query #6**

    WITH excl_cte as
    
    (
    SELECT
    	order_id,
        unnest(string_to_array(exclusions, ',')) :: INT as exclusions
      	
    FROM customer_orders_cln 
    )
    
    SELECT
        pt.topping_name as most_excluded,
        COUNT(pt.topping_name) as number_of_time_excluded
        
    FROM excl_cte as ex
        JOIN pizza_runner.pizza_toppings as pt
        ON ex.exclusions = pt.topping_id
        
    GROUP BY pt.topping_name
    
    ORDER BY 2 DESC
    
    LIMIT 1;

| most_excluded | number_of_time_excluded |
| ------------- | ----------------------- |
| Cheese        | 4                       |

---
**Query #7**

    alter table customer_orders_cln ADD COLUMN record_id SERIAL;

There are no results to be displayed.

---
**Query #8**

    WITH ex_cte AS
    (
      SELECT 
          record_id,
          pizza_id,
          unnest(string_to_array(extras, ',')) :: INT as extras,
          unnest(string_to_array(exclusions, ',')) :: INT as exclusions
    
      FROM customer_orders_cln 
    ),
    
    extra_cte AS
    (
      SELECT
          ex.record_id,
          array_agg(pt.topping_name) as extras_item
      
      FROM ex_cte as ex
          JOIN pizza_runner.pizza_toppings as pt
          ON ex.extras = pt.topping_id
      
      GROUP BY ex.record_id
    ),
    
    exclusion_cte AS
    (
      SELECT
          ex.record_id,
          array_agg(pt.topping_name) as exclusion_item
      
      FROM ex_cte as ex
        JOIN pizza_runner.pizza_toppings as pt
        ON ex.exclusions = pt.topping_id
      
      GROUP BY ex.record_id
    ),
    
    all_cte AS
    (
      SELECT 
          co.record_id,
          pn.pizza_name,
          co.extras,
          co.exclusions
      FROM customer_orders_cln as co
          JOIN pizza_runner.pizza_names as pn
          ON co.pizza_id = pn.pizza_id
      ),
      
    fnl AS 
    
    (
    SELECT 
    	a.record_id,
        a.pizza_name,
        array_to_string(e.extras_item, ', ') :: TEXT as extras,
        array_to_string(ex.exclusion_item, ', ') :: TEXT as exclusions
    FROM all_cte as a 
    	LEFT JOIN extra_cte as e
        ON a.record_id = e.record_id
        LEFT JOIN exclusion_cte as ex
        ON a.record_id = ex.record_id
    )
    
    SELECT
    	record_id,
        CASE
        	WHEN extras IS NULL and exclusions IS NULL
            THEN pizza_name
            WHEN extras IS NULL and exclusions IS NOT NULL
            THEN CONCAT(pizza_name, ' : ', '** Exclude ', exclusions)
            WHEN extras IS NOT NULL and exclusions IS NULL
            THEN CONCAT(pizza_name, ' : ', '* Extra ', extras)
            WHEN extras IS NOT NULL and exclusions IS NOT NULL
            THEN CONCAT(pizza_name, ' : ', '* Extra ', extras, ' ** ', 'Exclude ', exclusions)
        END as pizza_order_instructions
    FROM fnl
    ORDER BY 1;

| record_id | pizza_order_instructions                                           |
| --------- | ------------------------------------------------------------------ |
| 1         | Meatlovers                                                         |
| 2         | Meatlovers                                                         |
| 3         | Meatlovers                                                         |
| 4         | Vegetarian                                                         |
| 5         | Meatlovers : ** Exclude Cheese                                     |
| 6         | Meatlovers : ** Exclude Cheese                                     |
| 7         | Vegetarian : ** Exclude Cheese                                     |
| 8         | Meatlovers : * Extra Bacon                                         |
| 9         | Vegetarian                                                         |
| 10        | Vegetarian : * Extra Bacon                                         |
| 11        | Meatlovers                                                         |
| 12        | Meatlovers : * Extra Bacon, Chicken ** Exclude Cheese              |
| 13        | Meatlovers                                                         |
| 14        | Meatlovers : * Extra Bacon, Cheese ** Exclude BBQ Sauce, Mushrooms |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
