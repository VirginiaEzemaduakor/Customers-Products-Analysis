# Customers-Products-Analysis

## Table of contents
- [Introduction](#introduction)
   - [Schema diagram](#schema-diagram)
- [Database summary](#database-summary)
- [Analysis](#analysis)
- [Conclusion](#conclusion)


---

## Introduction
The goal of this project is to analyze data from a sales records database for scale model cars and extract information for decision-making.

Below are the questions we want to answer for this project. 

1. Which products should we order more of or less of?
2. How should we tailor marketing and communication strategies to customer behaviors?
3. How much can we spend on acquiring new customers?


### Schema diagram
![Alt text](<image/database schema.PNG>)
## Database Summary

This project made us of DB Browser for SQLite to handle the databases and were queried with different codes to obtain insights necessary to increase donations.
![Alt text](<image/database summary.PNG>)

 
               
## Analysis
### **1. Products to order more of or less of**

This refers to inventory reports, including low stock(i.e. product in demand) and product performance. This will optimize the supply and the user experience by preventing the best-selling products from going out-of-stock.

#### Low stock
The low stock represents the quantity of the sum of each product ordered divided by the quantity of product in stock. We can consider the ten highest rates. These will be the top ten products that are almost out-of-stock or completely out-of-stock.

``` 
SELECT productCode, 
       ROUND(SUM(quantityOrdered) * 1.0 / (SELECT quantityInStock
                                             FROM products p
                                            WHERE od.productCode = p.productCode), 2) AS low_stock
  FROM orderdetails od
 GROUP BY productCode
 ORDER BY low_stock DESC
 LIMIT 10;
```
![Alt text](image/low_stock.PNG)


#### Product performance
The product performance represents the sum of sales per product.

```
SELECT productCode, 
       SUM(quantityOrdered * priceEach) AS prod_perf
  FROM orderdetails od
 GROUP BY productCode 
 ORDER BY prod_perf DESC
 LIMIT 10;
```
![Alt text](image/product_performance.PNG)

#### Priority products for restocking
Priority products for restocking are those with high product performance that are on the brink of being out of stock.

``` 
WITH
  prfrm AS (
    SELECT productCode,
           SUM(quantityOrdered) * 1.0 AS qntOrdr,	        SUM(quantityOrdered * priceEach) AS prod_perf
      FROM orderdetails
     GROUP BY productCode
  ),
  lstk AS (
    SELECT pr.productCode, 
	       pr.productName, 
		   pr.productLine,
           ROUND(SUM(prfrm.qntOrdr * 1.0) / pr.quantityInstock, 2) AS low_stock
      FROM products pr
	  JOIN prfrm
	    ON pr.productCode = prfrm.productCode
     GROUP BY pr.productCode
	 ORDER BY low_stock
	 LIMIT 10
  )
    SELECT lstk.productName, 
	       lstk.productLine
	  FROM lstk
	  JOIN prfrm
	    ON lstk.productCode = prfrm.productCode
	 ORDER BY prfrm.prod_perf DESC;
```
![Alt text](image/priority_product_for_restocking.PNG)


   
### 2. **Tailoring marketing and communication strategies to customer behaviors**
This involves categorizing customers: finding the VIP (very important person)customers (that bring in the most profit for the store) and those who are less engaged (customers that bring in less profit).

#### Revenue by customer
How much revenue or profit each customer generates for the store.

``` 
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber;
 ``` 
 

#### Top 5 VIP customers 
```
WITH 

money_in_by_customer_table AS (
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber
)

SELECT contactLastName, contactFirstName, city, country, mc.revenue
  FROM customers c
  JOIN money_in_by_customer_table mc
    ON mc.customerNumber = c.customerNumber
 ORDER BY mc.revenue DESC
 LIMIT 5;
 ```
 ![Alt text](image/top_5_vip_customer.PNG)


#### Top 5 least-engaged customers

```
WITH 

money_in_by_customer_table AS (
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber
)

SELECT contactLastName, contactFirstName, city, country, mc.revenue
  FROM customers c
  JOIN money_in_by_customer_table mc
    ON mc.customerNumber = c.customerNumber
 ORDER BY mc.revenue
 LIMIT 5;
 
```
![Alt text](image/top_5_least_engaged_customer.PNG)


### 3. **How much can we spend on acquiring new customers**
To determine how much money we can spend acquiring new customers, we can compute the Customer Lifetime Value (LTV), which represents the average amount of money a customer generates. 

#### Customer lifetime value

```
WITH 

money_in_by_customer_table AS (
SELECT o.customerNumber, SUM(quantityOrdered * (priceEach - buyPrice)) AS revenue
  FROM products p
  JOIN orderdetails od
    ON p.productCode = od.productCode
  JOIN orders o
    ON o.orderNumber = od.orderNumber
 GROUP BY o.customerNumber
)

SELECT AVG(mc.revenue) AS lyf_tym_val
  FROM money_in_by_customer_table mc;

```
![Alt text](image/ltv.PNG)


## Conclusion
The conclusion conveys the three aims of this project:
1. *Which products should we order more of or less of?*

   Analysing the query results of comparing low stock with product performance we can see that, 6 out 10 cars belong to 'Classic Cars' product line. They sell frequently with high product performance. As such we should re-stock these frequently. 

2. *How should we tailor marketing and communication strategies to customer behaviors?*
  
     Analysing the query results of top VIP customers and top least-engaged customers in terms of revenu or profit generation,
              we need to offer loyalty rewards and priority services for our top VIP customers to retain them.
			  Also for top least-engaged customers we need to solicit feedback to better understand their preferences, 
			  expected pricing, discount and offers to increase our sales.
 
3. *How much can we spend on acquiring new customers?*
  
     The average customer liftime value of our store is $ 39,040. This means for every new customer we make profit of 39,040 dollars. 
	          We can use this to predict how much we can spend on new customer acquisition, 
			  at the same time maintain or increase our profit levels.
	          
 <div align="right">[ <a href="#table-of-contents">↑ Back to top ↑</a> ]</div>
                        
