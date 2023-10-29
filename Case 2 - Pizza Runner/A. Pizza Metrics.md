**Schema (PostgreSQL v13)**


---
**Query #4**

    SELECT
     	COUNT(*) as Number_of_pizza_ordered
     
     FROM customer_orders_cln;

| number_of_pizza_ordered |
| ----------------------- |
| 14                      |

---
**Query #5**

    SELECT
     	COUNT(DISTINCT order_id) as Number_of_Orders
     
     FROM customer_orders_cln;

| number_of_orders |
| ---------------- |
| 10               |

---
**Query #6**

    SELECT
        runner_id,
        COUNT(order_id)
        
     FROM runner_orders_cln
     
     WHERE cancellation ISNULL
     GROUP BY runner_id;

| runner_id | count |
| --------- | ----- |
| 1         | 4     |
| 2         | 3     |
| 3         | 1     |

---
**Query #7**

    SELECT
        co.pizza_id,
        pn.pizza_name,
        COUNT(co.pizza_id) as number_of_orders
        
    FROM customer_orders_cln as co
    	LEFT JOIN pizza_runner.pizza_names as pn
        ON co.pizza_id = pn.pizza_id
        LEFT JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id
            
     WHERE ro.cancellation ISNULL
      
     GROUP BY co.pizza_id, pn.pizza_name;

| pizza_id | pizza_name | number_of_orders |
| -------- | ---------- | ---------------- |
| 1        | Meatlovers | 9                |
| 2        | Vegetarian | 3                |

---
**Query #8**

    SELECT
        co.customer_id,
        pn.pizza_name,
        COUNT(co.pizza_id)

     FROM customer_orders_cln as co
     	LEFT JOIN pizza_runner.pizza_names as pn
      	ON co.pizza_id = pn.pizza_id
            
     GROUP BY co.customer_id, pn,pizza_name
     
     ORDER BY 1;

| customer_id | pizza_name | count |
| ----------- | ---------- | ----- |
| 101         | Meatlovers | 2     |
| 101         | Vegetarian | 1     |
| 102         | Meatlovers | 2     |
| 102         | Vegetarian | 1     |
| 103         | Meatlovers | 3     |
| 103         | Vegetarian | 1     |
| 104         | Meatlovers | 3     |
| 105         | Vegetarian | 1     |

---
**Query #9**

    SELECT
        co.order_id,
        co.customer_id,
        COUNT(co.order_id) as most_pizzas_ordered_in_one_single_order
    
    FROM customer_orders_cln as co
    	LEFT JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id
        
    WHERE ro.cancellation ISNULL
    
    GROUP BY co.order_id, co.customer_id
    
    ORDER BY 3 DESC
    
    LIMIT 1;

| order_id | customer_id | most_pizzas_ordered_in_one_single_order |
| -------- | ----------- | --------------------------------------- |
| 4        | 103         | 3                                       |

---
**Query #10**

    SELECT
        co.customer_id,
        COUNT(co.pizza_id) as number_of_pizza_with_at_least_1_change
        
    FROM customer_orders_cln as co
    	LEFT JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id
        
    WHERE ro.cancellation ISNULL 
    	AND (co.exclusions IS NOT NULL OR co.extras IS NOT NULL)
    
    GROUP BY co.customer_id;

| customer_id | number_of_pizza_with_at_least_1_change |
| ----------- | -------------------------------------- |
| 103         | 3                                      |
| 104         | 2                                      |
| 105         | 1                                      |

---
**Query #11**

    SELECT
    	COUNT(co.pizza_id) as pizza_delievered_with_exclusion_and_extra
        
    FROM customer_orders_cln as co
    	JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id
        
    WHERE ro.cancellation ISNULL
    	AND co.exclusions IS NOT NULL
        AND co.extras IS NOT NULL;

| pizza_delievered_with_exclusion_and_extra |
| ----------------------------------------- |
| 1                                         |

---
**Query #12**

    SELECT
        DATE_PART('HOUR', order_time) as Hour_of_Day, 
        COUNT(pizza_id)

    FROM customer_orders_cln
    
    GROUP BY DATE_PART('HOUR', order_time)
                       
    ORDER BY 1;

| hour_of_day | count |
| ----------- | ----- |
| 11          | 1     |
| 13          | 3     |
| 18          | 3     |
| 19          | 1     |
| 21          | 3     |
| 23          | 3     |

---
**Query #13**

    SELECT
        TO_CHAR(order_time, 'day') as day_of_week,
        COUNT(pizza_id)

    FROM customer_orders_cln
    
    GROUP BY TO_CHAR(order_time, 'day');

| day_of_week | count |
| ----------- | ----- |
| wednesday   | 5     |
| thursday    | 3     |
| friday      | 1     |
| saturday    | 5     |

---
