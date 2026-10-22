## Create view to store global average discount rate by product

From Q1, we identified almost 80% gap in 2024-Q3 is driven by South region. Now we are looking into the gap by analyzing discount %.
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

If we bring South region discount rates down just to the global average, we would instantly recover **$228,000 in net revenue**. This isn't a product strategy; this is a process leak. We need to look at who is authorizing these blanket discounts.


**Check discount % by rep**
  ```sql
SELECT 
    rep_name, region_name, SUM(gross_revenue) AS gross$, SUM(discount_value) AS disc$, 
    ROUND(disc$ / gross$ * 100.0,1) AS disc_rate
FROM v_deal_enriched
GROUP BY rep_name, region_name,
ORDER BY rep_name;
  ```
| rep_name | region_name | gross$ | disc$ | disc_rate |
| :--- | :--- | ---: | ---: | ---: |
| Anika Rao | North | 1200000 | 107750 | 9 |
| Ben Okafor | North | 1250000 | 113250 | 9.1 |
| Carla Mendes | North | 925000 | 78750 | 8.5 |
| Daniel Park | North | 1150000 | 114500 | 10 |
| Emma Rossi | West | 950000 | 85000 | 8.9 |
| Farid Haddad | West | 900000 | 80250 | 8.9 |
| Grace Lim | West | 1000000 | 97250 | 9.7 |
| Hugo Silva | West | 950000 | 67000 | 7.1 |
| Ishan Verma | South | 1125000 | 197000 | 17.5 |
| Julia Novak | South | 1100000 | 193500 | 17.6 |
| Kwame Asante | South | 950000 | 167500 | 17.6 |
| Lena Fischer | South | 850000 | 140250 | 16.5 |
