# Analyzing Inventory Data for Mint Classics Company

![Yellow-Model-Car](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/c0893ddc-6f11-4493-801e-353cf4dc8344)

## Table of Contents
- [Project Objectives](#project-objectives)
- [Data Description](#data-description)
- [Part 1: Exploring the Data](#part-1-exploring-the-data)
- [Part 2: Inventory and Sales](#part-2-inventory-and-sales)
- [Part 3: Examining Warehouse Efficiency](#part-3-examining-warehouse-efficiency)
- [Analysis and Recommendations](#analysis-and-recommendations)
- [Mistake Log](#mistake-log)
- [Just Show Me the Queries](#sql-query-log)

## Introduction

The Mint Classics Company (MC) is a completely made-up company that sells model cars. The company faces the challenge of optimizing its inventory across multiple storage facilities. 

As part of its strategic initiative to enhance operational efficiency, MC aims to explore its current inventory data and derive insights that can guide decisions around inventory reorganization or reduction.

This project delves into an exploratory data analysis (EDA) of the MC database, examining various facets of inventory management, sales correlations, and warehouse utilization. 

By leveraging SQL queries on the provided database schema, this analysis seeks to identify patterns, uncover relationships between inventory and sales figures, and recommend potential actions to streamline inventory while ensuring timely service to customers.

**Key Takeaway:** I'm a real analyst helping a fake business solve a fake problem to demonstrate my data analysis skills to you!

---

### Project Objectives

1. **Explore the Data**: Demonstrate an understanding of the data we have available and better understand the Mint Classics company.
2. **Sales Analysis**: Explore product sales. Try to identify any products with high sales and low inventory.
3. **Warehouse Efficiency Evaluation**: Analyze warehouse capacities and suggest potential consolidation strategies.

Through this exploration, the project aims to offer actionable data-driven recommendations that can assist Mint Classics in making informed decisions to optimize their inventory management practices. Who knows, maybe we'll even close a warehouse or two! No promises though.

---

# Data Description

## Database Schema

The project utilizes a MySQL database with the following schema:

### Tables

1. **customers**: Contains information about customers.
   - Primary Key: `customerNumber`
   
2. **employees**: Stores details of company employees.
   - Primary Key: `employeeNumber`
   
3. **offices**: Holds data related to company offices and locations.
   - Primary Key: `officeCode`
   
4. **orders**: Includes order details such as order number, date, and status.
   - Primary Key: `orderNumber`
   - Foreign Key: `customerNumber` (references `customers` table)
   
5. **orderdetails**: Stores specific information about each order line item.
   - Composite Primary Key: (`orderNumber`, `productCode`)
   - Foreign Key: `orderNumber` (references `orders` table)
   - Foreign Key: `productCode` (references `products` table)
   
6. **payments**: Contains payment information from customers.
   - Composite Primary Key: (`customerNumber`, `checkNumber`)
   - Foreign Key: `customerNumber` (references `customers` table)
   
7. **productlines**: Stores different product lines and descriptions.
   - Primary Key: `productLine`
   
8. **products**: Contains information about various products available.
   - Primary Key: `productCode`
   - Foreign Key: `productLine` (references `productlines` table)
   - Foreign Key: `warehouseCode` (references `warehouses` table)
   
9. **warehouses**: Holds data about different storage facilities.
   - Primary Key: `warehouseCode`

## Database Extended Entity-Relationship (ERR)

![Database](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/10d15756-2c9f-4591-83c7-d31cdd7d77cc)

The database itself was uploaded from a database script. The code for that is very long. Just know I downloaded the script and imported it into MySQL Workbench to set up this whole database.

Let's get started acquainting ourselves with the data:

---

## Part 1: Exploring the Data
 
**Key Takeaways**:
- The data was collected over **29 months** from 2003 to 2005. 
- The company has **4 different warehouses with 110 unique products** that are spread over **7 product lines**.
- There are **7 different offices**, housing **23 total employees**.
- The company has shipped to **21 different countries**.

### Query 1-2: Date and Products

My first step was to try to get oriented with the data at a high level. First I need to figure out **when the data is from** and how long of a period this data was collected. To do this I looked at the oldest and newest order date.

```SQL
-- Finding the oldest order date --
SELECT MIN(orderDate) AS OldestOrderDate 		-- Selecting the minimum order date
FROM orders; 						-- From the 'orders' table

-- Finding the newest order date --
SELECT MAX(orderDate) AS NewestOrderDate 		-- Selecting the maximum order date
FROM orders; 						-- From the 'orders' table
```

The results showed that the **first order was from January 6, 2003** and the **last order was on May 3, 2005**. My first pice of advice for this company would be to get newer data!

So that gives us a window of roughly **29 months of data**.

Next, I wanted to get a better idea of what the company sold, the type of products, and the general warehouse situation.

To do this I ran a query that returned the **total number of products, product lines, and warehouses**.

```SQL
-- This query finds the total number of unique products, product lines, and warehouses. --
SELECT 
    COUNT(DISTINCT productCode) AS UniqueProducts, 		-- Uses the COUNT function to find distinct product codes.
    COUNT(DISTINCT productLine) AS UniqueProductLines,		-- Uses the COUNT function to find distinct product lines.
    COUNT(DISTINCT warehouseCode) AS TotalWarehouses 		-- Uses the COUNT function to find distinct warehouse codes.
FROM products;
```

Here are the results:

![Query1-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/e9b8a292-4cba-4db8-94b3-6a17c5310454)


Ok great, analysis complete! This fake company now has all the information (based on 20-year-old data) they could ever need. And on top of that, I've demonstrated my *elite querying abilities*!


### Query 3: Warehouse Inventory

Obviously, I couldn't stop there! The next query drills down on these numbers even further to paint a better picture of the data. Remaining focused on the business objective helps us decide what type of picture to paint.

In this case, the company is looking to reduce inventory and maybe even close a warehouse. So this query looks at the **number of products at each warehouse**, number of **different product lines** at that warehouse, **total products in stock at each warehouse**, and how full each warehouse is.

```SQL
-- This query finds the total number of unique products, product lines, and inventory at each warehouse. --
SELECT
    w.warehouseName AS WarehouseName,                        -- Selects the name of the warehouse
    COUNT(DISTINCT p.productCode) AS UniqueProducts,         -- Counts the unique products in each warehouse
    COUNT(DISTINCT p.productLine) AS UniqueProductLines,     -- Counts the unique product lines in each warehouse
    SUM(p.quantityInStock) AS TotalQuantityInStock,          -- Sums up the total quantity in stock at each warehouse
    w.warehousePctCap AS WarehouseCapacity                   -- Includes the warehouse percentage capacity
FROM
    warehouses w                                            -- Alias 'w' for the warehouses table
LEFT JOIN
    products p ON w.warehouseCode = p.warehouseCode         -- Joins products with warehouses on matching warehouseCode
GROUP BY
    w.warehouseName, w.warehousePctCap                      -- Groups the results by warehouseName and warehousePctCap
ORDER BY
	TotalQuantityInStock DESC;			     -- Orders the results by TotalQuantity
```

Don't look now, we're over here joining tables and creating aliases!

Here are the results:

![Results-Warehouse-Query](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/82054468-657c-44fb-b510-d92df6b39236)

The results show warehouse 'East' has by far the most types of products (38) and the largest quantity of those items in stock (219,183). Interestingly enough, it only houses one product line.

### Query 4: Employees and Offices

Now, we'll look further into specific details about the company itself.

```SQL
-- Query to view employee count across offices --
SELECT 
    offices.city, 						-- Selecting the office city
    COUNT(employees.employeeNumber) AS NumberOfEmployees 	-- Counting the number of employees
FROM 
    offices 							-- Selecting from the offices table
LEFT JOIN 
    employees ON offices.officeCode = employees.officeCode 	-- Joining employees based on officeCode
GROUP BY 
    offices.officeCode, offices.city 				-- Grouping by officeCode and city
ORDER BY 
    NumberOfEmployees DESC; 					-- Ordering the results by NumberOfEmployees in descending order
```

Results:

![Different-Office-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/debbef46-d669-467f-9f63-0275c3fbd550)

I did independent verifications for the distinct count of total employees and offices to make sure nothing was being left out. Everything matched up, so we can clearly see **7 independent offices** (different from warehouses) with a total of **23 employees**.

### Query 5: Orders by Country

MintClassics is a global company. This means getting an idea of what countries we're shipping to and the quantity we're shipping could prove to be useful. 

```SQL
-- Query to view orders across different countries --
SELECT 
    c.country, 						-- Selecting the country column
    COUNT(DISTINCT o.orderNumber) AS totalOrders, 	-- Counting total unique orders per country
    SUM(od.quantityOrdered) AS totalProductsShipped, 	-- Summing total products shipped per country
    ROUND(SUM(od.quantityOrdered) / COUNT(DISTINCT o.orderNumber), 0) AS avgOrderSize -- Calculating and rounding the average order size per country
FROM 
    orders o 						-- Selecting from the orders table
LEFT JOIN 
    orderdetails od ON o.orderNumber = od.orderNumber 	-- Joining orderdetails table using orderNumber
LEFT JOIN 
    customers c ON o.customerNumber = c.customerNumber 	-- Joining customers table using customerNumber
GROUP BY 
    c.country 						-- Grouping the results by country
```

Here is a sample of the results:

![Country-Orders-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/79c911b3-9a0f-49f6-99cd-4a00f852f6c9)

So Mint Classics has sent orders to **21 different countries**. The results show both the number of unique orders to each country and the total number of products included in those orders. 


**Wrap-up**: We now have a better picture of both the data and the structure of Mint Classics as a company. From here we dig further into sales and warehouse operations.

---

## Part 2: Inventory and Sales

**Key Takeaways**:

- **Most popular product**: 1992 red Ferrari Spider
- **Least popular product**: Toyota Supra
- **Most popular product line**: Classic Cars
- **Least popular product line**: Trains
- **Most Profitable Product**: 1992 red Ferrari Spider
- **Least Profitable Product**: Toyota Supra

### Query 1-2: Most and Least Popular Products & Product Lines

Starting things off for this section we'll do some exploration into the most and least popular products. To do this we'll look at total orders across all products.

```SQL
-- Query to find most popular products --
SELECT p.productName, p.productLine, SUM(od.quantityOrdered) AS TotalQuantityOrdered -- Selecting product name, product line, and total quantity ordered
FROM products p
LEFT JOIN orderdetails od ON p.productCode = od.productCode 	-- Joining 'products' and 'orderdetails' tables on product code
GROUP BY p.productCode, p.productName 				-- Grouping results by product code and product name
ORDER BY TotalQuantityOrdered DESC 				-- Ordering the results by total quantity ordered in descending order
LIMIT 10; 							-- Limiting the results to the top 10

-- Query to find least popular products --
SELECT p.productName, p.productLine, SUM(od.quantityOrdered) AS TotalQuantityOrdered -- Selecting product name, product line, and total quantity ordered
FROM products p
LEFT JOIN orderdetails od ON p.productCode = od.productCode 	-- Joining 'products' and 'orderdetails' tables on product code
GROUP BY p.productCode, p.productName 				-- Grouping results by product code and product name
ORDER BY TotalQuantityOrdered 					-- Ordering the results by total quantity ordered in ascending order
LIMIT 10; 							-- Limiting the results to the bottom 10
```

Here are the results:

![Most-and-Least-Popular-Products](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/ebf7043e-0bbd-4980-a6bf-0b0be28a3814)


To make the results easier to digest, I only included the top-5 most and least popular products in the chart seen above. The **1992 red Ferrari Spider is by far the most popular product**, with almost 40% more orders than the next closest product.

By contrast, the **Toyota Supra is by far the least popular product with zero orders**. Paul Walker must be turning over in his grave. 

Vintage cars do make up three of the five most popular products though. So let's look at the most popular product lines and see if that holds true for the entire dataset.

### Query 3-4: The Most Popular Product Lines

Next, let's look at each product line and how popular it is. This will give us a better idea of what percentage of sales each product line is responsible for.

Before looking at that, let's first verify our total sum or orders for each product. This way when we'll be able to tell if any products don't have a product line attached to them.

```SQL
-- This query shows the total number of products ordered. --
SELECT SUM(quantityOrdered) AS TotalOrders 			-- Selecting the total sum of quantity ordered across all products
FROM orderdetails; 						-- Retrieving data from the 'orderdetails' table
```

The results showed we have 105,516 total orders. So when we break sales down across product lines, that should be our total.

![Total-Quanity-Orders-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/7e87860d-475e-4702-9123-a63e7150a85f)

Cool. Cool. Cool. Great attention to detail Eric. Way to demonstrate your commitment to data accuracy! 

```SQL
-- Query to calculate the total orders for each product line --
SELECT pl.productLine, SUM(od.quantityOrdered) AS TotalOrders 	-- Selecting product line and sum of quantity ordered
FROM productlines pl 						-- Referencing the 'productlines' table as 'pl'
JOIN products p ON pl.productLine = p.productLine		-- Joining 'productlines' and 'products' tables on product line
JOIN orderdetails od ON p.productCode = od.productCode 		-- Joining 'products' and 'orderdetails' tables on product code
JOIN orders o ON od.orderNumber = o.orderNumber 		-- Joining 'orders' and 'orderdetails' tables on order number
GROUP BY pl.productLine 					-- Grouping results by product line
ORDER BY TotalOrders DESC; 					-- Sorting results by total orders in descending order
```

Here are the results displayed as a pie chart (because I'm not afraid to dip my toe into controversial and divisive data visualization techniques).

![Most-Popular-Product-Lines](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/4f72a01b-5203-4198-87ef-116035f25586)

So looking at the full data set, the **Classic Cars product line proved to be the most popular**. It's clear the bulk of the sales belong to Classic and Vintage cars, the **two car-based lines account for over 50% of overall sales**.

Sales are nice, but profit is even better. Next, we'll look at which products are the most and least profitable for Mint Classics.


### Query 5: Product Profitability

Sales only tell part of the story. A company's best-selling item could be generating only average returns if it isn't that profitable. To check this out we'll look at MC's most and least profitable items in two ways.

We'll look at **profit per product**. How much does the company buy it for and how much do they sell it for. The results will show the expected profitability and the actual profitability per unit. We wanted to make sure the actual profit per unit reflected the amount Mint Classic actually sold the product for and not just the suggested MSRP. However, the query also includes the *expected profit*, which shows the difference between MSRP and the price Mint Classics paid for the product.

This query will also take that actual profit per product and multiply it by the total number of that product sold. This will reveal the **total profit for each product**.

To view all of this information, I got a little ambitious and squeezed everything into one query then exported the data to make it easier to analyze. It also has the benefit of creating a file with all the data for a later date. 

```SQL
-- Query to calculate the profit per product and total revenue. --
SELECT 
    p.productName,
    p.productLine,
    FORMAT(SUM((od.quantityOrdered * p.buyPrice)), 2) AS TotalPaid, 		-- Total amount paid for products sold
    FORMAT(SUM(od.quantityOrdered * od.priceEach), 2) AS TotalRevenue, 		-- Total revenue generated
    FORMAT(SUM((od.quantityOrdered * od.priceEach) - (od.quantityOrdered * p.buyPrice)), 2) AS TotalProfit, -- Total profit
    FORMAT(p.MSRP - p.buyPrice, 2) AS ExpectedProfitability, 			-- Expected profitability
    FORMAT((SUM(od.quantityOrdered * od.priceEach) / SUM(od.quantityOrdered)) - p.buyPrice, 2) AS ActualProfitability, -- Actual profitability
    FORMAT(((p.MSRP - p.buyPrice) - ((SUM(od.quantityOrdered * od.priceEach) / SUM(od.quantityOrdered)) - p.buyPrice)) / (p.MSRP - p.buyPrice) * 100, 2) AS DifferenceInProfitability 				-- Difference in profitability as a percentage
FROM 
    products p
JOIN 
    orderdetails od ON p.productCode = od.productCode
GROUP BY 
    p.productCode, p.productName
ORDER BY 
    TotalProfit DESC;
```

There's a lot going in that query. But we can break the results down to make them easier to digest.

Top 10 products by total profit:

![Total-Profit-Products-Results-Most](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/aa7c2141-0530-4826-821d-2bc57dccad1b)

Bottom 10 products by total profit*:

![Total-Profit-Products-Results-Least](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/8d9b744e-ba90-4848-90a9-8271953c43d4)

Top 10 products by actual profitability per unit:

![Most-Profitable-Products-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/8b9b78a1-7eee-4d15-aa1c-ba614b610dbb)

Bottom 10 products by acutal profitability per unit*:

![Least-Profitable-Products-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/ce68517f-71d2-4386-a04e-2c0a7c6643e2)

**Note*: The Toyota Supra is not included because it did not have any sales.

Here are the results visualized on a scatter plot. We can see a clear trend line, aside from the outlier 1992 Ferrari Spyder.

![Product-Profitability](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/fcdfd40d-da1e-4f4c-a67e-f4eeafe06926)

This scatter plot isn't perfect, I think it could be made more useful by color-coding each plot to its product line. Still, it does illustrate the correlation between unit profit and total profitability.

For example, **every product but one that has a unit profit over \\$50 returns over $50,000 in total profit** for the company.

Yet another *ground-breaking discovery* from this analysis: products that are more profitable per unit make more money for the company. Who knew??? Mint Classics might be regretting their decision to hire me as an analyst right now.

**Wrap-up**: We looked at sales from a few different perspectives and identified some areas of opportunity for increasing profit. Next, we'll examine how the different warehouses are performing.

---


# Part 3: Examining Warehouse Efficiency

**Key Takeaways**:

- There are several areas for product placement optimization.
- The South warehouse is small but mighty.

### Query 1: Product Placement Optimization

One of the main objectives of this analysis to examine Mint Classics' warehouses and if they're being properly utilized. 

An interesting way to do that is to find products that are typically ordered together. This can help the company in two ways.

First, if products are frequently ordered together but are housed in different warehouses, these are easy candidates for redistribution.

Second, if products are frequently ordered together and already in the same warehouse, these are easy candidates for reorganization in the warehouse. I.e. moving the two products closer together to enhance picking efficiency.


This is quite a long query, so bear  with me.

```SQL
-- Query to find products that are commonly ordered together --

SELECT 
    p1.productName AS Product1,            -- Selecting the name of the first product
    w1.warehouseCode AS Warehouse1,       -- Selecting the warehouse code of the first product
    w1.warehouseName AS Warehouse1Name,   -- Selecting the warehouse name of the first product
    p2.productName AS Product2,            -- Selecting the name of the second product
    w2.warehouseCode AS Warehouse2,       -- Selecting the warehouse code of the second product
    w2.warehouseName AS Warehouse2Name,   -- Selecting the warehouse name of the second product
    COUNT(*) AS CoOccurrenceCount,        -- Counting the co-occurrences of product pairs
    CASE 
        WHEN w1.warehouseCode = w2.warehouseCode THEN 'Yes'  -- Conditional check if the warehouses are the same
        ELSE 'No'
    END AS SameWarehouse                  -- Labeling if the warehouses are the same or different
FROM 
    orderdetails od1                        -- First set of order details
JOIN 
    orderdetails od2 ON od1.orderNumber = od2.orderNumber AND od1.productCode < od2.productCode
    -- Joining order details to itself to find different product pairs in the same order
JOIN 
    products p1 ON od1.productCode = p1.productCode 		-- Joining product details for the first product
JOIN 
    products p2 ON od2.productCode = p2.productCode 		-- Joining product details for the second product
JOIN 
    warehouses w1 ON p1.warehouseCode = w1.warehouseCode 	-- Joining warehouse details for the first product
JOIN 
    warehouses w2 ON p2.warehouseCode = w2.warehouseCode 	-- Joining warehouse details for the second product
GROUP BY 
    p1.productName, w1.warehouseCode, p2.productName, w2.warehouseCode
 								-- Grouping the results by product names and warehouse codes
ORDER BY 
    CoOccurrenceCount DESC;               			 -- Ordering the results by co-occurrence count in descending order
```

Quite the doozy! Here is a sample of the results:

![Results-Common-Orders-Query](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/58f1aeab-f02b-4b76-9e44-27643c5c7d19)

This proved to be a very interesting query, as we can see some unusual common product combinations. Also of note was that the most common product combination was not at the same warehouse.

These results are sure to yield some high-specific recommendations during our analysis.

### Query 2: Warehouse Efficiency

The next query uses inventory turnover rate to paint a better picture of which warehouses are being run the most efficiently.

```SQL
-- Query to analyze warehouse turnover rate --
SELECT    
    w.warehouseName,                            -- Selecting the warehouse name
    SUM(od.quantityOrdered) AS TotalQuantitySold,-- Calculating the total quantity sold
    w.estimatedMaxCapacity,                      -- Selecting the estimated maximum capacity of the warehouse
    AVG(p.quantityInStock) AS AverageQuantityInStock, -- Calculating the average quantity in stock
    (SUM(od.quantityOrdered) / AVG(p.quantityInStock)) AS InventoryTurnoverRate -- Calculating the inventory turnover rate
FROM
    warehouses w
JOIN
    products p ON w.warehouseCode = p.warehouseCode
JOIN
    orderdetails od ON p.productCode = od.productCode
GROUP BY
    w.warehouseCode, w.warehouseName            -- Grouping results by warehouse code and name
ORDER BY
    InventoryTurnoverRate DESC;                  -- Ordering results by inventory turnover rate
```
Here are the results:

![Warehouse-Efficiency-Results](https://github.com/E-Gilley/MintClassicsAnalysis/assets/150806239/2e1b03d3-9f3c-4df8-922e-8bfa5782af8c)

The **South warehouse is the most efficient**, while the **West is the least efficient**.

A glance at the results shows why. The West warehouse has the second largest maximum capacity, but close to the fewest sales. I'm sure the workers at the southern warehouse would chalk that up to the laid-back work ethic of people on the West Coast...

**Wrap-Up**: There are several opportunities for product organization and increasing efficiency. Next, we'll start analyzing everything to deliver some real results.

---

# Analysis and Recommendations

**Recommendation Summary**:

- Death to the Toyota Supra
- Negotiate Lower Buy Prices for High-Volume Low-Profit Items
- Consolidate High-Demand Items in Single Warehouse Locations
- Should you Close a Warehouse?!? (cliffhanger)

### Product Sales/Inventory Recommendations 

Recommendation: Discontinue Toyota Supra.

Rationale: The Supra is the only product with zero sales to speak of. This could be an error in the data or there could be a more complicated reason. Assuming the data is correct and there aren't any unknown reasons for the lack of sales, the Supra should be yanked from shelves. Given there is current stock of over 7,000 of these cars the company could include them in each order as a special promotion.

Recommendation: Increase the stock of items with less than 1,000 units in stock and over 950 orders.

Rationale: Items like the 1960 BSA Gold Star only have 15 units in stock and over 1,000 orders. That means the stock for these items is dangerously low. The average number of products sold is just about 950 so having 1,000 of all reasonably popular items will make sure products are available.

# Reflection

### Data Limitations

This project was limited by the availability of more warehouse-specific data.

Specifically, more data around inventory dynamics would've proved incredibly useful.

The lack of data on inventory stock replenishment cycles or product storage durations within warehouses restricted insights into inventory management efficiency. Analyzing this data could have facilitated inventory optimization, reducing holding costs and ensuring product availability.

### Areas for Further Exploration

Given the significant number of orders from the United States, a deeper analysis aimed at optimizing operations within this market could be immensely beneficial. Several strategies could be explored:

- **Customer Segmentation**: Conducting a detailed analysis of customer segments within the US market based on purchase history, demographics, or product preferences. This could enable targeted marketing campaigns or personalized recommendations, enhancing customer engagement and retention.

- **Predictive Analytics**: Implementing predictive models based on historical order data to forecast demand trends within the US market. This could assist in proactive inventory management and resource allocation.

- **Market Basket Analysis**: Exploring associations between products frequently purchased together by US customers. This analysis could facilitate bundling strategies, cross-selling opportunities, or inventory stocking optimization.

- **Customer Satisfaction and Feedback Analysis**: Leveraging customer feedback and satisfaction data to identify pain points, enhance service quality, and tailor offerings to meet US customers' evolving needs.

# Mistake Log

- While calculating the orders for each country, I first used the total number of unique orders for each country. This wouldn't have been wrong; however, that wouldn't have painted the most accurate picture. This method would've counted each order the same and not taken the quantity of each order into account.

- When calculating profitabilty I initally just tried to use MSRP subtracted by the buy price for each item. This was simple enough and fairly accurate. However, after further exploring the orderdetails table I realized that the MSRP did not accurately reflect what each person paid for an item. The price people paid varied by order. This would have proven to be a mistake that dramatically impacts the result we got. 

In fact, it probably would've been interesting to see which items had the biggest discrepency between projected MSRP and average actual price paid. This could be brought back to the supplier to potentially negociate a lower buy price for Mint Classics.


- When calculating the 'Actual Profitability per unit' I needed to get the average price paid for a specific product. My original query just got the average of the column that contained the priced paid information. However, I realized this wouldn't take into account the quantity of an item that was ordered. The average had to be accurately weighted by how much of a product was ordered at a certain pice.

# SQL Query Log

If you're just here to make sure I know a thing or two about SQL and not interested in my ramblings, I'm going to just include all of the queries I wrote below in one place. I like your style. Anyway, here are all the queries used in this project in order:

```SQL
-- Finding the oldest order date --
SELECT MIN(orderDate) AS OldestOrderDate 		-- Selecting the minimum order date
FROM orders; 						-- From the 'orders' table

-- Finding the newest order date
SELECT MAX(orderDate) AS NewestOrderDate 		-- Selecting the maximum order date
FROM orders; 						-- From the 'orders' table
```
---

```SQL
-- This query finds the total number of unique products, product lines, and warehouses. --
SELECT 
    COUNT(DISTINCT productCode) AS UniqueProducts, 		-- Uses the COUNT function to find distinct product codes.
    COUNT(DISTINCT productLine) AS UniqueProductLines,		-- Uses the COUNT function to find distinct product lines.
    COUNT(DISTINCT warehouseCode) AS TotalWarehouses 		-- Uses the COUNT function to find distinct warehouse codes.
FROM products;
```
---

```SQL
-- This query finds the total number of unique products, product lines, and inventory at each warehouse. --
SELECT
    w.warehouseName AS WarehouseName,                        -- Selects the name of the warehouse
    COUNT(DISTINCT p.productCode) AS UniqueProducts,         -- Counts the unique products in each warehouse
    COUNT(DISTINCT p.productLine) AS UniqueProductLines,     -- Counts the unique product lines in each warehouse
    SUM(p.quantityInStock) AS TotalQuantityInStock,          -- Sums up the total quantity in stock at each warehouse
    w.warehousePctCap AS WarehouseCapacity                   -- Includes the warehouse percentage capacity
FROM
    warehouses w                                            -- Alias 'w' for the warehouses table
LEFT JOIN
    products p ON w.warehouseCode = p.warehouseCode         -- Joins products with warehouses on matching warehouseCode
GROUP BY
    w.warehouseName, w.warehousePctCap                      -- Groups the results by warehouseName and warehousePctCap
ORDER BY
	TotalQuantityInStock DESC;			     -- Orders the results by TotalQuantity
```
---

```SQL
-- Query to view employee count across offices --
SELECT 
    offices.city, 						-- Selecting the office city
    COUNT(employees.employeeNumber) AS NumberOfEmployees 	-- Counting the number of employees
FROM 
    offices 							-- Selecting from the offices table
LEFT JOIN 
    employees ON offices.officeCode = employees.officeCode 	-- Joining employees based on officeCode
GROUP BY 
    offices.officeCode, offices.city 				-- Grouping by officeCode and city
ORDER BY 
    NumberOfEmployees DESC; 					-- Ordering the results by NumberOfEmployees in descending order
```
---

```SQL
-- Query to view orders across different countries --
SELECT 
    c.country, 						-- Selecting the country column
    COUNT(DISTINCT o.orderNumber) AS totalOrders, 	-- Counting total unique orders per country
    SUM(od.quantityOrdered) AS totalProductsShipped, 	-- Summing total products shipped per country
    ROUND(SUM(od.quantityOrdered) / COUNT(DISTINCT o.orderNumber), 0) AS avgOrderSize -- Calculating and rounding the average order size per country
FROM 
    orders o 						-- Selecting from the orders table
LEFT JOIN 
    orderdetails od ON o.orderNumber = od.orderNumber 	-- Joining orderdetails table using orderNumber
LEFT JOIN 
    customers c ON o.customerNumber = c.customerNumber 	-- Joining customers table using customerNumber
GROUP BY 
    c.country 						-- Grouping the results by country
```
---

```SQL
-- Query to find most popular products --
SELECT p.productName, p.productLine, SUM(od.quantityOrdered) AS TotalQuantityOrdered -- Selecting product name, product line, and total quantity ordered
FROM products p
LEFT JOIN orderdetails od ON p.productCode = od.productCode 	-- Joining 'products' and 'orderdetails' tables on product code
GROUP BY p.productCode, p.productName 				-- Grouping results by product code and product name
ORDER BY TotalQuantityOrdered DESC 				-- Ordering the results by total quantity ordered in descending order
LIMIT 10; 							-- Limiting the results to the top 10

-- Query to find least popular products --
SELECT p.productName, p.productLine, SUM(od.quantityOrdered) AS TotalQuantityOrdered -- Selecting product name, product line, and total quantity ordered
FROM products p
LEFT JOIN orderdetails od ON p.productCode = od.productCode 	-- Joining 'products' and 'orderdetails' tables on product code
GROUP BY p.productCode, p.productName 				-- Grouping results by product code and product name
ORDER BY TotalQuantityOrdered 					-- Ordering the results by total quantity ordered in ascending order
LIMIT 10; 							-- Limiting the results to the bottom 10
```
---

```SQL
-- This query shows the total number of products ordered. --
SELECT SUM(quantityOrdered) AS TotalOrders 			-- Selecting the total sum of quantity ordered across all products
FROM orderdetails; 						-- Retrieving data from the 'orderdetails' table
```
---

```SQL
-- Query to calculate the total orders for each product line --
SELECT pl.productLine, SUM(od.quantityOrdered) AS TotalOrders 	-- Selecting product line and sum of quantity ordered
FROM productlines pl 						-- Referencing the 'productlines' table as 'pl'
JOIN products p ON pl.productLine = p.productLine		-- Joining 'productlines' and 'products' tables on product line
JOIN orderdetails od ON p.productCode = od.productCode 		-- Joining 'products' and 'orderdetails' tables on product code
JOIN orders o ON od.orderNumber = o.orderNumber 		-- Joining 'orders' and 'orderdetails' tables on order number
GROUP BY pl.productLine 					-- Grouping results by product line
ORDER BY TotalOrders DESC; 					-- Sorting results by total orders in descending order
```
---

```SQL
-- Query to calculate the profit per product and total revenue. --
SELECT 
    p.productName,
    p.productLine,
    FORMAT(SUM((od.quantityOrdered * p.buyPrice)), 2) AS TotalPaid, 		-- Total amount paid for products sold
    FORMAT(SUM(od.quantityOrdered * od.priceEach), 2) AS TotalRevenue, 		-- Total revenue generated
    FORMAT(SUM((od.quantityOrdered * od.priceEach) - (od.quantityOrdered * p.buyPrice)), 2) AS TotalProfit, -- Total profit
    FORMAT(p.MSRP - p.buyPrice, 2) AS ExpectedProfitability, 			-- Expected profitability
    FORMAT((SUM(od.quantityOrdered * od.priceEach) / SUM(od.quantityOrdered)) - p.buyPrice, 2) AS ActualProfitability, -- Actual profitability
    FORMAT(((p.MSRP - p.buyPrice) - ((SUM(od.quantityOrdered * od.priceEach) / SUM(od.quantityOrdered)) - p.buyPrice)) / (p.MSRP - p.buyPrice) * 100, 2) AS DifferenceInProfitability 				-- Difference in profitability as a percentage
FROM 
    products p
JOIN 
    orderdetails od ON p.productCode = od.productCode
GROUP BY 
    p.productCode, p.productName
ORDER BY 
    TotalProfit DESC;
```
---

```SQL
-- Query to find products that are commonly ordered together --

SELECT 
    p1.productName AS Product1,            -- Selecting the name of the first product
    w1.warehouseCode AS Warehouse1,       -- Selecting the warehouse code of the first product
    w1.warehouseName AS Warehouse1Name,   -- Selecting the warehouse name of the first product
    p2.productName AS Product2,            -- Selecting the name of the second product
    w2.warehouseCode AS Warehouse2,       -- Selecting the warehouse code of the second product
    w2.warehouseName AS Warehouse2Name,   -- Selecting the warehouse name of the second product
    COUNT(*) AS CoOccurrenceCount,        -- Counting the co-occurrences of product pairs
    CASE 
        WHEN w1.warehouseCode = w2.warehouseCode THEN 'Yes'  -- Conditional check if the warehouses are the same
        ELSE 'No'
    END AS SameWarehouse                  -- Labeling if the warehouses are the same or different
FROM 
    orderdetails od1                        -- First set of order details
JOIN 
    orderdetails od2 ON od1.orderNumber = od2.orderNumber AND od1.productCode < od2.productCode
    -- Joining order details to itself to find different product pairs in the same order
JOIN 
    products p1 ON od1.productCode = p1.productCode 		-- Joining product details for the first product
JOIN 
    products p2 ON od2.productCode = p2.productCode 		-- Joining product details for the second product
JOIN 
    warehouses w1 ON p1.warehouseCode = w1.warehouseCode 	-- Joining warehouse details for the first product
JOIN 
    warehouses w2 ON p2.warehouseCode = w2.warehouseCode 	-- Joining warehouse details for the second product
GROUP BY 
    p1.productName, w1.warehouseCode, p2.productName, w2.warehouseCode
 								-- Grouping the results by product names and warehouse codes
ORDER BY 
    CoOccurrenceCount DESC;

---

```SQL
-- Query to analyze warehouse turnover rate --
SELECT    
    w.warehouseName,                            -- Selecting the warehouse name
    SUM(od.quantityOrdered) AS TotalQuantitySold,-- Calculating the total quantity sold
    w.estimatedMaxCapacity,                      -- Selecting the estimated maximum capacity of the warehouse
    AVG(p.quantityInStock) AS AverageQuantityInStock, -- Calculating the average quantity in stock
    (SUM(od.quantityOrdered) / AVG(p.quantityInStock)) AS InventoryTurnoverRate -- Calculating the inventory turnover rate
FROM
    warehouses w
JOIN
    products p ON w.warehouseCode = p.warehouseCode
JOIN
    orderdetails od ON p.productCode = od.productCode
GROUP BY
    w.warehouseCode, w.warehouseName            -- Grouping results by warehouse code and name
ORDER BY
    InventoryTurnoverRate DESC;                  -- Ordering results by inventory turnover rate
```



