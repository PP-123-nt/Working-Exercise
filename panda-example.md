## Common Panda code used for FBP

**run in google colab**

```sql
import pandas as pd
import numpy as np

# 1. Load the data (Pandas)
sales = pd.read_csv('sales_data.csv')
marketing = pd.read_excel('marketing_spend.xlsx') # Ensure this is read_excel

# 2. Clean the data (Pandas)
sales['revenue'] = sales['revenue'].fillna(0)

# 3. Merge the data - VLOOKUP (Pandas)
combined = pd.merge(sales, marketing, on='campaign_id', how='left')

# 4. Calculate Marketing ROI (Pandas)
# (This row-level ROI is still useful for individual campaign analysis)
combined['marketing_roi'] = (combined['revenue'] - combined['marketing_spend']) / combined['marketing_spend']

# 5. Create a Pivot Table for the VP (Pandas)
# First: Aggregate the Total Revenue and Total Spend
report = combined.groupby(['region', 'product_category']).agg(
    total_revenue=('revenue', 'sum'),
    total_spend=('marketing_spend', 'sum')
).reset_index()

# Second: Calculate the overall ROI using the totals and round to 1 decimal place
report['roi'] = ((report['total_revenue'] - report['total_spend']) / report['total_spend']).round(1)

# 6. Export to Excel to send to stakeholders (Pandas)
report.to_excel('Q3_Sales_and_Marketing_Performance.xlsx', index=False)

print("Report generated successfully!")
print(report.head()) # This will print the first 5 rows so you can check the new 'roi' column
```

## 📊 Raw Data

### 1. Sales Data (`sales_data.csv`)
Contains revenue, region, and product category for each campaign.

| campaign_id | revenue | region | product_category |
| :--- | :--- | :--- | :--- |
| C001 | 50000 | North | Electronics |
| C002 | 15000 | South | Apparel |
| C003 | 25000 | North | Electronics |
| C004 | 8000 | West | Home Goods |
| C005 | 45000 | East | Electronics |
| C006 | 12000 | South | Apparel |
| C007 | *(blank)* | West | Home Goods |
| C008 | 23000 | Central | Apparel |

### 2. Marketing Spend Data (`marketing_spend.xlsx`)
Contains the marketing spend associated with each campaign.

| campaign_id | marketing_spend |
| :--- | :--- |
| C001 | 10000 |
| C002 | 5000 |
| C003 | 6000 |
| C004 | 2000 |
| C005 | 12000 |
| C006 | 3000 |
| C007 | 1500 |
| C008 | 2500 |

### 3. Sales & Marketing Performance Output (`Q3_Sales_and_Marketing_Performance.xlsx`)

| region | product_category | total_revenue | total_spend | roi |
| :--- | :--- | :--- | :--- | :--- |
| Central | Apparel | 23000 | 2500 | 8.2 |
| East | Electronics | 45000 | 12000 | 2.8 |
| North | Electronics | 75000 | 16000 | 3.7 |
| South | Apparel | 27000 | 8000 | 2.4 |
| West | Home Goods | 8000 | 3500 | 1.3 |


<img width="596" height="413" alt="image" src="https://github.com/user-attachments/assets/78034296-e066-40fe-ba7c-9e521e374b1e" />


