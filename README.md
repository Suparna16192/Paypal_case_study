# Paypal_case_study

### This project explores and analyzes a PayPal transaction dataset to uncover patterns, identify potential anomalies, and generate actionable insights. The primary objectives were to understand transaction trends, analyze behavior of the merchants, and evaluate risk factors through descriptive and exploratory data analysis.
---
## 📁 File
Contains 20 SQL queries with outputs, each designed to solve a specific analytics problem.

---
## 📊 Case Study with Insights
Each question below includes the business problem, the SQL query, and the corresponding insights derived from the results.

---
## Basic Queries


### 1.List the top 10 highest-value transactions (amount + date + currency).
```sql
select transaction_amount, transaction_date, currency_code 
from transactions 
order by transaction_amount desc 
limit 10;

```
**Insight**: Most of the highest transactions are made with GBP, EUR and JPY currencies.

---

### 2.Show all transactions done in INR during 2023.
```sql
select transaction_amount, transaction_date, currency_code
from transactions
where currency_code = "INR" and year(transaction_date) = "2023"; 

```
**Insight**: List of transactions made in INR in 2023.

---
### 3.Find the total transaction amount per currency.
```sql
select sum(transaction_amount), currency_code
from transactions
group by currency_code;

```
**Insight**: Highest transactions are made with GBP, INR and USD.

---
### 4.Count how many transactions each sender made.
```sql
select sender_id, count(transaction_id) as no_of_transactions
from transactions
group by sender_id
order by no_of_transactions desc; 

```

**Insight**: Sender 558, 333, 759 ,639 made the highest number of transactions i.e 21.

---
### 5.Find the average transaction amount in USD.
```sql
select currency_code, avg(transaction_amount) as avg_transaction_amount
from transactions
where currency_code = "USD"; 

```
**Insight**: The Average transaction amount in USD is $5036.14.

---

## Joins & Lookups

### 6.Show all transactions with the sender and recipient merchant names (join transactions with merchants twice).
```sql
select T.sender_id, M1.business_name as sender_name, T.recipient_id, M2.business_name as recipient_name, T.transaction_amount
from transactions T
join merchants M1 on T.sender_id = M1.merchant_id
join merchants M2 on T.recipient_id = M2.merchant_id; 

```
**Insight**: List of all the transactions along with the name of the sender and recipient.

---

### 7.List transactions along with the country names of both sender and recipient.
```sql
select T.sender_id, C.country_name as sender_country, T.recipient_id, C2.country_name as recipient_country, T.transaction_amount
from transactions T
join merchants M1 on T.sender_id = M1.merchant_id
join merchants M2 on T.recipient_id = M2.merchant_id
join countries C on M1.country_id = C.country_id
join countries C2 on M2.country_id = C2.country_id;

```

**Insight**: List of sender's and recipient's along with the transaction amounts.

---
### 8.Find the top 5 countries by total transaction amount (consider both senders and recipients).
```sql
select C.Country_name, sum(T.transaction_amount) as total_transactions_senders
from transactions T 
join Merchants M on T.sender_id = M.merchant_id
join countries C on M.country_id = C.country_id
group by C.Country_name
order by total_transactions_senders desc
limit 5;


select C.Country_name, sum(T.transaction_amount) as total_transactions_recipients
from transactions T 
join Merchants M on T.recipient_id = M.merchant_id
join countries C on M.country_id = C.country_id
group by C.Country_name
order by total_transactions_recipients desc
limit 5; 

```
**Insight**: List of top 5 senders and top 5 recipients countries along with total transaction amount.

---

### 9.Show merchants from "India" along with their total transaction amount received.
```sql
select M.business_name, C.country_name, sum(T.transaction_amount) from transactions T 
join Merchants M on T.recipient_id = M.merchant_id
join countries C on M.country_id = C.country_id
where C.country_name ="India"
group by M.business_name, C.country_name;

select * from countries
where country_name = "India"; 

```
**Insight**: India is not included in the countries data sheet.

---

## Date & Time Analysis

### 10.Find the month with the highest total transaction value.
```sql

select * from transactions;

select date_format(transaction_date, "%Y-%m") as month, sum(transaction_amount) as total_monthly_transaction from transactions
group by month
order by total_monthly_transaction desc
limit 1; 

```
**Insight**: The highest total transaction value if for the month of Jan, 2024.

