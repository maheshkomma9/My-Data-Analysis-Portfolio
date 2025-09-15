# 🧑‍💼 HR Data Analysis (SQL)

This project analyzes HR employee data using SQL.  
The dataset contains details such as employee demographics, job roles, performance, engagement, training programs, and costs.  
The goal is to generate insights that help HR teams make better workforce and training decisions.

---

## 📂 Dataset Overview
Columns include:
- **Employee Info**: Employee ID, Title, Business Unit, Department, Division, Age, Gender, Race, Marital Status  
- **Job & Performance**: Start Date, Employee Status, Employee Type, Pay Zone, Performance Score, Employee Rating  
- **Engagement & Survey**: Survey Date, Engagement Score, Satisfaction Score, Work-Life Balance Score  
- **Training Data**: Training Date, Program Name, Training Type, Training Outcome, Training Duration, Training Cost  

---

## 🔑 Key Queries & Insights

### 👨‍💻 Basic Analysis
- Active employees  
- Unique job titles  
- Count employees by status (Active vs Inactive)  
- Employees working in analyst/accounting roles  

### 👥 Demographics & Engagement
- Average age of employees by department  
- Average satisfaction score by marital status  
- Workforce distribution by gender & race  
- Engagement scores across business units  

### 📈 Performance
- Employees exceeding expectations  
- Correlation between performance score and engagement/satisfaction  

### 🎓 Training Analysis
- Top 5 employees with highest training costs  
- Department with highest total training cost  
- Training program cost & duration summary  
- Training success rate per program  
- Employees who failed but had above-average training cost  

### 🪟 Advanced Window Functions
- Top 3 employees with highest training cost per department  
- Running total of training costs by department over time  
- Cumulative average engagement scores across survey dates  
- Ranking business units by average satisfaction  

### 👵 Workforce Analytics
- Employees nearing retirement (Age > 58)  
- Employee tenure (years worked since Start Date)  
- Top 5 employees with highest engagement  

---

## 🛠️ SQL Features Used
- **Aggregations**: `SUM`, `AVG`, `COUNT`, `ROUND`  
- **Subqueries & Correlated Subqueries**  
- **Window Functions**: `ROW_NUMBER()`, `RANK()`, `SUM() OVER()`, `AVG() OVER()`  
- **CTEs (Common Table Expressions)** for step-by-step analysis  
- **Conditional Aggregation** using `CASE WHEN`  

---

## 📌 Full Code
```
#Create Database & Table
CREATE DATABASE HR_database;
DROP TABLE IF EXISTS hr_data;
USE HR_database;

CREATE TABLE hr_data (
    Employee_ID INT PRIMARY KEY,
    StartDate VARCHAR(15),
    Title VARCHAR(30),
    BusinessUnit VARCHAR(20),
    Employee_Status VARCHAR(10),
    Employee_Type VARCHAR(10),
    PayZone VARCHAR(10),
    EmployeeClassificationType VARCHAR(15),
    DepartmentType VARCHAR(20),
    Division VARCHAR(30),
    DOB VARCHAR(15),
    State VARCHAR(10),
    Gender VARCHAR(10),
    Race VARCHAR(10),
    Marital_Status VARCHAR(10),
    Performance_Score VARCHAR(20),
    Current_Employee_Rating FLOAT,
    Survey_Date VARCHAR(15),
    Engagement_Score FLOAT,
    Satisfaction_Score FLOAT,
    Work_Life_Balance_Score FLOAT,
    Training_Date VARCHAR(15),
    Training_Program_Name VARCHAR(30),
    Training_Type VARCHAR(10),
    Training_Outcome VARCHAR(10),
    Training_Duration INT,
    Training_Cost FLOAT,
    Age INT
);

# Preview data
SELECT * FROM hr_data LIMIT 5;

#1 Active employees
SELECT Employee_ID, Employee_Status 
FROM hr_data 
WHERE Employee_Status = 'Active';

#2 Unique job titles
SELECT DISTINCT Title 
FROM hr_data 
ORDER BY Title;

#3 Employees in analyst or accounting
SELECT Employee_ID, Title 
FROM hr_data 
WHERE Title LIKE '%Data Analyst%' OR Title LIKE '%Accountant%';

#4 Count employees by status
SELECT Employee_Status, COUNT(*) AS total_count,
       ROUND((COUNT(*) * 100.0 / (SELECT COUNT(*) FROM hr_data)), 2) AS percentage
FROM hr_data 
GROUP BY Employee_Status;

#5 Average age by department
SELECT DepartmentType, ROUND(AVG(Age),0) AS avg_age
FROM hr_data 
GROUP BY DepartmentType 
ORDER BY avg_age;

#6 Avg satisfaction by marital status
SELECT Marital_Status, ROUND(AVG(Satisfaction_Score),1) AS avg_satisfaction
FROM hr_data 
GROUP BY Marital_Status;

#7 Distribution by gender & race
SELECT Gender, Race, COUNT(*) AS total_count,
       ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM hr_data),2) AS percentage
FROM hr_data 
GROUP BY Gender, Race;

#8 Avg engagement score by business unit
SELECT BusinessUnit, ROUND(AVG(Engagement_Score),2) AS avg_engagement
FROM hr_data 
GROUP BY BusinessUnit;

#9 Employees exceeding expectations
SELECT Employee_ID, Title 
FROM hr_data 
WHERE Performance_Score = 'Exceeds';

#10 Correlation check: performance vs engagement
SELECT Performance_Score,
       ROUND(AVG(Engagement_Score),2) AS avg_engagement,
       ROUND(AVG(Satisfaction_Score),2) AS avg_satisfaction
FROM hr_data
GROUP BY Performance_Score
ORDER BY avg_engagement DESC;

#11 Top 3 employees with highest training cost per department
WITH ranked_training AS (
    SELECT Employee_ID, DepartmentType, Training_Program_Name, Training_Cost,
           ROW_NUMBER() OVER (PARTITION BY DepartmentType ORDER BY Training_Cost DESC) AS rn
    FROM hr_data
)
SELECT Employee_ID, DepartmentType, Training_Program_Name, Training_Cost, rn
FROM ranked_training
WHERE rn <= 3
ORDER BY DepartmentType, rn;

#12 Running total of training cost by department
SELECT DepartmentType, Training_Date,
       SUM(Training_Cost) OVER (
           PARTITION BY DepartmentType
           ORDER BY Training_Date
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS dept_running_total
FROM hr_data;

#13 Cumulative average engagement over survey dates
SELECT Survey_Date,
       ROUND(AVG(Engagement_Score) OVER (
           ORDER BY Survey_Date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ),2) AS cumulative_avg
FROM hr_data;

#14 Rank business units by avg satisfaction
SELECT BusinessUnit,
       ROUND(AVG(Satisfaction_Score),2) AS avg_satisfaction,
       RANK() OVER (ORDER BY AVG(Satisfaction_Score) DESC) AS satisfaction_rank
FROM hr_data
GROUP BY BusinessUnit;

#15 Employees nearing retirement (Age > 58)
SELECT Employee_ID, Title, Age, DepartmentType
FROM hr_data
WHERE Age > 58
ORDER BY Age DESC;

#16 Top 5 employees with highest engagement
SELECT Employee_ID, Engagement_Score, Satisfaction_Score
FROM hr_data
ORDER BY Engagement_Score DESC
LIMIT 5;'''
