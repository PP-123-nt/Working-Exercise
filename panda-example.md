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

# Second: Calculate the overall ROI using the totals
report['roi'] = (report['total_revenue'] - report['total_spend']) / report['total_spend']

# 6. Export to Excel to send to stakeholders (Pandas)
report.to_excel('Q3_Sales_and_Marketing_Performance.xlsx', index=False)

print("Report generated successfully!")
print(report.head()) # This will print the first 5 rows so you can check the new 'roi' column
```

<img width="746" height="410" alt="image" src="https://github.com/user-attachments/assets/dce42bca-e122-4ecb-b073-c789a25e8f6c" />
