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

## ER Diagram

[/.Bank_anomaly_data_analysis](ER.jpg)


## 📂 Full Code
```-- 🏦 BANK DATABASE PROJECT

-- Drop and recreate database
DROP DATABASE IF EXISTS Bank_DB;
CREATE DATABASE Bank_DB;
USE Bank_DB;

-- Create Main Table (Raw Data)

CREATE TABLE WHOLE_DATA (
    Customer_ID INT,
    First_Name VARCHAR(30),
    Last_Name VARCHAR(30),
    Age INT,
    Gender VARCHAR(10),
    Address VARCHAR(100),
    City VARCHAR(50),
    Contact_Number VARCHAR(20),
    Email VARCHAR(100),
    
    Account_ID INT,
    Account_Type VARCHAR(20),
    Account_Balance FLOAT,
    Date_Of_Account_Opening DATE,
    Last_Transaction_Date DATE,
    Transaction_ID INT,
    Transaction_Date DATE,
    Transaction_Type VARCHAR(20),
    Transaction_Amount FLOAT,
    Account_Balance_After_Transaction FLOAT,
    Branch_ID INT,

    Loan_ID INT,
    Loan_Amount FLOAT,
    Loan_Type VARCHAR(20),
    Interest_Rate FLOAT,
    Loan_Term INT,
    Approval_Rejection_Date DATE,
    Loan_Status VARCHAR(20),

    Card_ID INT,
    Card_Type VARCHAR(20),
    Credit_Limit FLOAT,
    Credit_Card_Balance FLOAT,
    Minimum_Payment_Due FLOAT,
    Payment_Due_Date DATE,
    Last_Credit_Card_Payment_Date DATE,
    Rewards_Points INT,

    Feedback_ID INT,
    Feedback_Date DATE,
    Feedback_Type VARCHAR(20),
    Resolution_Status VARCHAR(20),
    Resolution_Date DATE,

    Anomaly_Flag INT
);


-- 2️ Create Dimension Tables

-- Customers
CREATE TABLE Customers (
    Customer_ID INT PRIMARY KEY,
    First_Name VARCHAR(30),
    Last_Name VARCHAR(30),
    Age INT,
    Gender VARCHAR(10),
    Address VARCHAR(100),
    City VARCHAR(50),
    Contact_Number VARCHAR(20),
    Email VARCHAR(100)
);

INSERT INTO Customers
SELECT DISTINCT 
    Customer_ID, First_Name, Last_Name, Age, Gender,
    Address, City, Contact_Number, Email
FROM WHOLE_DATA;

-----------------------------------------------------
-- Accounts
-----------------------------------------------------
CREATE TABLE Accounts (
    Account_ID INT AUTO_INCREMENT PRIMARY KEY,
    Customer_ID INT,
    Account_Type VARCHAR(20),
    Account_Balance FLOAT,
    Date_Of_Account_Opening DATE,
    Last_Transaction_Date DATE,
    Branch_ID INT,
    FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID)
);

INSERT INTO Accounts
(Customer_ID, Account_Type, Account_Balance, Date_Of_Account_Opening, Last_Transaction_Date, Branch_ID)
SELECT DISTINCT Customer_ID, Account_Type, Account_Balance, Date_Of_Account_Opening, Last_Transaction_Date, Branch_ID
FROM WHOLE_DATA;

-----------------------------------------------------
-- Transactions
-----------------------------------------------------
CREATE TABLE Transactions (
    Transaction_ID INT PRIMARY KEY,
    Account_ID INT,
    Transaction_Date DATE,
    Transaction_Type VARCHAR(20),
    Transaction_Amount FLOAT,
    Account_Balance_After_Transaction FLOAT,
    Anomaly_Flag INT,
    FOREIGN KEY (Account_ID) REFERENCES Accounts(Account_ID)
);

INSERT INTO Transactions
(Transaction_ID, Account_ID, Transaction_Date, Transaction_Type, Transaction_Amount, Account_Balance_After_Transaction, Anomaly_Flag)
SELECT DISTINCT 
    s.Transaction_ID, a.Account_ID, s.Transaction_Date, s.Transaction_Type, 
    s.Transaction_Amount, s.Account_Balance_After_Transaction, s.Anomaly_Flag
FROM WHOLE_DATA s
JOIN Accounts a ON s.Customer_ID = a.Customer_ID;

-----------------------------------------------------
-- Loans
-----------------------------------------------------
CREATE TABLE Loans (
    Loan_ID INT PRIMARY KEY,
    Customer_ID INT,
    Loan_Amount FLOAT,
    Loan_Type VARCHAR(20),
    Interest_Rate FLOAT,
    Loan_Term INT,
    Approval_Rejection_Date DATE,
    Loan_Status VARCHAR(20),
    FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID)
);

INSERT INTO Loans
SELECT DISTINCT 
    Loan_ID, Customer_ID, Loan_Amount, Loan_Type, 
    Interest_Rate, Loan_Term, Approval_Rejection_Date, Loan_Status
FROM WHOLE_DATA
WHERE Loan_ID IS NOT NULL;

-----------------------------------------------------
-- Credit Cards
-----------------------------------------------------
CREATE TABLE Credit_Cards (
    Card_ID INT PRIMARY KEY,
    Customer_ID INT,
    Card_Type VARCHAR(20),
    Credit_Limit FLOAT,
    Credit_Card_Balance FLOAT,
    Minimum_Payment_Due FLOAT,
    Payment_Due_Date DATE,
    Last_Credit_Card_Payment_Date DATE,
    Rewards_Points INT,
    FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID)
);

INSERT INTO Credit_Cards
SELECT DISTINCT 
    Card_ID, Customer_ID, Card_Type, Credit_Limit, Credit_Card_Balance, 
    Minimum_Payment_Due, Payment_Due_Date, Last_Credit_Card_Payment_Date, Rewards_Points
FROM WHOLE_DATA
WHERE Card_ID IS NOT NULL;

-----------------------------------------------------
-- Feedback
-----------------------------------------------------
CREATE TABLE Feedback (
    Feedback_ID INT PRIMARY KEY,
    Customer_ID INT,
    Feedback_Date DATE,
    Feedback_Type VARCHAR(20),
    Resolution_Status VARCHAR(20),
    Resolution_Date DATE,
    FOREIGN KEY (Customer_ID) REFERENCES Customers(Customer_ID)
);

INSERT INTO Feedback
SELECT DISTINCT 
    Feedback_ID, Customer_ID, Feedback_Date, Feedback_Type, Resolution_Status, Resolution_Date
FROM WHOLE_DATA
WHERE Feedback_ID IS NOT NULL;

-----------------------------------------------------
-- 🔍 ANALYSIS QUERIES
-----------------------------------------------------

-- 1. Retrieve all customer names and cities
SELECT first_name, last_name, city FROM Customers;

-- 2. Show first 10 transactions sorted by date
SELECT transaction_id, transaction_date, transaction_amount
FROM Transactions
ORDER BY transaction_date DESC
LIMIT 10;

-- 3. Total number of customers
SELECT COUNT(*) AS total_customers FROM Customers;

-- 4. Distinct loan types
SELECT DISTINCT loan_type FROM Loans;

-- 5. Average age of customers by gender
SELECT gender, ROUND(AVG(age), 0) AS average_age FROM Customers GROUP BY gender;

-- 6. Total account balance per branch
SELECT branch_id, ROUND(SUM(account_balance), 2) AS total_balance
FROM Accounts
GROUP BY branch_id
ORDER BY total_balance DESC;

-- 7. Monthly trend of transactions
SELECT 
    DATE_FORMAT(transaction_date, '%Y-%m') AS month,
    COUNT(transaction_id) AS total_transactions,
    ROUND(SUM(transaction_amount), 2) AS total_value
FROM Transactions
GROUP BY month;

-- 8. Maximum loan amount and customer
SELECT 
    l.loan_amount, c.first_name, c.last_name
FROM Loans l
JOIN Customers c ON l.customer_id = c.customer_id
WHERE l.loan_amount = (SELECT MAX(loan_amount) FROM Loans);

-- 9. Each customer with account balance and loan amount
SELECT c.first_name, c.last_name, l.loan_amount
FROM Customers c
JOIN Loans l ON c.customer_id = l.customer_id
ORDER BY l.loan_amount DESC;

-- 10. Credit card utilization and spend category
SELECT 
    customer_id,
    credit_limit,
    credit_card_balance,
    ROUND((credit_card_balance / credit_limit) * 100, 2) AS utilization,
    CASE
        WHEN (credit_card_balance / credit_limit) * 100 > 70 THEN 'over_spend'
        WHEN (credit_card_balance / credit_limit) * 100 < 50 THEN 'less_spend'
        ELSE 'balanced'
    END AS credit_usage
FROM Credit_Cards
WHERE credit_limit >= credit_card_balance
ORDER BY utilization DESC;

-- 11. All transactions with customer name and branch
SELECT 
    t.transaction_id, t.transaction_date, t.transaction_amount,
    c.first_name, c.last_name, a.branch_id
FROM Transactions t
JOIN Accounts a ON t.account_id = a.account_id
JOIN Customers c ON a.customer_id = c.customer_id
ORDER BY t.transaction_date DESC;

-- 12. Customers who have both a loan and a credit card
WITH Loan_Customers AS (
    SELECT customer_id, first_name, last_name, loan_amount FROM Customers c JOIN Loans l ON c.customer_id = l.customer_id
),
Credit_Customers AS (
    SELECT customer_id, first_name, last_name, card_id, card_type FROM Customers c JOIN Credit_Cards cd ON c.customer_id = cd.customer_id
)
SELECT lc.*, cc.card_type
FROM Loan_Customers lc
JOIN Credit_Customers cc ON lc.customer_id = cc.customer_id;

-- 13. Number of loans approved vs rejected
SELECT 
    loan_status, 
    COUNT(*) AS total_count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM Loans), 2) AS percent
FROM Loans
GROUP BY loan_status;

-- 14. Flag customers as High Value if balance > 50,000
SELECT 
    account_id, account_balance,
    CASE WHEN account_balance > 50000 THEN 'High Value' ELSE 'Low Value' END AS account_value
FROM Accounts;

-- 15. Categorize loan amounts
SELECT 
    loan_id, loan_amount,
    CASE 
        WHEN loan_amount <= 10000 THEN 'Small Loan'
        WHEN loan_amount BETWEEN 10000 AND 50000 THEN 'Medium Loan'
        ELSE 'Large Loan'
    END AS loan_category
FROM Loans;

-- 16. Highest account balance in each branch
SELECT 
    c.first_name, c.last_name, a.branch_id, a.account_balance AS max_balance
FROM Accounts a
JOIN Customers c ON a.customer_id = c.customer_id
WHERE a.account_balance = (
    SELECT MAX(a2.account_balance) FROM Accounts a2 WHERE a2.branch_id = a.branch_id
);

-- 17. Loans with interest rate higher than average for their type
SELECT 
    l.loan_id, l.loan_type, l.interest_rate
FROM Loans l
JOIN (
    SELECT loan_type, AVG(interest_rate) AS avg_interest
    FROM Loans
    GROUP BY loan_type
) avg_table ON l.loan_type = avg_table.loan_type
WHERE l.interest_rate > avg_table.avg_interest;

-- 18. Customers whose credit card balance > average by card type
SELECT 
    c.first_name, c.last_name, cd.card_type, cd.credit_card_balance
FROM Customers c
JOIN Credit_Cards cd ON c.customer_id = cd.customer_id
JOIN (
    SELECT card_type, AVG(credit_card_balance) AS avg_balance
    FROM Credit_Cards
    GROUP BY card_type
) avg_table ON cd.card_type = avg_table.card_type
WHERE cd.credit_card_balance > avg_table.avg_balance
ORDER BY cd.credit_card_balance;

-- 19. Rank customers by account balance within each branch
WITH Customer_Rank AS (
    SELECT 
        c.first_name, c.last_name, a.branch_id, a.account_balance,
        RANK() OVER (PARTITION BY a.branch_id ORDER BY a.account_balance DESC) AS rank_no
    FROM Customers c
    JOIN Accounts a ON c.customer_id = a.customer_id
)
SELECT * FROM Customer_Rank;

-- 20. Top 3 largest transactions per customer
WITH Customer_Transactions AS (
    SELECT 
        c.customer_id, c.first_name, c.last_name,
        t.transaction_id, t.transaction_amount,
        RANK() OVER (PARTITION BY c.customer_id ORDER BY t.transaction_amount DESC) AS rank_no
    FROM Customers c
    JOIN Accounts a ON c.customer_id = a.customer_id
    JOIN Transactions t ON a.account_id = t.account_id
)
SELECT * 
FROM Customer_Transactions
WHERE rank_no <= 3
ORDER BY first_name, last_name, rank_no;

-- 21. Running total of transaction amounts by date
SELECT 
    transaction_date,
    transaction_amount,
    SUM(transaction_amount) OVER (ORDER BY transaction_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM Transactions;

-- 22. Average loan amount per type and difference from average
SELECT 
    loan_type,
    loan_amount,
    ROUND(avg_loan_amount_per_type, 2) AS avg_loan_amount,
    ROUND(difference_from_avg, 2) AS difference
FROM (
    SELECT 
        loan_type,
        loan_amount,
        AVG(loan_amount) OVER (PARTITION BY loan_type) AS avg_loan_amount_per_type,
        loan_amount - AVG(loan_amount) OVER (PARTITION BY loan_type) AS difference_from_avg
    FROM Loans
) AS t;

-- 23. Months with >400 transactions
WITH total_transaction_per_month AS (
    SELECT MONTH(transaction_date) AS months,
           COUNT(transaction_id) AS total_transactions
    FROM Transactions
    GROUP BY months
)
SELECT * 
FROM total_transaction_per_month
WHERE total_transactions > 400;

-- 24. Customers with more than one loan
WITH customers_loan AS (
    SELECT c.first_name, c.last_name, COUNT(l.loan_id) AS total_loans
    FROM Loans l
    JOIN Customers c ON l.customer_id = c.customer_id
    GROUP BY c.first_name, c.last_name
)
SELECT * FROM customers_loan WHERE total_loans > 1;

-- 25. Customers who missed credit card payments
WITH customers_missed_credit_payment AS (
    SELECT 
        customer_id, credit_card_balance, minimum_payment_due, 
        payment_due_date, last_credit_card_payment_date
    FROM Credit_Cards
    WHERE last_credit_card_payment_date IS NULL 
       OR last_credit_card_payment_date > payment_due_date
)
SELECT 
    c.first_name, c.last_name, c.contact_number, c.email, cm.*
FROM Customers c
JOIN customers_missed_credit_payment cm 
    ON c.customer_id = cm.customer_id;

-- 26. Branches with anomaly count
WITH branch_anomalies AS (
    SELECT 
        a.branch_id,
        COUNT(*) AS total_anomalies,
        ROUND(SUM(t.transaction_amount), 2) AS total_anomaly_amount
    FROM Accounts a
    JOIN Transactions t ON a.account_id = t.account_id
    WHERE t.anomaly_flag = -1
    GROUP BY a.branch_id
)
SELECT * 
FROM branch_anomalies
ORDER BY total_anomalies DESC;

-- 27. Average feedback resolution time & branch ranking
WITH resolved_feedback AS (
    SELECT 
        customer_id, feedback_type, resolution_status,
        DATEDIFF(resolution_date, feedback_date) AS duration
    FROM Feedback
    WHERE resolution_status = 'resolved' 
      AND resolution_date > feedback_date
),
branch_join AS (
    SELECT rf.*, a.branch_id
    FROM resolved_feedback rf
    JOIN Accounts a ON rf.customer_id = a.customer_id
),
avg_resolution_rank AS (
    SELECT 
        branch_id,
        AVG(duration) AS avg_resolution_days,
        RANK() OVER (ORDER BY AVG(duration)) AS branch_rank
    FROM branch_join
    GROUP BY branch_id
)
SELECT * FROM avg_resolution_rank;

-- 28. Top 5 customers by combined revenue
WITH customer_transaction AS (
    SELECT 
        c.customer_id, c.first_name, c.last_name,
        SUM(t.transaction_amount) AS total_transactions,
        SUM(l.loan_amount) AS total_loans,
        SUM(cd.credit_card_balance) AS total_credit_balance
    FROM Customers c
    JOIN Accounts a ON c.customer_id = a.customer_id
    JOIN Transactions t ON a.account_id = t.account_id
    JOIN Loans l ON l.customer_id = c.customer_id
    JOIN Credit_Cards cd ON cd.customer_id = c.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT 
    first_name, last_name,
    ROUND(total_transactions + total_loans + total_credit_balance, 2) AS total_value
FROM customer_transaction
ORDER BY total_value DESC
LIMIT 5;

-- 29 Find customers with multiple anomalies flagged
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    COUNT(t.transaction_id) AS total_anomalies,
    ROUND(SUM(t.transaction_amount), 2) AS total_transaction_amount
FROM accounts a
JOIN transactions t 
    ON a.account_id = t.account_id
JOIN customers c 
    ON a.customer_id = c.customer_id
WHERE t.anomaly_flag = -1                 -- Only count actual anomalies
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_anomalies DESC, total_transaction_amount DESC;

-- 30 Find branches with the highest number of anomalies
WITH branch_anomalies AS (
    SELECT 
        a.branch_id,
        COUNT(*) AS total_anomalies,
        ROUND(SUM(t.transaction_amount), 2) AS total_anomaly_amount
    FROM accounts a
    JOIN transactions t 
        ON a.account_id = t.account_id
    WHERE t.anomaly_flag = -1
    GROUP BY a.branch_id
)
SELECT 
    branch_id,
    total_anomalies,
    total_anomaly_amount
FROM branch_anomalies
ORDER BY total_anomalies DESC, total_anomaly_amount DESC;


