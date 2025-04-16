# data_mart
An online supermarket that specialises in fresh produce.

## PROJECT OVERVIEW
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

## PROBLEM STATEMENT
- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

### ABOUT DATASET
For this case study there is only a single table: data_mart.weekly_sales
Data Mart has international operations using a multi-region strategy
Data Mart has both, a retail and online platform in the form of a Shopify store front to serve their customers
Customer segment and customer_type data relates to personal age and demographics information that is shared with Data Mart
transactions is the count of unique purchases made through Data Mart and sales is the actual dollar amount of purchases
Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a week_date value which represents the start of the sales week.

### DATA CLEANING STEPS
## SQL for cleaning
- Converted the week_date to a DATE format.
- A week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc.
- A month_number with the calendar month for each week_date value as the 3rd column.
- A calendar_year column as the 4th column containing either 2018, 2019 or 2020 values.
- A new column called age_band after the original segment column using the following mapping on the number inside the segment value.
- A new demographic column using the following mapping for the first letter in the segment values:

- Ensured all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns.
- Generated a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record.
## Power Bi for visualization

## BUSINESS QUESTIONS
- What day of the week is used for each week_date value?
```
SELECT week_date,DATEPART(dw,week_date)
FROM clean_weekly_sales;
-- show that this sales was computed on Monday weekly
```
- How many total transactions were there for each year in the dataset?
```
SELECT DISTINCT calendar_year,SUM(transactions) AS transaction_count
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY 2 DESC;
```
- What is the total sales for each region for each month?
```
SELECT  region,Month_no,calendar_year,SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY Month_no,region,calendar_year
ORDER BY 4 DESC;
-- Oceania has the highest total sales in the month of August 2020
-- Europe has the least total sales in the month of march 2018
```
- What is the total count of transactions for each platform ?
```
SELECT  platform,SUM(transactions) AS transaction_count
FROM clean_weekly_sales
GROUP BY platform
ORDER BY 2 DESC;
```
- What is the percentage of sales for Retail vs Shopify for each month?
```
WITH sales AS (SELECT  platform,calendar_year,Month_no,SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY platform,Month_no,calendar_year
)
SELECT platform,calendar_year,Month_no,total_sales,(total_sales * 100/SUM(total_sales) OVER (PARTITION 
BY calendar_year,Month_no)) AS sales_percent
FROM sales;
-- retail is leading 
```
- What is the percentage of sales by demographic for each year in the dataset?
```
WITH sales AS (SELECT demographic,calendar_year,SUM(sales) AS total_sales
FROM clean_weekly_sales
GROUP BY demographic,calendar_year
)
SELECT demographic,calendar_year,total_sales,(total_sales * 100/SUM(total_sales) OVER (PARTITION 
BY calendar_year)) AS sales_percent
FROM sales;
```
- Which age_band and demographic values contribute the most to Retail sales?
```
SELECT age_band,calendar_year,SUM(sales) AS total_sales,(SUM(sales)  * 100/SUM(SUM(sales) ) OVER (PARTITION 
BY calendar_year)) AS sales_percent
FROM clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY age_band,calendar_year;

-- for demographic
SELECT demographic,calendar_year,SUM(sales) AS total_sales,(SUM(sales)  * 100/SUM(SUM(sales) ) OVER (PARTITION 
BY calendar_year)) AS sales_percent
FROM clean_weekly_sales
WHERE platform = 'Retail'
GROUP BY demographic,calendar_year;
-- Retirees and a lot of people contributed followed by families
```
- The  average transaction size for each year for Retail vs Shopify?
```
SELECT calendar_year,platform,(total_sales/transaction_count) AS avg_transaction_value_year
FROM
	(SELECT calendar_year,platform,sales,transactions,SUM(sales) OVER(PARTITION BY calendar_year,platform) AS total_sales,
	 SUM(transactions) 
	OVER (PARTITION BY calendar_year,platform)AS transaction_count
	FROM clean_weekly_sales
	GROUP BY calendar_year,platform,sales,transactions) AS transaction_size
GROUP BY calendar_year,platform,total_sales,transaction_count;
-- this is the average amount of dollars ppl spend yearly for each platform
```
Average transaction value breakdown 
![AVT](https://github.com/Ifeoma28/data_mart/blob/a4b6f6319f2cb5d9f61e53790d890dac52d16d90/average%20transaction%20value%20breakdown.png)

## Analysis
- we want to inspect the impact of the package that was introduced in June 2020
- let us use 15th june as the baseline week where the data mart sustainable packaging changes came into effect
   that is 2 weeks into the month of june.
- let us see the total sales for the 4 weeks before the baseline date and after the date.
- also the total sales,total transactions for the 12 weeks before the baseline date and after the date.


### Sales percentage change 4 weeks within baseline date

![sales 4-week change](https://github.com/Ifeoma28/data_mart/blob/c60811719c337acc98f1b1815e9ba159bf179036/Sales%20%25%20change%20(4%20weeks).png)
Overall sales remained unchanged over the 4-week period, but this was largely due to new middle-aged families in Europe on the Shopify platform, who contributed a +5% lift.
Their growth helped absorb slowdowns in other segments, making them a critical buffer for the business during this period. 

## Sales percentage change 12 weeks within baseline date

![sales 12-weeks change](https://github.com/Ifeoma28/data_mart/blob/faee1e59a652235ec2ecc0fb28a49d5e1b450516/sales%20%25%20change%20(12%20weeks).png)
The -9% drop in sales is partially driven by couples in Oceania (within the young adults segment) on the Shopify platform. 
However, it's important to note that couples in Oceania have shown stagnant sales on Shopify, indicating a lack of growth in this segment.
The stagnation suggests that Shopify might be reaching its peak potential for this demographic in Oceania.

## Transaction percentage change 12 weeks within baseline date
The 9% decrease in transactions over the 12-week period is notably influenced by Europe, which has the highest transaction count. This market alone contributes 4% of the overall transaction percentage change reduction.
The drop in transactions is mainly driven by newly retired couples in Europe on the Retail platform, although this group has shown consistent growth over the years. The temporary reduction could be linked to seasonal factors or market adjustments.

## KEY INSIGHTS
- Europe, South America and USA remains a key market for long-term transaction growth with Europe having the highest transaction count.
- The decline in transactions during the 12 weeks change may indicate that they are either reducing spending or perhaps experiencing a shift in their shopping behavior as they adjust to a new lifestyle post-retirement.(new retired couples)
- Existing customers have the highest transaction count.
- A lot of our customers demographic were unkwown.
- Shopify platform has a higher average transaction value indicating higher spending customers

## RECOMMENDATION
- Re-engagement strategies or new initiatives might be required to spark growth again for young adults especially in Oceania.
- 
