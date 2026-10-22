## **Create Tables**
  
  ```sql
-- create tables for regional_fbp.db
CREATE TABLE regions (
  region_id      INTEGER PRIMARY KEY,
  region_name    VARCHAR NOT NULL UNIQUE,
  sales_director VARCHAR NOT NULL
);

CREATE TABLE sales_reps (
  rep_id    INTEGER PRIMARY KEY,
  rep_name  VARCHAR NOT NULL,
  region_id INTEGER NOT NULL REFERENCES regions(region_id),
  hire_date DATE    NOT NULL
);

CREATE TABLE customers (
  customer_id   VARCHAR PRIMARY KEY,
  customer_name VARCHAR NOT NULL,
  region_id     INTEGER NOT NULL REFERENCES regions(region_id)
);

CREATE TABLE products (
  product_id      VARCHAR PRIMARY KEY,
  product_name    VARCHAR NOT NULL,
  list_unit_price DECIMAL(12,2) NOT NULL
);

CREATE TABLE budget (
  region_id      INTEGER NOT NULL REFERENCES regions(region_id),
  product_id     VARCHAR NOT NULL REFERENCES products(product_id),
  quarter        VARCHAR NOT NULL,
  budget_revenue DECIMAL(14,2) NOT NULL,
  budget_units   INTEGER       NOT NULL,
  PRIMARY KEY (region_id, product_id, quarter)
);

CREATE TABLE deals (
  deal_id       VARCHAR PRIMARY KEY,
  rep_id        INTEGER NOT NULL REFERENCES sales_reps(rep_id),
  customer_id   VARCHAR NOT NULL REFERENCES customers(customer_id),
  product_id    VARCHAR NOT NULL REFERENCES products(product_id),
  close_date    DATE    NOT NULL,
  units         INTEGER NOT NULL CHECK (units > 0),
  gross_revenue DECIMAL(14,2) NOT NULL,
  discount_pct  DECIMAL(5,4)  NOT NULL CHECK (discount_pct BETWEEN 0 AND 1),
  net_revenue   DECIMAL(14,2) NOT NULL,
  status        VARCHAR NOT NULL
);
  ```


## **Import data from csv**

  ```sql
-- copy raw data from csv into tables created
COPY regions FROM 'C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\regions.csv' (HEADER);
COPY sales_reps FROM 'C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\sales_reps.csv' (HEADER);
COPY customers  FROM 'C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\customers.csv'  (HEADER);
COPY products   FROM 'C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\products.csv'   (HEADER);
COPY budget     FROM 'C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\budget.csv'     (HEADER);
COPY deals      FROM 'C:\Users\Yvonne\OneDrive\Desktop\NTU DSAI\Mock Real Life Example\Deepseek\deals.csv'      (HEADER);
  ```


## **Create View**

  ```sql
-- Deal-level analytical view. Deliberately does NOT include budget columns:
-- joining budget onto deals fans out budget_revenue across every deal row
-- and silently multiplies the plan figure. Budget is joined separately.
CREATE VIEW v_deal_enriched AS
SELECT
  d.deal_id,
  r.region_id,
  r.region_name,
  sr.rep_id,
  sr.rep_name,
  d.customer_id,
  c.customer_name,
  d.product_id,
  p.product_name,
  d.close_date,
  year(d.close_date) || '-Q' || quarter(d.close_date) AS quarter,
  d.units,
  d.gross_revenue,
  d.discount_pct,
  d.net_revenue,
  d.gross_revenue - d.net_revenue AS discount_value,
  d.status
FROM deals d
JOIN sales_reps sr ON d.rep_id     = sr.rep_id
JOIN regions    r  ON sr.region_id = r.region_id
JOIN customers  c  ON d.customer_id = c.customer_id
JOIN products   p  ON d.product_id  = p.product_id;
  ```
