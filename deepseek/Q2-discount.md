## Create view to store global average discount rate by product

  ```sql
CREATE VIEW v_product_benchmarks AS
WITH region_product_actuals AS (
    SELECT 
      region_name, 
      product_name,
      SUM(gross_revenue) AS gross,
      SUM(discount_value) AS disc_cost
    FROM v_deal_enriched
    GROUP BY region_name, product_name
),
product_benchmarks AS (   -- Calculate global average discount rate by product
    SELECT 
      product_name,
      SUM(disc_cost) / SUM(gross) AS global_avg_prod_disc_rate
    FROM region_product_actuals
    GROUP BY product_name
)
SELECT 
    r.region_name AS region,
    r.product_name AS product,
    r.gross,
    r.disc_cost,
    ROUND(100.0 * r.disc_cost / r.gross, 1) AS regional_disc_pct,
    ROUND(100.0 * p.global_avg_prod_disc_rate, 1) AS benchmark_prod_disc_pct,   
    ROUND(r.gross * (p.global_avg_prod_disc_rate - (r.disc_cost / r.gross)), 2) AS margin_variance
FROM region_product_actuals r
JOIN product_benchmarks p USING (product_name);
  ```

**Calculate South region revenue variance**
  ```sql
SELECT * FROM v_product_benchmarks WHERE region_name = 'South';
  ```

| region | product | gross | disc_cost | regional_disc_pct | benchmark_prod_disc_pct | margin_variance |
| :--- | :--- | ---: | ---: | ---: | ---: | ---: |
| South | Analytics Add-on | 1275000 | 217250 | 17 | 11.4 | -71850.96 |
| South | Core Platform | 1750000 | 305250 | 17.4 | 11.9 | -97278.99 |
| South | Enterprise Suite | 1000000 | 175750 | 17.6 | 11.7 | -59032.44 |

South region product level discount % is higher than global average product discount %

**If we bring South region discount rates down just to the global average, we would instantly recover $228,000 in net revenue. This isn't a product strategy; this is a process leak. We need to look at who is authorizing these blanket discounts.**
