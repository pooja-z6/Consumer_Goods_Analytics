# Consumer_Goods_Analytics

#### TASK 1

SELECT  market

FROM dim_customer

WHERE customer = "Atliq Exclusive" and region = "APAC";

#### TASK 2

WITH unique_products AS (

    SELECT 
    
        fiscal_year, 
        
        COUNT(DISTINCT Product_code) as unique_products 
        
    FROM 
    
        fact_sales_monthly
        
    GROUP BY 
    
        fiscal_year
        
)

SELECT 

    up_2020.unique_products as unique_products_2020,
    
    up_2021.unique_products as unique_products_2021,
    
    round((up_2021.unique_products - up_2020.unique_products)/up_2020.unique_products * 100,2) as percentage_change
    
FROM 

    unique_products up_2020
    
CROSS JOIN 

    unique_products up_2021
    
WHERE 

    up_2020.fiscal_year = 2020 
    
    AND up_2021.fiscal_year = 2021;

#### TASK 3

SELECT

segment,

COUNT(DISTINCT product_code) as product_counts

FROM dim_product

GROUP BY segment

ORDER BY product_counts DESC;

#### TASK 4

WITH temp_table AS (

    SELECT 
    
        p.segment,
        
        s.fiscal_year,
        
        COUNT(DISTINCT s.Product_code) as product_count
        
    FROM 
    
        fact_sales_monthly s
        
        JOIN dim_product p 
        
        ON s.product_code = p.product_code
        
    GROUP BY
    
        p.segment,
        
        s.fiscal_year
        
)

SELECT 

    up_2020.segment,
    
    up_2020.product_count as product_count_2020,
    
    up_2021.product_count as product_count_2021,
    
    up_2021.product_count - up_2020.product_count as difference
    
FROM 

    temp_table as up_2020
    
JOIN 

    temp_table as up_2021
    
ON 

    up_2020.segment = up_2021.segment
    
    AND up_2020.fiscal_year = 2020 
    
    AND up_2021.fiscal_year = 2021
    
ORDER BY

    difference DESC;
    
#### TASK 5

SELECT m.product_code,

p.product,

manufacturing_cost

FROM fact_manufacturing_cost m

JOIN dim_product p ON m.product_code = p.product_code

WHERE manufacturing_cost= 

(SELECT min(manufacturing_cost) FROM fact_manufacturing_cost)

or 

manufacturing_cost = 

(SELECT max(manufacturing_cost) FROM fact_manufacturing_cost) 

ORDER BY manufacturing_cost DESC;
    
#### TASK 6

SELECT c.customer_code,

c.customer,

round(AVG(pre_invoice_discount_pct),4) AS average_discount_percentage

FROM fact_pre_invoice_deductions AS d

join dim_customer as c

ON d.customer_code= c.customer_code

where c.market="India" AND fiscal_year="2021"

GROUP BY customer_code

ORDER BY average_discount_percentage DESC

limit 5;

#### TASK 7

WITH Cte AS (

SELECT customer,

monthname(date) AS month,

month(date) AS month_number,

year(date) AS year,

(sold_quantity*gross_price) AS gross_sales_amount

FROM fact_sales_monthly as s

JOIN fact_gross_price as gp

ON s.product_code = gp.product_code

JOIN dim_customer as c

ON s.customer_code = c.customer_code

WHERE customer="Atliq exclusive"

)

SELECT month,year,

concat(round(sum(gross_sales_amount)/1000000,2)," M") AS gross_sales_amount FROM Cte

GROUP BY year,month

ORDER BY year,month_number;

#### TASK 8 

WITH Cte AS (

  SELECT date,month(date_add(date,interval 4 month)) AS period,
  
  fiscal_year,
  
  sold_quantity 
  
FROM fact_sales_monthly

)

SELECT CASE 

   when period/3 <= 1 then "Q1"
   
   when period/3 <= 2 and period/3 > 1 then "Q2"
   
   when period/3 <=3 and period/3 > 2 then "Q3"
   
   when period/3 <=4 and period/3 > 3 then "Q4" END quarter,
   
 round(sum(sold_quantity)/1000000,2) as total_sold_quanity_in_millions FROM Cte
 
WHERE fiscal_year = 2020

GROUP BY quarter

ORDER BY total_sold_quanity_in_millions DESC ;

#### TASK 9

WITH cte AS (

      SELECT c.channel,
      
      sum(s.sold_quantity * g.gross_price) AS total_sales
      
  FROM
  
  fact_sales_monthly s 
  
  JOIN fact_gross_price g ON s.product_code = g.product_code
  
  JOIN dim_customer c ON s.customer_code = c.customer_code
  
  WHERE s.fiscal_year= 2021
  
  GROUP BY c.channel
  
  ORDER BY total_sales DESC
  
)

SELECT 

  channel,
  
  round(total_sales/1000000,2) AS gross_sales_in_millions,
  
  round(total_sales/(sum(total_sales) OVER())*100,2) AS percentage 
  
FROm Cte;

#### TASK 10

WITH Cte AS (

    select division,
    
    s.product_code,
    
    p.product, 
    
    sum(sold_quantity) AS total_sold_quantity,
    
    rank() OVER (partition by division order by sum(sold_quantity) desc) AS rank_order
    
 FROM fact_sales_monthly s
 
 JOIN dim_product p
 
 ON s.product_code = p.product_code
 
 WHERE fiscal_year = 2021
 
 GROUP BY product_code
 
)

SELECT * FROM Cte

WHERE rank_order IN (1,2,3);
