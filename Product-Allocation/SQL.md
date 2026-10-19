## Create Table

```sql
CREATE TABLE claims_data (
    client_id INT,
    Claim_id INT,
    claim_date VARCHAR(50),   -- if use DATE type need to change date data format to YYYY-MM-DD
    claim_amt DECIMAL(18, 2),
    car_type VARCHAR(50)
);
```

## Insert Data
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

## Compute Allocation Amount
**1. allocate car_type = NA claim amount to claim with valid car_type within the same client**

**2. allocated amount = claim amount if a client only has car_type = NA claim(s)**
```sql
WITH ClientStats AS (
    SELECT 
        client_id, Claim_id, car_type, claim_amt,
        SUM(CASE WHEN car_type = 'NA' THEN claim_amt ELSE 0 END) OVER (PARTITION BY client_id) AS total_na_amt,
        SUM(CASE WHEN car_type != 'NA' THEN claim_amt ELSE 0 END) OVER (PARTITION BY client_id) AS total_valid_amt,
        COUNT(CASE WHEN car_type != 'NA' THEN 1 END) OVER (PARTITION BY client_id) AS valid_claim_count
    FROM claims_data
)
SELECT 
    client_id, Claim_id, car_type, claim_amt,
    CAST(                                                 -- Wrap the CASE statement in a CAST to force 2 decimal places
        CASE 
            WHEN car_type != 'NA' AND valid_claim_count > 0 AND total_na_amt > 0     -- Rule: Distribute to valid claims
                THEN claim_amt + (claim_amt / total_valid_amt) * total_na_amt
            WHEN car_type = 'NA' AND valid_claim_count > 0 AND total_na_amt > 0      -- Rule: NA claim = 0 if distributed
                THEN 0 
            ELSE claim_amt    -- Rule: Keep original amount
        END AS DECIMAL(18, 2)
    ) AS Amt_Allocation
FROM ClientStats
ORDER BY client_id, Claim_id;
```


| client_id | Claim_id | car_type | claim_amt | Amt Allocation | &nbsp; | total_na_amt | total_valid_amt | valid_claim_count |
| :--- | :--- | :--- | ---: | ---: | :---: | ---: | ---: | ---: |
| 1 | 169535646 | Truck | $532.00 | $549.93 | &nbsp; | $234.00 | $6,942.00 | 3 |
| 1 | 734265621 | Van | $5,733.00 | $5,926.25 | &nbsp; | $234.00 | $6,942.00 | 3 |
| 1 | 747557690 | Car | $677.00 | $699.82 | &nbsp; | $234.00 | $6,942.00 | 3 |
| 1 | 847557635 | NA | $234.00 | $0 | &nbsp; | $234.00 | $6,942.00 | 3 |
| 3 | 697949195 | Van | $623.00 | $1,310.80 | &nbsp; | $2,314.00 | $2,096.00 | 3 |
| 3 | 797969175 | Van | $907.00 | $1,908.33 | &nbsp; | $2,314.00 | $2,096.00 | 3 |
| 3 | 797670195 | NA | $2,314.00 | $0 | &nbsp; | $2,314.00 | $2,096.00 | 3 |
| 3 | 872231801 | Car | $566.00 | $1,190.87 | &nbsp; | $2,314.00 | $2,096.00 | 3 |
| 4 | 971694618 | Truck | $788.00 | $788.00 | &nbsp; | $0 | $5,331.00 | 2 |
| 4 | 991694620 | Van | $4,543.00 | $4,543.00 | &nbsp; | $0 | $5,331.00 | 2 |
| 10 | 971696623 | NA | $432.00 | $432.00 | &nbsp; | $432.00 | $0 | 0 |
| 11 | 268535746 | Truck | $5,232.00 | $5,565.00 | &nbsp; | $333.00 | $5,232.00 | 1 |
| 11 | 334265625 | NA | $333.00 | $0 | &nbsp; | $333.00 | $5,232.00 | 1 |
| 2 | 166535746 | NA | $332.00 | $332.00 | &nbsp; | $1,065.00 | $0 | 0 |
| 2 | 235265625 | NA | $733.00 | $733.00 | &nbsp; | $1,065.00 | $0 | 0 |
| 6 | 275330646 | NA | $1,332.00 | $0 | &nbsp; | $3,807.00 | $888.00 | 1 |
| 6 | 635265643 | NA | $2,475.00 | $0 | &nbsp; | $3,807.00 | $888.00 | 1 |
| 6 | 215265656 | Van | $888.00 | $4,695.00 | &nbsp; | $3,807.00 | $888.00 | 1 |





