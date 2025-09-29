# plsql-window-functions-Divince-Cyusa
# Overview
#### Prime Suit is an online suit rental company that allows customers to borrow and return suits without physically visiting the shop.

# Step 1: Problem Definition

## Business Context:
Prime Suit is a digital suit rental service that provides customers with a convenient way to borrow and return suits without the need to visit a physical store. Through the online platform, users can browse a wide selection of formal wear, choose the style and size that fits their needs, and have it delivered directly to their doorstep. Once the event or occasion is over, the suit can be easily returned using the company’s return process, saving customers both time and effort. This model combines the flexibility of e-commerce with the practicality of clothing rentals, making formal attire more accessible and hassle-free.

## Data Challenge:
In normal way, customers have to visit the shop in person to select and rent a suit. However, with our online platform, the entire process is done digitally — customers can place their order online, and we deliver the suit directly to their location, saving them the time and effort of coming to the store. 

## Expected Outcome:
Gain insights into customer borrowing frequency, identify the most popular suits, track monthly rental trends, and segment customers for better promotions and inventory management.

## Prime Suit Database Schema
### Customer Table
```
CREATE TABLE customers (
    customer_id   NUMBER PRIMARY KEY,
    name          VARCHAR2(40) NOT NULL,
    region        VARCHAR2(50),
    phonenumber   NUMBER(15) 
);
```
### Suits Table

```CREATE TABLE suits (
    suit_id       NUMBER PRIMARY KEY,
    name          VARCHAR2(100) NOT NULL,
    category      VARCHAR2(50)
);
```
### Rentals Table 

```
CREATE TABLE rentals (
    rental_id     NUMBER PRIMARY KEY,
    customer_id   NUMBER REFERENCES customers(customer_id),
    suit_id       NUMBER REFERENCES suits(suit_id),
    rental_date   DATE NOT NULL,
    return_date   DATE,
    amount        NUMBER(10,2)
);
```
Clarification about the tables I have used:

 #### 🧑‍💼Customers Table
Stores all customer-related information, including personal details and contact information.

#### 🏠Rentals Table
Holds comprehensive details about all rental transactions, such as rental dates, items rented, and payment status.

#### 👗Suits Table
Contains information about all suits, including type, styles.

# Queries 

**1.Ranking**: Getting first 3 customers based on total spent(in USD) on rental suit.

```
SELECT name, total_spent, rank
FROM (
    SELECT 
        c.name,
        SUM(r.amount) AS total_spent,
        RANK() OVER (ORDER BY SUM(r.amount) DESC) AS rank
    FROM Customers c
    JOIN Rentals r ON c.customer_id = r.customer_id
    GROUP BY c.name
)
WHERE ROWNUM <= 3
ORDER BY total_spent DESC;
```

***2. Aggregate***: Show total amount per month and running total up to that month.

```
SELECT 
    TO_CHAR(r.rental_date, 'YYYY-MM') AS month,
    SUM(r.amount) AS monthly_total,
    SUM(SUM(r.amount)) OVER (ORDER BY TO_CHAR(r.rental_date, 'YYYY-MM')) AS running_total
FROM rentals r
GROUP BY TO_CHAR(r.rental_date, 'YYYY-MM')
ORDER BY month;
```
***3. Navigation***: Calculate month-over-month growth according to how suits are being rented.

```
WITH monthly AS (
    SELECT 
        TRUNC(rental_date, 'MM') AS month,
        SUM(amount) AS monthly_total
    FROM Rentals
    GROUP BY TRUNC(rental_date, 'MM')
)
SELECT 
    month,
    monthly_total,
    LAG(monthly_total) OVER (ORDER BY month) AS prev_month,
    ROUND(
        ( (monthly_total - LAG(monthly_total) OVER (ORDER BY month)) 
          / NULLIF(LAG(monthly_total) OVER (ORDER BY month), 0) ) * 100,
        2
    ) AS growth_percent
FROM monthly
ORDER BY month;
```
***4. Distribution***: Divide customers into 4 quartiles based on total spending.

```
SELECT 
    c.name,
    SUM(r.amount) AS total_spent,
    NTILE(4) OVER (ORDER BY SUM(r.amount) DESC) AS quartile
FROM Rentals r
JOIN Customers c ON r.customer_id = c.customer_id
GROUP BY c.name;
```
## Results Analysis

#### 1. Descriptive – What happened?
The most popular suits are spread across different categories (formal, business, wedding), with clear customer preferences visible in the rental frequency.
Customer spending shows distinct segments: some customers rent frequently and spend more (top quartile), while others rent only once or twice (bottom quartile).
Monthly revenue analysis shows a steady growth. Outliers like unusually high rental amounts may indicate premium suit rentals.

#### 2. Diagnostic – Why did it happen?

Higher revenue months correspond to special seasons/events (e.g., weddings, graduations), which increase demand for suits.
Loyal customers in the top quartile are often from urban regions (like Kigali), suggesting regional influence on rental frequency.
Lower-spending customers may rent less due to distance from service centers, price sensitivity, or fewer occasions needing formal wear.
Suit categories influence rental amounts: wedding suits tend to cost more, raising the average rental fee.

#### 3. Prescriptive – What next?

Inventory Strategy: Keep more stock of high-demand suits (e.g., wedding suits before peak season) to avoid shortages.
Customer Engagement: Reward top-quartile customers (loyal renters) with discounts or loyalty points to retain them.
Regional Marketing: Run targeted promotions in regions with low rentals (like rural areas) to increase customer activity.


## References

1. TutorialsPoint – [SQL Aggregate & Window Functions](https://www.tutorialspoint.com/sql/sql-window-functions.htm)  
2. Kaggle – [Customer Segmentation using SQL](https://www.kaggle.com/code)  
3. FreeCodeCamp – [SQL Window Functions Guide](https://www.freecodecamp.org/news/sql-window-functions/)  
4. Academic Paper: “Window Functions in OLAP Analytics” – [Link](https://www.sciencedirect.com/)  
5. Stack Overflow Discussions – [SQL Window Functions Examples](https://stackoverflow.com/questions/tagged/sql-window-functions)  
6. Oracle Live SQL – [Window Functions Examples](https://livesql.oracle.com/)  
7. Oracle.com [Oracle](https://docs.oracle.com/cd/E17952_01/mysql-8.0-en/window-functions-usage.html)
8. GeeksForGeeks [GeeksForGeeks](https://www.geeksforgeeks.org/plsql/window-functions-in-plsql/)
9. PostgreSQL Official Documentation – [Window Functions](https://www.postgresql.org/docs/current/functions-window.html)  
10. Oracle PL/SQL Documentation – [Analytic Functions](https://docs.oracle.com/cd/B28359_01/server.111/b28286/functions001.htm)  


All sources are properly cited. Implementations and analysis represent original work. No AI-generated content was copied without attribution or adaptation.

---
## Academic Integrity Statement

**Professional & Ethical Note:**  
*"Whoever is faithful in very little is also faithful in much." – Luke 16:10*  
As database professionals, uphold accuracy, confidentiality, and integrity. The reputation is built on consistent honesty, quality, and responsibility.












