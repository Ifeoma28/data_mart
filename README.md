# data_mart
An online supermarket that specialises in fresh produce.
![datamart](https://github.com/Ifeoma28/data_mart/blob/a315e7e75f17a97c710a87f7a08c458e1f4f3ea9/datamart%20visual%20dashboard.png)

## PROJECT OVERVIEW
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

## PROBLEM STATEMENT
- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment and customer types were the most impacted by this change?
- What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

### ABOUT DATASET
For this case study there is only a single table: data_mart.weekly_sales. 
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
- We developed a metric called New customer impact which tells us the percentage of new customers within the 12 weeks baseline window.


### Sales percentage change 4 weeks within baseline date

Even with a net -1% dip in overall sales, strong upward trends in key segments suggest:

New acquisition efforts are working, especially among retiree couples in Europe.

Retail remains a high-impact platform for certain customer groups.

Some platforms or regions may be underperforming and pulling the overall percentage down.

## Sales percentage change 12 weeks within baseline date


The primary driver of the overall -1% sales drop is Shopify > South America > Guest > Unknown Age Band & Demographic, which shows a +11% contribution to change.

This means guests in South America with unknown profiles increased their activity, but this likely diluted the average sales quality/value, contributing negatively overall.

The increase in guest sales (especially those with unknown customer details) may seem positive in volume but doesn’t necessarily convert to high-value sales, leading to a negative net effect on performance.




## Transaction percentage change 12 weeks within baseline date
The transaction decline of -1% overall is modest, considering the strong positive activity from couples and retirees.

The drop in Shopify transactions among couples slightly offset these gains.

Retail is the stronger-performing platform for the key demographic of retiree couples.

## KEY INSIGHTS
- Overall Platform Impact (12 Weeks)
Sales decreased by 1%, with Shopify up +0.05% and Retail down -0.01%.

Transactions declined by 1%, though Shopify rose +0.06%, and Retail fell -0.03%.

Average Transaction Value (ATV) improved by +5%.

- New customers contributed:

+3% to overall sales.

+9% to transaction growth – suggesting effective acquisition/onboarding.

Retirees showed a +14% boost in transactions – a growing, highly engaged segment.

Couples made up 47.04% of total transactions and drove an 83% transaction growth in Retail.

Guest customers logged the highest transaction count.

Shopify is losing Couples, contributing -2% to platform sales – potential disengagement or dissatisfaction.

- Regional & Platform Trends
Europe drove growth with a +4% contribution.

Shopify sales saw regional strength in Asia and South America.

South America is a standout driver for Shopify sales (noted in flow diagram).


- Demographics
A high number of unknown age bands and demographics exist, especially in Shopify.

"Unknown" group showed a +5.5% transaction increase.

Highlights the need for better segmentation and data completeness.

- Shopify has the highest ATV at 81.27%, pointing to higher-spending customers.

Sustainability update appears to positively impact Shopify more than Retail.


## RECOMMENDATION
- Better Segmentation
- Double down on new customer acquisition 
- Continue to invest in campaigns,referral programs or first-time buyer incentives to sustain this positive momentum
- We can see from the sales by customer type that existing customers show a slight negative impact on sales
  To help this, analyzing churn behaviour or drop-off points in each region and collecting feedbacks from existing customers to
  identify pain-points(eg surveys)
- Prioritize Shopify Engagement
Capitalize on Shopify’s high ATV (81.27%) and positive sales growth by tailoring premium offers or loyalty programs to this high-value segment.

- Monitor Retail Trends & Couples’ Behavior
Retail platform shows overall decline. The Retail-Couples group had an 83% increase in transactions, highlighting a potential niche—invest in storytelling, promotions, or bundles that resonate with this group.

Examine 15% retail sales decline contributors and test targeted recovery strategies.


- Make region-specific plans rather than one-size-fits-all approaches. for instance Shopify in Europe stays consistent but doesn’t grow much—indicating a mature market.
- Re-evaluate the retail model for Canada, Europe and South America to understand the sales stagnancy.
By Assessing store format, pricing, and customer experience.

## How to Minimize Future Impact of Sustainability Updates
-Platform Optimization:
Focus Shopify as the lead platform for product categories or campaigns tied to sustainability.

Leverage its higher average transaction value and steady customer base to absorb potential impact.

- Understanding which demographic and age band affects sales more and getting their preferences through Surveys before a sustainability change is made.
