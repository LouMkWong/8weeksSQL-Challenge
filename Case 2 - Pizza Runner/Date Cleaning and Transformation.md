**Schema (PostgreSQL v13)**


---

**Query #1**

    CREATE TEMP TABLE customer_orders_cln AS
    
    SELECT 
    	order_id,
        customer_id,
        pizza_id,
        CASE
        	WHEN exclusions = 'null' OR exclusions = '' THEN null
            ELSE exclusions
            END AS exclusions,
        CASE
        	WHEN extras = 'null' OR extras = '' THEN null
            ELSE extras
            END AS extras,
        order_time
        
    FROM pizza_runner.customer_orders;

There are no results to be displayed.

---
**Query #2**

    CREATE TEMP TABLE runner_orders_cln AS
    
    SELECT
    	order_id,
        runner_id,
        CASE
        	WHEN pickup_time = 'null' OR pickup_time = '' THEN null
            ELSE pickup_time
            END :: TIMESTAMP AS pickup_time,
        CASE
        	WHEN distance = 'null' OR distance = '' THEN null
            ELSE TRIM ('km' FROM distance)
            END :: NUMERIC distance,
        CASE
        	WHEN duration = 'null' OR duration = '' THEN null
            ELSE SUBSTRING (duration, 1, 2)
            END :: INT AS duration,
        CASE
        	WHEN cancellation = 'null' OR cancellation = '' THEN null
            ELSE cancellation
            END AS cancellation
            
     FROM pizza_runner.runner_orders;

There are no results to be displayed.

---
**Query #3**

    CREATE TEMP TABLE pizza_recipes_cln AS
     
     SELECT 
     	pizza_id,
        UNNEST(string_to_array(toppings, ',')) :: INT as toppings
     
     FROM pizza_runner.pizza_recipes;

There are no results to be displayed.