---

### 11.Show year-over-year growth in transaction amounts.
```sql
with cte as (select year(transaction_date) as year, sum(transaction_amount) as total_yearly_transaction from transactions
group by year
order by year)

select year, total_yearly_transaction, lag(total_yearly_transaction)over(order by year) as previous_year_transaction,
total_yearly_transaction-lag(total_yearly_transaction)over(order by year) as year_on_year_growth from cte; 

```
**Insight**: Year-over-year growth is significantly low, which may suggest market saturation, seasonal impact, etc.

---

### 12.Find all transactions that happened on weekends.
```sql
select transaction_id, transaction_date, dayname(transaction_date) as day, transaction_amount from transactions
where dayofweek(transaction_date) in (1,7); 

```
**Insight**: List of all the transactions done on a Saturday or a Sunday.

---

### 13.Get the first transaction date for each merchant using FIRST_VALUE().

```sql
select * from merchants; 
select * from transactions;

select distinct M.merchant_id, M.business_name, first_value(T.transaction_date)over(partition by T.sender_id order by T.transaction_date) as first_transaction
from transactions T
join merchants M on T.sender_id = M.merchant_id;

```
**Insight**: Transaction dates for all of the Merchant's first transaction.

---
### 14.Calculate the time difference in days between consecutive transactions for each sender using LAG().

```sql
with cte as (select transaction_id, sender_id, transaction_date, lag(transaction_date)over(partition by sender_id order by transaction_date) as previous_transaction_date
from transactions)
select *, datediff(transaction_date, previous_transaction_date) as days_between_transaction from cte;

```
**Insight**: Time taken by the sender for their next transaction.

---

## Window Functions & Analytics

### 15.Rank all transactions per sender by amount (RANK()).

```sql
select transaction_id, sender_id, transaction_amount, rank()over(partition by sender_id order by transaction_amount) as transaction_rank
from transactions;

```
**Insight**: Ranked transactions for all the senders as per the transaction amount.

---
### 16.For each sender, find the previous transaction amount (LAG()) and difference from the current.

```sql
with cte as (select transaction_id, sender_id, transaction_amount, lag(transaction_amount)over(partition by sender_id) as pervious_transaction_amount
from transactions)
select *, round((transaction_amount-pervious_transaction_amount),2) as difference from cte; 

```
**Insight**: Previous transction amount for each sender and the difference with the current amount.

---

### 17.Find the largest transaction per currency using MAX() OVER().

```sql
select distinct currency_code, max(transaction_amount)over(partition by currency_code) as max_transaction_amount
from transactions
order by max_transaction_amount desc; 

```
**Insight**: GBP, EUR and JPY has the largest transaction per currency.

---
### 18.Get cumulative transaction amounts per sender ordered by date (SUM() OVER()).

```sql
select sender_id, transaction_date, transaction_amount, sum(transaction_amount)over(partition by sender_id order by transaction_date) as cumulative_amount
from transactions;

```
**Insight**: Cumulative transaction amounts per sender according to transaction dates.

---

## Data Cleaning & Integrity

### 19.Find transactions with a blank currency_code.
```sql
select * from transactions
where currency_code =""; 
```
**Insight**: List of Transactions records witn no currency code.

---

### 20.Show transactions where the sender or recipient ID doesn’t exist in merchants.
```sql
SELECT t.* FROM transactions t
LEFT JOIN merchants m_sender ON t.sender_id = m_sender.merchant_id
LEFT JOIN merchants m_recipient ON t.recipient_id = m_recipient.merchant_id
WHERE m_sender.merchant_id IS NULL OR m_recipient.merchant_id IS NULL;
```

**Insight**: List of transactions without the sender's or recipient's ID.

---

## 🧠 Skills Demonstrated

- SQL Aggregations & Window Functions
- Joins and Subqueries
- Growth Rate Calculation with `LAG()`

---

## 🛠 Tech Stack

- MySQL
- Retail Dataset

---

## 🙋‍♂️ Author

**Suparna Guha**
[Github](https://github.com/Suparna16192/Paypal_case_study) || [LinkedIn](https://www.linkedin.com/in/suparna-guha-22220119b/)

