
**2. Creating a Schema:**
## Create Table

```sql
CREATE TABLE claims_data (
    client_id BIGINT,
    Claim_id BIGINT,
    claim_date VARCHAR(50),
    claim_amt BIGINT,
    car_type VARCHAR(50)
);
```

** Insert Data **
```sql
INSERT INTO claims_data (client_id, Claim_id, claim_date, claim_amt, car_type) VALUES
(1, 169535646, '21/6/2019', 532, 'Truck'),
(1, 734265621, '18/9/2030', 5733, 'Van'),
(1, 747557690, '19/8/2027', 677, 'Car'),
(1, 847557635, '18/6/2024', 234, 'NA'),
(3, 697949195, '19/4/2021', 623, 'Van'),
(3, 797969175, '19/4/2022', 907, 'Van'),
(3, 797670195, '17/7/2022', 2314, 'NA'),
(3, 872231801, '19/7/2026', 566, 'Car'),
(4, 971694618, '19/1/2016', 788, 'Truck'),
(4, 991694620, '1/7/2020', 4543, 'Van'),
(10, 971696623, '28/5/2020', 432, 'NA'),
(2, 166535746, '12/3/2027', 332, 'NA'),
(2, 235265625, '21/1/2018', 733, 'NA'),
(6, 275330646, '12/3/2027', 1332, 'NA'),
(6, 635265643, '19/12/2022', 2475, 'NA'),
(6, 215265656, '22/10/2012', 888, 'Van'),
(11, 268535746, '21/7/2017', 5232, 'Truck'),
(11, 334265625, '11/10/2028', 333, 'NA');
```


** Computr Allocation Amount for car_type = NA **
```sql
WITH ClientStats AS (
    SELECT 
        client_id, Claim_id, car_type, claim_amt,
        SUM(CASE WHEN car_type = 'NA' THEN claim_amt ELSE 0 END)   -- Calculate total NA amount for the client
            OVER (PARTITION BY client_id) AS total_na_amt,
        SUM(CASE WHEN car_type != 'NA' THEN claim_amt ELSE 0 END)  -- Calculate total valid amount for the client
            OVER (PARTITION BY client_id) AS total_valid_amt,
        COUNT(CASE WHEN car_type != 'NA' THEN 1 END)               -- Count how many non NA claims a client has
            OVER (PARTITION BY client_id) AS valid_claim_count
    FROM claims_data
)
SELECT 
    client_id, Claim_id, car_type, claim_amt,
    CASE 
        WHEN car_type != 'NA' AND valid_claim_count > 0 AND total_na_amt > 0      -- 1. Distribute to valid claims IF there are multiple valid claims and an NA claim exists
            THEN ROUND(claim_amt + (claim_amt::numeric / total_valid_amt) * total_na_amt, 0)  -- Cast to numeric/decimal to prevent integer division truncation
        WHEN car_type = 'NA' AND valid_claim_count > 0 AND total_na_amt > 0      -- 2. Zero out the NA claim if it was distributed to multiple valid claims
            THEN 0       -- Represents the '-' in your image
        ELSE claim_amt   -- 3. Leave everything else exactly as it is (e.g., Client 4 or 10)
    END AS Amt_Allocation
FROM ClientStats
ORDER BY client_id, Claim_id;
```
