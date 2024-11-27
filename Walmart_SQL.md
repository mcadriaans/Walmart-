# Walmart SQL Queries
## Data imported to PSQL
```sql
SELECT * FROM walmart;
```
![image](https://github.com/user-attachments/assets/c3c7488e-b4d1-4ff7-a0e8-f5e37ba27e6d)

## 🔍 Exploratory Data Analysis

### What is the start date, end date and time period for the transactions captured in the data?
```sql
SELECT 
 MIN(date) AS start_date,
 MAX(date) AS end_date,
 AGE(MAX(date), MIN(date)) AS time_period
FROM walmart;
```
![image](https://github.com/user-attachments/assets/f4ad0576-47e4-480f-8cdd-a3d9771e9a62)


### How many transactions are there?
```sql
SELECT 
 COUNT(*) AS nr_of_transactions
FROM walmart;
```
![image](https://github.com/user-attachments/assets/55571d53-5397-429b-8a9f-1e0a812a7d17)


### How many store branches do we have?
```sql
SELECT 
 COUNT(DISTINCT branch) AS total_branches
FROM walmart;
```
![image](https://github.com/user-attachments/assets/8d6a5401-f3ce-474a-9b00-1305cfcbadfd)

### How many transaction were made for each payment method?</b>
```sql
SELECT 
 payment_method,
 COUNT(*) AS nr_of_transactions
FROM walmart
GROUP BY 1;
```
![image](https://github.com/user-attachments/assets/199c8f82-e98d-4433-a2cb-a4b5dcd774b7)
```python
## Total transactions by payment method
ax =sns.countplot(x='payment_method', data=df, hue='payment_method')

# Iterate over each bar and display the count value above it
for p in ax.patches:
    ax.annotate(f'{p.get_height()}', (p.get_x() + p.get_width() / 2., p.get_height()),
                ha='center', va='center', xytext=(0, 5), textcoords='offset points')
    
plt.show()
```
![image](https://github.com/user-attachments/assets/64ad7463-2154-407a-be27-d6573fec0a05)


## Solving Business Problems
### 💰 Revenue 
#### Calculate the total revenue generated by each branch.
```sql
SELECT 
	branch,
	ROUND(SUM(total_revenue)::NUMERIC,2) AS total_branch_revenue
FROM walmart
GROUP BY 1
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/f2afd552-8498-4a47-91de-5b06e8c16f33)


#### Identify the top 3 cities with the highest total revenue.
```sql
SELECT 
 city,
 ROUND(SUM(total_revenue)::NUMERIC, 2) AS total_cty_revenue
FROM walmart
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
```
![image](https://github.com/user-attachments/assets/84bc8f5b-5981-4b26-b6fc-8f8d82fe4d67)


#### Calculate the total revenue generated for each month.
```sql
SELECT 
 EXTRACT(MONTH from date) AS mnth_nr,
 TO_CHAR(date, 'Month') AS month,
 ROUND(SUM(total_revenue)::NUMERIC, 2) AS daily_total_revenue
FROM walmart
GROUP BY 1, 2
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/221d3629-35be-433f-9288-08968965221e)


#### Find the branch with the highest total revenue for the "Health and beauty" category.
```sql
SELECT 
 branch,
 city,
 ROUND(SUM(total_revenue)::NUMERIC, 2) AS branch_total_revenue
FROM walmart
WHERE category = 'Health and beauty'
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 1;
```
![image](https://github.com/user-attachments/assets/3c6c9f9c-ad15-4203-a7e5-b2433e2659f7)


### 💲 Profits
#### Calculate the average profit margin for each payment method.</b>
```sql
SELECT 
 payment_method,
 ROUND(AVG(profit_margin)::NUMERIC,2) AS avg_profit_payment_method
FROM walmart
GROUP BY 1
ORDER BY 2 DESC;
```
![image](https://github.com/user-attachments/assets/26186197-2beb-47d7-9b81-65a1c2f737af)


#### Calculate the total net profit for each category.
```sql
SELECT 
	category,
	ROUND(SUM(total_revenue * profit_margin)::NUMERIC, 2) AS category_net_profit
FROM walmart
GROUP BY 1
ORDER BY 2 DESC;
```
![image](https://github.com/user-attachments/assets/583484a1-6e6d-400b-b70e-11b51dbb95ce)


### ⭐ Ratings
#### Determine the average , minimum and maximum rating of products for each city.</b>
```sql
SELECT 
 city,
 category,
 MIN(rating) AS min_rating,
 ROUND(AVG(rating)::NUMERIC, 2) AS avg_rating,
 MAX(rating) AS max_rating
FROM walmart
GROUP BY 1, 2
ORDER BY 1, 2;
```
![image](https://github.com/user-attachments/assets/00f38190-073f-45c2-825b-81ad439d3149)


#### Identify the highest-rated category in each branch. Display the branch, category and average rating.</b>
```sql
SELECT
 branch,
 category,
 ROUND(avg_rating::NUMERIC, 2)
FROM (
    SELECT
     branch,
     category,
     AVG(rating) AS avg_rating,
     RANK() OVER (
       PARTITION BY branch
       ORDER BY AVG(rating) DESC
     ) AS rnk
    FROM walmart
    GROUP BY branch, category
) AS subquery
WHERE rnk = 1;
```
![image](https://github.com/user-attachments/assets/a35dabbe-6439-4405-bcc6-40289acdb08a)


### 💳 Payments
#### Find the different payment methods their total number of transactions and total quantities sold.</b>
```sql
SELECT 
 payment_method,
 COUNT(*) AS nr_of_transactions,
 SUM(quantity) AS quantity_sold
FROM walmart
GROUP BY 1;
```
![image](https://github.com/user-attachments/assets/788bd8ba-2c3e-4327-b27c-1974dda4562c)


#### Determine the most common payment method for each Branch.</b>
```sql
SELECT
 branch,
 payment_method
FROM (
  SELECT 
   branch,
   payment_method,
   COUNT(payment_method),
   RANK()OVER(
     PARTITION BY branch
     ORDER BY COUNT(payment_method) DESC
   )AS rnk
   FROM walmart
   GROUP BY 1, 2
)
WHERE rnk = 1
ORDER BY 1;
```
![image](https://github.com/user-attachments/assets/a05dc569-9784-4e5c-bb03-562324a5f5d6)


### 📈 Sales Trends
#### Identify the busiest day of the week for each branch based on the number of transactions.</b>
``` sql
SELECT
 branch,
 day_name,
 nr_of_transactions
FROM (
 SELECT
  branch,
  TO_CHAR(date, 'Day') AS day_name,
  COUNT(*) AS nr_of_transactions,
  RANK()OVER(
    PARTITION BY branch
    ORDER BY COUNT(*) DESC 
 )AS rnk
 FROM walmart
 GROUP BY 1, 2
)
WHERE rnk = 1;
```
![image](https://github.com/user-attachments/assets/2a36828d-6c3f-421a-bacd-8631740eab61)


#### How many transactions were completed in each shift (Morning, Afternoon, Evening) for each branch at Walmart?</b>
```sql
SELECT
  branch,
  CASE
   WHEN time BETWEEN '06:00:00' AND '11:59:59' THEN 'Morning'
   WHEN time BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
   WHEN time BETWEEN '18:00:00' AND '23:59:59' THEN 'Evening'
  END AS shift,
  COUNT(*) AS nr_of_transactions
 FROM walmart
 GROUP BY 1, 2
 ORDER BY 1, 3 DESC;
```
![image](https://github.com/user-attachments/assets/801e8046-dfe9-4015-a7d4-19a581760e42)


#### Determine the top 5 branches that experienced the greatest percentage decrease in revenue from 2022 to 2023.</b>
```sql
WITH yearly_revenue AS (
 SELECT
  branch,
  EXTRACT(YEAR from date) AS year,
  SUM(total_revenue) AS total_revenue
 FROM walmart
 GROUP BY 1, 2
 ORDER BY 1
), compare_revenues AS(
 SELECT 
   r1.branch,
   r1.total_revenue AS total_revenue_2022,
   r2.total_revenue AS total_revenue_2023,
  ROUND(((r1.total_revenue - r2.total_revenue) / r1.total_revenue)::NUMERIC * 100 , 2) AS percent_decrease
 FROM(SELECT * FROM yearly_revenue WHERE year = 2022) AS r1
 JOIN(SELECT * FROM yearly_revenue WHERE year = 2023) AS r2
   ON r1.branch = r2.branch
)

SELECT *
FROM compare_revenues
WHERE percent_decrease > 0
ORDER BY percent_decrease DESC
LIMIT 5;
```
![image](https://github.com/user-attachments/assets/719551f6-713b-459d-aca1-0f940c206e6b)