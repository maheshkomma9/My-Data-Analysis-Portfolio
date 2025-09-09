# Uber Ride Analysis (SQL)

This project analyzes Uber ride data using SQL. The dataset contains detailed booking information, including:

- Booking ID, Date, Time
- Customer and Vehicle details
- Pickup and Drop locations
- Booking value and ride distance
- Driver and customer ratings
- Payment method
- Booking status, cancellations, and incomplete rides

The goal of this project is to explore the data, generate actionable insights, and demonstrate advanced SQL skills such as subqueries, CTEs, and window functions.

---

## Key Queries & Insights

1. **Completed vs Cancelled Rides**  
   Analyze the proportion of completed bookings versus cancellations.

2. **Average Booking Value per Payment Method**  
   Understand which payment methods contribute the most to revenue.

3. **Top 5 Customers by Total Spend**  
   Identify the most valuable customers.

4. **Success Rate per Vehicle Type**  
   Measure the efficiency and reliability of each vehicle type.

5. **Peak Booking Hours**  
   Determine the busiest hours for Uber bookings.

6. **Moving Averages and Top-Ranked Customers per Vehicle Type**  
   Track trends over time and identify top spenders per vehicle type using window functions and ranking.

---

## SQL Features Used

- **Aggregations**: `SUM()`, `COUNT()`, `AVG()` for summarizing data.  
- **Subqueries & Nested Queries**: Compare customer totals against averages, calculate per-day metrics.  
- **Window Functions**: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `AVG() OVER(PARTITION BY...)` for running totals, moving averages, and ranking.  
- **CTEs (Common Table Expressions)**: Organize complex queries into readable steps.  
- **Conditional Aggregation**: `COUNT(CASE WHEN ...)` for success rates and cancellations.

---

## Full SQL Code

