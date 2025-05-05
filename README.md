# Atliq-consumer-goods
# My Sql code for all Ad Hoc requests


## For 1st Business Question 

         SELECT 
         DISTINCT(market),
         FROM 
         dim_customer
         WHERE
         redion = 'APAC'

## For 2nd Business Question 
         
          SELECT
          (SELECT
          COUNT(DISTINCT dim_product.product)
          FROM
          fact_sales_monthly
          JOIN
          dim_product ON fact_sales_monthly.product_code = dim_product.product_code
          WHERE
          fact_sales_monthly.fiscal year 2020) AS unique_products_2020,
          (SELECT
          COUNT(DISTINCT dim_product.product)
          FROM
          fact_sales_monthly
          JOIN
          dim_product ON fact_sales_monthly.product_code = dim_product.product_code
          WHERE
          fact_sales_monthly.fiscal year = 2021) AS unique_products_2021,
          (((SELECT
          COUNT(DISTINCT dim_product.product)
          FROM
          fact_sales_monthly
          JOIN
          dim_product ON fact_sales_monthly.product_code = dim_product.product_code
          WHERE
          fact_sales_monthly.fiscal year 2021) - (SELECT COUNT(DISTINCT dim_product.product)
          FROM
          fact_sales_monthly
          JOIN
          dim_product ON fact_sales_monthly.product_code = dim_product.product_code
          WHERE
          fact_sales_monthly.fiscal year 2020)) / NULLIF((SELECT
           COUNT(DISTINCT dim_product.product)
          FROM
          fact_sales_monthly
          JOIN
          dim_product ON fact_sales_monthly.product_code = dim_product.product_code
          WHERE
          fact_sales_monthly.fiscal year = 2020),
          0) * 100) AS percentage change;
          
## For 3rd Business Question 
         
          SELECT
          segment, COUNT(DISTINCT (product)) AS product_count
          FROM
          dim_product
          GROUP BY segment
          ORDER BY product_count DESC;


## For 4th Business Question 
         
          SELECT
          dp.segment,
          COUNT(DISTINCT CASE
          WHEN fsm.fiscal year 2020 THEN fsm.product_code END) AS uni_2020,
          COUNT(DISTINCT CASE
          WHEN fsm.fiscal year 2021 THEN fsm.product_code END) AS uni_2021,
          COUNT(DISTINCT CASE
          WHEN fsm.fiscal year 2021 THEN fsm.product_code END) - COUNT(DISTINCT CASE
          WHEN fsm.fiscal year 2020 THEN fsm.product_code END) AS difference
          FROM
          fact_sales_monthly fsm
          JOIN
          dim_product dp ON fsm.product_code = dp.product_code 
          GROUP BY dp.segment
          ORDER BY segment ASC;
     
## For 5th Business Question 

        
          (SELECT
          dp.product_code AS Product_code, dp.product AS Product,
          fmc.manufacturing_cost AS Cost, 'Min Cost' AS Cost_Type
          FROM
          fact_manufacturing_cost fmc
          JOIN
          dim_product dp ON fmc.product_code= dp.product_code 
          ORDER BY fmc.manufacturing_cost ASC
          LIMIT 1) UNION ALL (SELECT
          dp.product_code AS Product_code, dp.product AS Product,
          fmc.manufacturing_cost AS Cost, 'Max Cost' AS Cost Type
          FROM
          fact_manufacturing_cost fmc
          JOIN
          dim_product dp ON fmc.product_code = dp.product_code
          ORDER BY fmc.manufacturing_cost DESC
          LIMIT 1)

## For 6th Business Question 
          
          SELECT
          dc.customer_code,
          dc.customer AS Customer,
          MAX(pid.pre_invoice_discount_pct) AS max_discount_pct
          FROM
          dim_customer dc
          JOIN
          fact_pre_invoice_deductions pid ON pid.customer_code = dc.customer_code
          WHERE
          dc.market 'India'
          GROUP BY dc.customer_code, dc.customer
          ORDER BY max_discount_pct DESC
          LIMIT 5;

## For 7th Business Question
          SELECT
          MONTHNAME(fsm.date) AS Month,
          fsm. fiscal_year,
          SUM(fsm.sold_quantity * fgp.gross_price) AS gross_sales_amount
          FROM
          fact_sales_monthly fsm
          JOIN
          fact_gross_price fgp ON fgp.product_code = fsm.product_code
          AND fgp.fiscal_year = fsm.fiscal_year
          JOIN
          dim_customer dc ON fsm.customer_code = dc.customer_code
          WHERE
          dc.customer = ‘Atliq Exclusive’
          GROUP BY
          MONTHNAME(fsm.date),
          fsm.fiscal_year,
          MONTH(fsm.date)
          ORDER BY
          fsm.fiscal_year,
          MONTH(fsm. date);

## For 8th Business Question
          SELECT
          CASE
          WHEN MONTH(date) BETWEEN 9 AND 11 THEN 'Q1'
          WHEN MONTH(date}) IN (12, 1, 2) THEN 'Q2'
          WHEN MONTH(date) BETWEEN 3 AND 5 THEN 'Q3'
          WHEN MONTH(date) BETWEEN 6 AND 8 THEN 'Q4'
          END AS fiscal_quarter,
          SUM(sold_quantity) AS total_sold_quantity
          FROM
          fact_sales_monthly
          WHERE
          fiscal_year = 2020
          GROUP BY
          fiscal_quarter
          ORDER BY
          fiscal_quarter;

## For 9th Business Question
          WITH channel_sales AS (
          SELECT
          dc.channel,
          ROUND(SUM( fsm.sold_quantity * fgp.gross_price) / 1000000, 2) AS gross_sales_mln
          FROM
          dim_customer dc
          JOIN
          fact_sales_monthly fsm
          ON fsm.customer_code = dc.customer_code
          AND fsm.fiscal_year = 2021
          JOIN
          fact_gross_price fgp
          ON fgp.product_code = fsm.product_code
          AND fgp.fiscal_year = fsm.fiscal_year
          GROUP BY
          dc.channel
          }
          SELECT
          channel,
          gross_sales_mln,
          CONCAT( ROUND(gross_sales_mln * 100 / SUM(gross_sales_mln) OVER (), 1), '%') AS
          percentage_contribution
          FROM
          channel_sales
          ORDER BY
          gross_sales_mln DESC;

## For 10th Business Question
          WITH product_sales AS (
          SELECT
          dp.division,
          dp.product_code,
          dp.product,
          SUM( fsm.sold_quantity) AS total_sold_quantity,
          RANK() OVER (PARTITION BY dp.division ORDER BY SUM(fsm.sold_quantity) DESC) AS rank_order
          FROM
          dim_product dp
          JOIN
          fact_sales_monthly fsm ON dp.product_code = fsm.product_code
          WHERE
          fsm.fiscal_year = 2021
          GROUP BY
          dp.division, dp.product_code, dp.product
          )
          SELECT
          division,
          product_code,
          product,
          total_sold_quantity,
          rank_order
          FROM
          product_sales
          WHERE
          rank_order <= 3
          ORDER BY
          division,
          rank_order;
