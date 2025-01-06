# Super-market Sales Data Analysis using PostgreSQL

## Overview
This project involves a comprehensive analysis of Super-market sale data using PostgreSQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, problems and solutions.


## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Students_mental_health](https://www.kaggle.com/datasets/arunjangir245/super-market-sales)

## CREATING_TABLE

```sql
DROP TABLE IF EXISTS super_market;
CREATE TABLE super_market (
    Invoice_ID VARCHAR(15) PRIMARY KEY,
    Branch CHAR(1),
    City VARCHAR(15),
    Customer_type VARCHAR(15),
    Gender VARCHAR(10),
    Product_line VARCHAR(25),
    Unit_price DECIMAL,
    Quantity INT,
    Tax_5_percent DECIMAL,
    Total DECIMAL,
    Sale_Date DATE,
    Sale_Time TIME,
    Payment VARCHAR(25),
    cogs DECIMAL,
    gross_income DECIMAL,
    Rating DECIMAL
);
```
--Check column data type
SELECT * FROM super_market;


## Problems and Solutions 

### (1) Finding average customer rating of product_line by branch.

```sql
SELECT CONCAT(city,' branch'), product_line, 
    ROUND(AVG(rating),3) AS average_rating
FROM super_market
    GROUP BY 1,2
    ORDER BY 1, 3 DESC ;
```


### (2) Finding total sale count of all prouct_line by time periods(Morning, Afternoon and evening).

```sql
WITH CTE_Time_periods_order AS (
SELECT 
    product_line,
    CASE 
        WHEN sale_time < '12:00:00' THEN 'Morning'
        WHEN sale_time BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
        WHEN sale_time BETWEEN '18:00:00' AND '20:59:59' THEN 'Evening'
        ELSE 'Night' 
    END AS time_periods,
    COUNT(sale_time) AS total_sale_count
FROM super_market
    GROUP BY 1, 2 )
SELECT * 
FROM CTE_time_periods_order
     ORDER BY 1,
        CASE 
            WHEN time_periods = 'Morning' THEN 1
            WHEN time_periods = 'Afternoon' THEN 2
            WHEN time_periods = 'evening' THEN 3
            ELSE 4 
         END NULLS LAST ;
```


### (3) finding total sale item count by payment type in 2019

```sql
SELECT 
    EXTRACT (year FROM sale_date) AS year,
    payment AS payment_type,
    SUM(quantity) AS total_item
FROM super_market
    GROUP BY 1, 2 ;
```


### (4) Finding total sale amount and items on each product line by gender

```sql
SELECT product_line,
    gender,
    COUNT(quantity) AS total_item,
    SUM(total) AS total_Sale_amount
FROM super_market
    GROUP BY 1, 2
    ORDER BY 1, 3 DESC ;

```


### (5) Finding total gross income on each product by date.
 -And giving bonus to staff on gross income --> Electronic accessories = 5%, Fashion accessories= 8%,
 -Food and beverages =5%, Health and beauty =6%, Home and lifestyle=10%, Sports and travel=20%,
 -also finding final_gross_income(total_gross_income - giving_bonus_to_staff) 

```sql
WITH CTE_Final_gross AS (
    SELECT 
        date,
        product_line,
        total_gross_income,
        CASE 
            WHEN product_line = 'Electronic accessories' THEN total_gross_income * 0.05
            WHEN product_line = 'Fashion accessories' THEN total_gross_income *0.08
            WHEN product_line = 'Food and beverages' THEN total_gross_income *0.05
            WHEN product_line = 'Health and beauty' THEN total_gross_income *0.06
            WHEN product_line = 'Home and lifestyle' THEN total_gross_income *0.1
            WHEN product_line = 'Sports and travel' THEN total_gross_income *0.2
        END AS giving_bonus_to_staff
    FROM (
            SELECT 
                TO_CHAR(sale_date, 'YYYY-MM') AS Date,
                product_line,
                SUM(gross_income) AS total_gross_income
            FROM super_market
                GROUP BY 1, 2
                ORDER BY 2, 1 ) AS total_gross
        ORDER BY 1, 3 DESC )
SELECT *,
    total_gross_income - giving_bonus_to_staff AS Final_Gross_income
FROM CTE_final_gross ;
```


### (6) Finding total customer and total spent amount on customer type by gender

```sql
SELECT gender, customer_type,
    COUNT(customer_type) AS total_customer,
    SUM(total) AS total_amount 
FROM super_market 
    GROUP BY 1, 2
    ORDER BY  1 ; 
```


### (7) Finding average spend for food on gender by city

```sql
SELECT city, gender, ROUND(AVG(total), 2) AS average_spend_for_food
FROM super_market
    GROUP BY 1, 2
    ORDER BY 1, 3 DESC ;
```

### (8) if supermarket gift to customers 
-total quanties greater than 5 gift unbrella in mandalay
-total quanties greater than 5 gift tote bag in Yangon
-total quantites graeter than 3 gift discount cupon in naypyidaw
-finding gross income by deducing promption amount(unbreall=3, tote bag=1.5, cupon=0.5).

```sql
WITH CTE_gift AS (
    SELECT city, quantity,
        CASE 
            WHEN quantity > 5 AND city = 'Mandalay' THEN 'unbrella'
            WHEN quantity > 5 AND city = 'Yangon' THEN 'Tote bag'
            WHEN quantity > 3 AND city = 'Naypyitaw' THEN 'Discount Cupon'
            ELSE 'Tissue'
        END AS gift_item, gross_income
    FROM super_market )
SELECT *,
    CASE 
        WHEN gift_item = 'unbrella' THEN gross_income - 3
        WHEN gift_item = 'Tote bag' THEN gross_income - 1.5
        WHEN gift_item = 'Discount Cupon' THEN gross_income - 0.5
    ELSE 0.00
    END AS Final_gross_income
FROM CTE_gift ;

```


### (9) Giving customers 10% off who make purchase with Ewallet in Afternoon

```sql
SELECT *, ROUND(total * 0.9, 2) AS Ten_Percent_discounted_amount
FROM 
    (SELECT payment,
        CASE 
            WHEN sale_time < '12:00:00' THEN 'Morning'
            WHEN sale_time BETWEEN '12:00:00' AND '17:59:59' THEN 'Afternoon'
            WHEN sale_time BETWEEN '18:00:00' AND '20:59:59' THEN 'Evening'
            ELSE 'Night' 
        END AS time_periods, total
    FROM super_market) AS time_query
WHERE time_periods = 'Afternoon' 
    AND payment = 'Ewallet' ;
```


### (10) Ranking gross income by highest to lowest 

```sql
SELECT city, product_line, quantity, gross_income, 
    DENSE_RANK () OVER(ORDER BY gross_income DESC) AS gross_income_ranking
FROM super_market ;
```


## Conclusion

This analysis provides a comprehensive view of Super-market's sale content and can help inform content strategy and decision-making.