```sql
-- Create Database and Table
CREATE DATABASE IF NOT EXISTS uber_db;
USE uber_db;

DROP TABLE IF EXISTS uber_data;

CREATE TABLE uber_data (
    `Date` DATE,
    `Time` TIME,
    Booking_ID VARCHAR(12) PRIMARY KEY,
    Booking_Status VARCHAR(25),
    Customer_ID VARCHAR(12) NOT NULL,
    Vehicle_Type VARCHAR(15),
    Pickup_Location VARCHAR(30) NOT NULL,
    Drop_Location VARCHAR(30) NOT NULL,
    Avg_VTAT FLOAT,
    Avg_CTAT FLOAT,
    Cancelled_Rides_by_Customer INT,
    Reason_for_cancelling_by_Customer VARCHAR(50),
    Cancelled_Rides_by_Driver INT,
    Driver_Cancellation_Reason VARCHAR(50),
    Incomplete_Rides INT,
    Incomplete_Rides_Reason VARCHAR(50),
    Booking_Value INT,
    Ride_Distance FLOAT,
    Driver_Ratings FLOAT,
    Customer_Rating FLOAT,
    Payment_Method VARCHAR(20)
);

-- Preview first 5 rows
SELECT * FROM uber_data LIMIT 5;

-- 1. Completed bookings
SELECT * FROM uber_data WHERE Booking_Status = 'Completed';

-- 2. Customers who paid by UPI
SELECT Booking_ID, Customer_ID, Booking_Value 
FROM uber_data WHERE Payment_Method = 'UPI';

-- 3. Total bookings on a specific date
SELECT `Date`, COUNT(Booking_ID) AS Total_Bookings 
FROM uber_data WHERE `Date` = '2024-06-16';

-- 4. List distinct vehicle types
SELECT DISTINCT Vehicle_Type AS Vehicles FROM uber_data;

-- 5. Bookings with value greater than 500
SELECT Booking_ID, Pickup_Location, Drop_Location 
FROM uber_data WHERE Booking_Value > 500 ORDER BY Booking_ID;

-- 6. Total bookings per vehicle type
SELECT Vehicle_Type, COUNT(Booking_ID) AS total_bookings 
FROM uber_data GROUP BY Vehicle_Type ORDER BY total_bookings DESC;

-- 7. Average booking value per payment method
SELECT Payment_Method, ROUND(AVG(Booking_Value),2) AS avg_booking_value 
FROM uber_data WHERE Payment_Method IS NOT NULL 
GROUP BY Payment_Method;

-- 8. Max and Min ride distance for completed bookings
SELECT MAX(Ride_Distance) AS maximum_distance,
       MIN(Ride_Distance) AS minimum_distance 
FROM uber_data WHERE Booking_Status = 'Completed';

-- 9. Completed vs cancelled rides percentage
SELECT Booking_Status,
       COUNT(*) AS bookings,
       ROUND((COUNT(*) * 100.0 / (SELECT COUNT(*) FROM uber_data)), 2) AS Percentage
FROM uber_data GROUP BY Booking_Status;

-- 10. Average ratings per vehicle type
SELECT Vehicle_Type,
       ROUND(AVG(Customer_Rating),1) AS avg_customer_rating,
       ROUND(AVG(Driver_Ratings),1) AS avg_driver_rating
FROM uber_data GROUP BY Vehicle_Type;

-- 11. Top 5 customers by spend
SELECT Customer_ID, SUM(Booking_Value) AS total_spent
FROM uber_data GROUP BY Customer_ID ORDER BY total_spent DESC LIMIT 5;

-- 12. Most common reason for customer cancellation
SELECT COUNT(*) AS count, Reason_for_cancelling_by_Customer AS reason
FROM uber_data WHERE Reason_for_cancelling_by_Customer IS NOT NULL
GROUP BY Reason_for_cancelling_by_Customer
ORDER BY count DESC LIMIT 1;

-- 13. Total distance per vehicle type
SELECT Vehicle_Type, ROUND(SUM(Ride_Distance),2) AS total_distance
FROM uber_data GROUP BY Vehicle_Type ORDER BY total_distance DESC;

-- 14. Customers who cancelled more than 2 times
SELECT Customer_ID, COUNT(Cancelled_Rides_by_Driver) AS cancellations
FROM uber_data GROUP BY Customer_ID HAVING cancellations >= 2;

-- 15. Booking share per vehicle type
SELECT Vehicle_Type, COUNT(*) AS count,
       ROUND((COUNT(*) * 100.0 / (SELECT COUNT(*) FROM uber_data)),2) AS share
FROM uber_data WHERE Booking_Status='Completed' GROUP BY Vehicle_Type;

-- 16. Success rate per vehicle type
SELECT Vehicle_Type,
       COUNT(CASE WHEN Booking_Status='Completed' THEN 1 END) AS Completed_Bookings,
       COUNT(*) AS Total_Bookings,
       ROUND((COUNT(CASE WHEN Booking_Status='Completed' THEN 1 END) * 100.0 / COUNT(*)),2) AS Success_Rate
FROM uber_data GROUP BY Vehicle_Type;

-- 17. Peak booking hours
SELECT HOUR(`Time`) AS booking_hour, COUNT(Booking_ID) AS total_bookings
FROM uber_data GROUP BY HOUR(`Time`) ORDER BY total_bookings DESC LIMIT 5;

-- 18. Total revenue per payment method
SELECT Payment_Method, ROUND(SUM(Booking_Value),2) AS total_revenue
FROM uber_data WHERE Booking_Status='Completed' AND Payment_Method IS NOT NULL
GROUP BY Payment_Method ORDER BY total_revenue DESC;

-- 19. Top 3 pickup locations
SELECT Pickup_Location, COUNT(Booking_ID) AS total_bookings
FROM uber_data GROUP BY Pickup_Location ORDER BY total_bookings DESC LIMIT 3;

-- 20. Monthly bookings and revenue
SELECT MONTHNAME(`Date`) AS Month_Name, COUNT(Booking_ID) AS Total_Bookings, SUM(Booking_Value) AS Total_Revenue
FROM uber_data GROUP BY MONTH(`Date`), MONTHNAME(`Date`) ORDER BY MONTH(`Date`);

-- 21. Customers with total booking value above average
SELECT Customer_ID, SUM(Booking_Value) AS total_booking_value
FROM uber_data GROUP BY Customer_ID
HAVING SUM(Booking_Value) > (
    SELECT AVG(cust_booking_value)
    FROM (SELECT SUM(Booking_Value) AS cust_booking_value FROM uber_data GROUP BY Customer_ID) AS avg_table
) ORDER BY total_booking_value DESC;

-- 22. Ride distance greater than average per vehicle type
SELECT Vehicle_Type, Ride_Distance, ROUND(avg_distance,2) AS avg_distance
FROM (
    SELECT Vehicle_Type, Ride_Distance, AVG(Ride_Distance) OVER(PARTITION BY Vehicle_Type) AS avg_distance
    FROM uber_data WHERE Ride_Distance IS NOT NULL
) AS t WHERE Ride_Distance > avg_distance;

-- 23. Booking value greater than daily average
SELECT `Date`, Booking_Value, avg_booking
FROM (
    SELECT `Date`, Booking_Value, AVG(Booking_Value) OVER(PARTITION BY `Date`) AS avg_booking
    FROM uber_data WHERE Booking_Value IS NOT NULL
) AS a WHERE Booking_Value > avg_booking;

-- 24. Vehicle type generating highest revenue
SELECT Vehicle_Type, total_revenue
FROM (
    SELECT Vehicle_Type, SUM(Booking_Value) AS total_revenue,
           RANK() OVER(ORDER BY SUM(Booking_Value) DESC) AS ranked_revenue
    FROM uber_data GROUP BY Vehicle_Type
) AS t WHERE ranked_revenue = 1;

-- 25. Moving average for 7 days
SELECT Date, AVG(Booking_Value) OVER(ORDER BY Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_booking
FROM uber_data;

-- 26. Top 3 customers per vehicle type
WITH customer_spend AS (
    SELECT Customer_ID, Vehicle_Type, SUM(Booking_Value) AS total_spend
    FROM uber_data GROUP BY Customer_ID, Vehicle_Type
),
ranked_customers AS (
    SELECT Customer_ID, Vehicle_Type, total_spend,
           ROW_NUMBER() OVER(PARTITION BY Vehicle_Type ORDER BY total_spend DESC) AS rn
    FROM customer_spend
)
SELECT Customer_ID, Vehicle_Type, total_spend
FROM ranked_customers
WHERE rn <= 3
ORDER BY Vehicle_Type, total_spend DESC;
