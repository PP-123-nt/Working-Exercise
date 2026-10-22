## **Regional VTB**

  ```sql
WITH act AS (
  SELECT region_id,
         SUM(net_revenue) AS actual_revenue,
         SUM(units)       AS actual_units,
         SUM(gross_revenue) AS gross_revenue
  FROM v_deal_enriched
  WHERE quarter = '2024-Q3'
  GROUP BY region_id
),
plan AS (
  SELECT region_id,
         SUM(budget_revenue) AS budget_revenue,
         SUM(budget_units)   AS budget_units
  FROM budget WHERE quarter = '2024-Q3'
  GROUP BY region_id
),
regional_variance AS (              -- Step 1: Calculate the base variance for each region
  SELECT r.region_name,
         p.budget_revenue                    AS budget,
         a.actual_revenue                    AS actual,
         a.actual_revenue - p.budget_revenue AS variance,
         ROUND(100.0 * (a.actual_revenue - p.budget_revenue) / p.budget_revenue, 1) AS variance_pct
  FROM plan p
  JOIN regions r USING (region_id)     -- similar to JOIN regions r ON r.region_id = p.region_id
  JOIN act     a USING (region_id)     -- similar to JOIN act a ON a.region_id = r.region_id
)
SELECT region_name,                 -- Step 2: Add the gap % column using a window function
       budget,
       actual,
       variance,
       variance_pct,
       ROUND(100.0 * variance / SUM(variance) OVER (), 1) AS gap_pct_of_total
FROM regional_variance
ORDER BY variance_pct;
  ```

**Output**

| region_name | budget  | actual  | variance | variance_pct | gap_pct_of_total |
| :---------- | ------: | ------: | -------: | -----------: | ---------------: |
| South       | 3800000 | 3326750 |  -473250 |        -12.5 |             79.9 |
| North       | 4200000 | 4110750 |   -89250 |         -2.1 |             15.1 |
| West        | 3500000 | 3470500 |   -29500 |         -0.8 |                5 |

Almost 80% gap in 2024-Q3 is driven by South region
