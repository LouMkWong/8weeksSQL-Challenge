**Schema (PostgreSQL v13)**

---
**Query #4**

    SELECT
        TO_CHAR(registration_date, 'w') as weeknum,
        COUNT(runner_id) as num_of_registration

    FROM pizza_runner.runners
    
    GROUP BY TO_CHAR(registration_date, 'w')
    
    ORDER BY 1;

| weeknum | num_of_registration |
| ------- | ------------------- |
| 1       | 2                   |
| 2       | 1                   |
| 3       | 1                   |

---
**Query #5**

    SELECT
        ro.runner_id,
        ROUND(AVG(DATE_PART('minute', ro.pickup_time - co.order_time)) :: NUMERIC, 2) as avg_pickup_time_minutes
        
    FROM customer_orders_cln as co
        JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id

    WHERE ro.pickup_time IS NOT NULL
    
    GROUP BY ro.runner_id;

| runner_id | avg_pickup_time_minutes |
| --------- | ----------------------- |
| 3         | 10.00                   |
| 2         | 23.40                   |
| 1         | 15.33                   |

---
**Query #6**

    WITH pizza_time_cte AS
    	(
      	SELECT
      		co.order_id,
      		COUNT(co.pizza_id) as num_of_pizza,
      		AVG(DATE_PART('minute', ro.pickup_time - co.order_time)) as prep_time_minutes,
      		AVG(DATE_PART('minute', ro.pickup_time - co.order_time)) :: NUMERIC / 
      			COUNT(co.pizza_id) :: NUMERIC as avg_time_per_pizza
      
      	FROM customer_orders_cln as co
      		JOIN runner_orders_cln as ro
      		ON co.order_id = ro.order_id
      
      	WHERE ro.pickup_time IS NOT NULL
      
      	GROUP BY co.order_id
      
      	ORDER BY 2
    	)
      
    SELECT 
        num_of_pizza,
        ROUND(AVG(avg_time_per_pizza), 2) as prep_time_needed_per_pizza
        
    FROM pizza_time_cte
    
    GROUP BY num_of_pizza;

| num_of_pizza | prep_time_needed_per_pizza |
| ------------ | -------------------------- |
| 1            | 12.00                      |
| 2            | 9.00                       |
| 3            | 9.67                       |

---
**Query #7**

    SELECT
        co.customer_id,
        ROUND(AVG(ro.distance), 2) as avg_distance

    FROM customer_orders_cln as co
        JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id

    GROUP BY co.customer_id
    
    ORDER BY 1;

| customer_id | avg_distance |
| ----------- | ------------ |
| 101         | 20.00        |
| 102         | 16.73        |
| 103         | 23.40        |
| 104         | 10.00        |
| 105         | 25.00        |

---
**Query #8**

    SELECT
        MAX(duration) as longest_delivery_time,
        MIN(duration) as shortest_delivery_time,
        MAX(duration) - MIN (duration) as Difference

    FROM runner_orders_cln;

| longest_delivery_time | shortest_delivery_time | difference |
| --------------------- | ---------------------- | ---------- |
| 40                    | 10                     | 30         |

---
**Query #9**

    SELECT 
        runner_id,
        ROUND(AVG(distance/duration*60), 2) as avg_speed_km_per_hour

    FROM runner_orders_cln
    
    GROUP BY runner_id
    
    ORDER BY 1;

| runner_id | avg_speed_km_per_hour |
| --------- | --------------------- |
| 1         | 45.54                 |
| 2         | 62.90                 |
| 3         | 40.00                 |

---
**Query #10**

    SELECT
        ro.runner_id,
        COUNT(co.order_time) as pizza_ordered,
        COUNT(ro.pickup_time) as pizza_delivered,
        ROUND((COUNT(ro.pickup_time) :: NUMERIC / COUNT(co.order_time) :: numeric * 100), 2) as deliver_pct
        
    FROM customer_orders_cln as co
        JOIN runner_orders_cln as ro
        ON co.order_id = ro.order_id

    GROUP BY ro.runner_id;

| runner_id | pizza_ordered | pizza_delivered | deliver_pct |
| --------- | ------------- | --------------- | ----------- |
| 3         | 2             | 1               | 50.00       |
| 2         | 6             | 5               | 83.33       |
| 1         | 6             | 6               | 100.00      |

---

