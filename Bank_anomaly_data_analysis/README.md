# 🏦 Bank Database Project

## Overview
This project is a comprehensive **Banking Database Analysis** using SQL. The dataset includes customer information, accounts, transactions, loans, credit cards, and feedback. The project demonstrates advanced SQL queries, CTEs, window functions, aggregation, and anomaly detection.

**Database Name:** `Bank_DB`

**Tables:**
1. `Customers` – Customer demographic information.
2. `Accounts` – Bank accounts and balances.
3. `Transactions` – Transaction history with anomaly flags.
4. `Loans` – Loan details including type and interest rates.
5. `Credit_Cards` – Credit card information and balances.
6. `Feedback` – Customer feedback and resolution tracking.

---

## ✅ Features and Analysis Queries

### 1. Customer and Account Insights
- Retrieve all customer names and cities.
- Average age of customers by gender.
- Flag customers as "High Value" if account balance > 50,000.

### 2. Transaction Analysis
- Show first 10 transactions sorted by date.
- Monthly transaction trends.
- Top 3 largest transactions per customer.
- Running total of transaction amounts by date.
- Detect anomalies: transactions flagged as unusual (anomaly_flag = -1).

### 3. Loan and Credit Card Insights
- List distinct loan types.
- Maximum loan amount and corresponding customer.
- Categorize loan amounts (Small, Medium, Large).
- Credit card utilization and spend categories.
- Customers who have both a loan and a credit card.
- Loans with interest rate higher than type average.

### 4. Advanced Analytics
- Customers with multiple anomalies flagged.
- Branches with highest anomalies.
- Average feedback resolution time by branch.
- Top 5 customers generating highest combined revenue (transactions + loans + credit card balances).

---

## 🔧 SQL Techniques Used
- **CTEs (Common Table Expressions)**
- **Window Functions** (`RANK()`, `SUM() OVER()`)
- **Aggregation** (`SUM`, `AVG`, `COUNT`)
- **Joins** (`INNER JOIN`, `LEFT JOIN`)
- **Conditional Logic** (`CASE WHEN`)
- Filtering with `HAVING` and `WHERE`
- Subqueries for advanced comparisons

---

## 🛠️ How to Run
1. Import the `WHOLE_DATA` CSV file into MySQL.
2. Run the provided SQL scripts to create tables and insert data.
3. Execute the analysis queries sequentially for insights.





## 📂 Project Structure
