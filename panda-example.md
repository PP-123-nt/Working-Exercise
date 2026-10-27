## Common Panda code used for FBP


## 🚀🚀🚀 **GOOGLE COLAB** 🚀🚀🚀

```sql
import pandas as pd
import numpy as np

# 1. Load the data (Pandas)
sales = pd.read_csv('sales_data.csv')
marketing = pd.read_excel('marketing_spend.xlsx') 

# 2. Clean the data (Pandas)
sales['revenue'] = sales['revenue'].fillna(0)

# 3. Merge the data - VLOOKUP (Pandas)
combined = pd.merge(sales, marketing, on='campaign_id', how='left')

# 4. Calculate Marketing ROI (Pandas)
combined['marketing_roi'] = (combined['revenue'] - combined['marketing_spend']) / combined['marketing_spend']

# 5. Create a Pivot Table for the VP (Pandas)
report = combined.groupby(['region', 'product_category']).agg(
    total_revenue=('revenue', 'sum'),
    total_spend=('marketing_spend', 'sum')
).reset_index()

# Calculate overall ROI and round to 1 decimal place
report['roi'] = ((report['total_revenue'] - report['total_spend']) / report['total_spend']).round(1)

# 6. Add ROI Ranking (Rank 1 = Highest ROI)
# method='min' means if there is a tie, they share the highest rank, and the next rank is skipped
report['roi_ranking'] = report['roi'].rank(ascending=False, method='min').astype(int)

# 7. Add ROI Tier
# np.where(condition, value_if_true, value_if_false)
report['roi_tier'] = np.where(report['roi'] > 5, 'High', 
                     np.where(report['roi'] < 2, 'Low', 'Standard'))

# 8. Sort output by ROI ranking (ascending, so rank 1 is at the top)
report = report.sort_values(by='roi_ranking', ascending=True).reset_index(drop=True)

# 9. Export to Excel to send to stakeholders (Pandas)
report.to_excel('Q3_Sales_and_Marketing_Performance.xlsx', index=False)

print("Report generated successfully! Here is a preview:")
print(report.head())
```


### 1. Raw data: Sales Data (`sales_data.csv`)
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

### 2. Raw Data: Marketing Spend Data (`marketing_spend.xlsx`)
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

### 3. Output: Sales & Marketing Performance (`Q3_Sales_and_Marketing_Performance.xlsx`)

| region | product_category | total_revenue | total_spend | roi | roi_ranking | roi_tier |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Central | Apparel | 23000 | 2500 | 8.2 | 1 | High |
| North | Electronics | 75000 | 16000 | 3.7 | 2 | Standard |
| East | Electronics | 45000 | 12000 | 2.8 | 3 | Standard |
| South | Apparel | 27000 | 8000 | 2.4 | 4 | Standard |
| West | Home Goods | 8000 | 3500 | 1.3 | 5 | Low |


<img width="554" height="392" alt="image" src="https://github.com/user-attachments/assets/da9996e3-d8af-4a6f-bb9a-9b797b58a2fb" />


<br>


## 📊📊📊 **PYTHON CODE IN EXCEL**  📊📊📊

If you want to use the new Python in Excel feature (available in Microsoft 365), there is a very important limitation you need to know: Excel's Python runs in a secure Microsoft Cloud environment, so it cannot directly read or write files on your local C: drive using paths like pd.read_csv('sales_data.csv') or .to_excel().

To make it work, you have to change how the data gets in and out of Python:

1. Input: Load the data into Excel sheets first, then use Excel's xl() function to bring it into Python.

2. Output: Let the Python code output the final table directly into the Excel grid, and then you manually save the file.

Here is the exact step-by-step guide to doing this in Excel:

### 📥 Step 1: Load Raw Data into Excel Sheets
Since Python in Excel can't read the local folder, you need to put the data inside the workbook.

1. Open a blank Excel workbook.

2. Go to Data > Get Data > From File > From Text/CSV. Select your sales_data.csv. When prompted, click Load.

3. In the new sheet, click anywhere inside the data and press Ctrl + T to make it a Table. Rename the Table (in the Table Design tab on the top ribbon) to SalesData.

4. Go to Data > Get Data > From File > From Excel Workbook. Select marketing_spend.xlsx and click Load.

5. Make that a Table too (Ctrl + T) and rename it to MarketingData.

### 🐍 Step 2: Write the Python Code in Excel
1. Click on an empty cell in a new sheet (e.g., cell A1).

2. On the Formulas tab, click Python (or type =PY().

3. In the formula bar, paste the following modified code. Notice how we use xl() instead of pd.read_csv() and remove the .to_excel() step at the end.

```sql
import pandas as pd
import numpy as np

# 1. Load the data from the Excel Tables we created
sales = xl("SalesData[#All]", headers=True)
marketing = xl("MarketingData[#All]", headers=True)

# 2. Clean the data
sales['revenue'] = sales['revenue'].fillna(0)

# 3. Merge the data
combined = pd.merge(sales, marketing, on='campaign_id', how='left')

# 4. Calculate Marketing ROI
combined['marketing_roi'] = (combined['revenue'] - combined['marketing_spend']) / combined['marketing_spend']

# 5. Create the Pivot Table
report = combined.groupby(['region', 'product_category']).agg(
    total_revenue=('revenue', 'sum'),
    total_spend=('marketing_spend', 'sum')
).reset_index()

# Calculate overall ROI and round to 1 decimal place
report['roi'] = ((report['total_revenue'] - report['total_spend']) / report['total_spend']).round(1)

# 6. Add ROI Ranking
report['roi_ranking'] = report['roi'].rank(ascending=False, method='min').astype(int)

# 7. Add ROI Tier
report['roi_tier'] = np.where(report['roi'] > 5, 'High', 
                     np.where(report['roi'] < 2, 'Low', 'Standard'))

# 8. Sort output by ROI ranking
report = report.sort_values(by='roi_ranking', ascending=True).reset_index(drop=True)

# 9. Output to the grid
report
```
4. Press Ctrl + Enter to run the code.

### 💾 Step 3: Export to the Same Folder
Once you run the code, Python will output the final DataFrame and "spill" it into the Excel grid as a table.

1. Select the spilled table.

2. Right-click and choose Convert to Data Range (this makes it easier to format).

3. Go to File > Save As "Q3_Sales_and_Marketing_Performance".


### 🔄 Step 4: Refresh
1. Once raw data files are updated with latest number, open "Q3_Sales_and_Marketing_Performance". 

2. Go to Data > Refresh All > Refresh All
