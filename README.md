# Analyzing Inventory Data for Mint Classics Company

## Introduction

The Mint Classics Company (MC), a totally made-up retailer specializing in classic model cars and vehicles, faces the challenge of optimizing its inventory across multiple storage facilities. 

As part of its strategic initiative to enhance operational efficiency, MC aims to explore its current inventory data and derive insights that can guide decisions around inventory reorganization or reduction.

This project delves into an exploratory data analysis (EDA) of the MC database, examining various facets of inventory management, sales correlations, and warehouse utilization. 

By leveraging SQL queries on the provided database schema, this analysis seeks to identify patterns, uncover relationships between inventory and sales figures, and recommend potential actions to streamline inventory while ensuring timely service to customers.

**Key takeaway:** I'm a real analyst helping a fake business solve a fake problem to demonstrate my skills to you!

### Project Objectives

1. **Explore Current Inventory**: Investigate the composition and distribution of products across warehouses.
2. **Sales Analysis**: Explore product sales. Try to identify any products with high sales and low inventory.
3. **Identify Inventory Movement**: Determine stagnant inventory items and their impact on warehouse utilization.
4. **Warehouse Efficiency Evaluation**: Analyze warehouse capacities and suggest potential consolidation strategies.

Through this exploration, the project aims to offer actionable data-driven recommendations that can assist Mint Classics in making informed decisions to optimize their inventory management practices (and maybe, just maybe close a warehouse or two).


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

The database itself was uploaded from a database script. The code for that is very long, and frankly not very important to this analysis. Just know I downloaded the script and imported in it in MySQL Workbench to set up this whole database.

Let's get started by better acquainting ourselves with the data:


## Part 1: Exploring Current Inventory
 
**Key takeaway**: The data was collected over a 29 month period from 2003 to 2005. The company has four different warehouses with 110 unique products that are spread over 7 different product lines. 

### Query 1-2: Getting a baseline

My first step was to try to get oriented with the data at a high level. First I need to figure out **when the data is from** and how long of a period this data was collected. To do this I looked at the oldest and newest order date.

![Screenshot 2023-12-01 at 11.30.25 AM.png](attachment:c36cf72d-1328-4b65-bd36-17963c55403f.png)

The results showed that the **first order was from January 6, 2003** and the **last order was on May 3, 2005**. My first pice of advice for this company would be to get newer data!

So that gives us a window of roughly **29 months of data**.

Next, I wanted to get a better idea of what the company sold, the type of products, and the general warehouse situation.

To do this I ran a query that returned the **total number of products, product lines, and warehouses**.

![Query1-Code.png](attachment:60ed1b4e-82d9-4c60-a43d-fc8db9bc139e.png)

Here are the results:

![Query1-Results.png](attachment:52372264-215a-4fa1-b8da-0cb3cf64cdc8.png)

Ok great, analysis complete! This fake company now has all the information (based on 20-year-old data) they could ever need. And on top of that, I've demonstrated my *elite querying abilities*!


### Query 3: Warehouse Inventory

Obviously, I couldn't stop there! The next query drills down on these numbers even further to paint a better picture of the data. Remaining focused on the business objective helps us decide what type of picture to paint.

In this case, the company is looking to reduce inventory and maybe even close a warehouse. So this query looks at the **number of products at each warehouse**, number of **different product lines** at that warehouse, **total products in stock at each warehouse**, and how full each warehouse is.

![Warehouse-Query.png](attachment:f014bada-4e1b-45ff-9c68-c2b3170d758b.png)

Don't look now, we're over here joining tables and creating aliases!

Here are the results:

![Results-Warehouse-Query.png](attachment:ef857433-f0e8-420a-8061-1e400962ac97.png)

The results show warehouse 'East' has by far the most types of products (38) and the most quantity of those items in stock(219,183). Interestingly enough, it only houses one product line.

But for now, with our baseline relatively set, we'll move on to looking at sales and the relationship that has with a product's inventory.

Remember, our goal here is to help this company consolidate and potentially downsize its warehouse operations.


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

![Query3-Code.png](attachment:d5182db3-c79b-4e40-baa7-eb6f31973c76.png)

Here are the results:

![Most-and-Least-Popular-Products.png](attachment:1e59b372-1bf3-4126-b804-30bd6c9c9f3d.png)

To make the results easier to digest, I only included the top-5 most and least popular products in the chart. The **1992 red Ferrari Spider is by far the most popular product**, almost 40% more orders than the next closest product.

By contrast, the **Toyota Supra is by far the least popular product with zero orders**. Paul Walker must be turning over in his grave. 

Vintage cars do make up three of the five most popular products though. So let's look at the most popular product lines and see if that holds true for the entire dataset.

### Query 3: The Most Popular Product Lines

Next, let's look at each product line and how popular it is. This will give us a better idea of what percentage of sales each product line is responsible for.

Before looking at that, let's first verify our total sum or orders for each product. This way when we'll be able to tell if any products don't have a product line attached to them.

![Total-Quanity-Orders-Query.png](attachment:da52ea77-90be-4ad5-a07f-3aa8301b95bc.png)

The results showed we have 105,516 total orders. So when we break sales down across product lines, that should be our total.

![Total-Quanity-Orders-Results.png](attachment:b3bc580a-1154-4ad2-ad51-2de640ea4095.png)

Cool. Cool. Cool. Great attention to detail Eric. Way to demonstrate your commitment to data accuracy! 

![Most-Popular-ProductLine-Query.png](attachment:f9bca8cf-fa1b-4eb5-acc6-586aa9dc3db6.png)


Here are the results displayed as a pie chart (because I'm not afraid to dip my toe into controversial and divisive data visualization techniques).

![Most-Popular-Product-Lines.png](attachment:76b62d32-50b3-4d45-b3e2-9c51c9c21735.png)

So looking at the full data set, the **Classic Cars product line proved to be the most popular**. It's clear the bulk of the sales belong to Classic and Vintage cars, the **two car-based lines account for over 50% of overall sales**.

Sales are nice, but profit is even better. Next, we'll look at which products are the most and least profitable for Mint Classics.


### Query 4-5: Product Profitability

Sales only tell one-half of the story. A company's best-selling item could be generating minimal returns if it isn't that profitable. To check this out we'll look at MC's most and least profitable items in two ways.

We'll look at **profit per product**. How much does the company buy it for and how much do they sell it for. The results will show the expected profitability and the actual profitability per unit. We wanted to make sure the actual profit per unit reflected the amount Mint Classic actually sold the product for and not just the suggested MSRP.

This query will also take that actual profit per product and multiply it by the total number of that product sold. This will reveal the **total profit for each product**.

To view all of this information, I got a little ambitious and squeezed everything into one query then exported the data to make it easier to analyze. It also has the benefit of having a file with all the data for a later date. 

![Profitability-Query.png](attachment:896d4369-dd3b-45fe-95a6-795ce85eb946.png)

There's a lot going in that query. But we can break the results down to make them easier to digest.

Top 10 products by total profit:

![Total-Profit-Products-Results-Most.png](attachment:4250cd44-5e73-4103-9428-8db62b594c6f.png)

Bottom 10 products by total profit*:

![Total-Profit-Products-Results-Least.png](attachment:a6e35164-1766-4d91-9db2-1c9c190addd0.png)


Top 10 products by actual profitability per unit:

![Most-Profitable-Products-Results.png](attachment:7e259914-8cc5-4df6-8917-8ba216448e43.png)

Bottom 10 products by acutal profitability per unit*:

![Least-Profitable-Products-Results.png](attachment:85b1030b-273f-48d0-87fc-10e9013de1d6.png)

*Note*: The Toyota Supra is not included because it did not have any sales.

Here are the results visualized on a scatter plot. We can see a clear trend line, aside from the outlier 1992 Ferrari Spyder.

![Product-Profitability.png](attachment:d93a3eec-ccbb-40da-ab74-b3ed7dded272.png)

This scatter plot isn't perfect, I think it could be made more useful by color-coding each plot to its product line. Still, it does illustrate the correlation between unit profit and total profitability.

For example, **every product but one that has a unit profit over \\$50 returns over $50,000 in total profit** for the company.

Yet another *ground-breaking discovery* from this analysis: products that are more profitable per unit make more money for the company. Who knew??? Mint Classics might be regretting their decision to hire me as an analyst right now.

Let's attempt to remedy that by digging deeper into the sales data.

# Part 3: Examining Warehouse Efficiency

**Key Takeaways**:

- There are several areas for product placement optimization.
-

### Query 6: Product Placement Optimization

One of the main objectives of this analysis to examine Mint Classics' warehouses and if they're being properly utilized. 

An interesting way to do that is to find products that are typically ordered together. This can help the company in two ways.

First, if products are frequently ordered together but are housed in different warehouses, these are easy candidates for redistribution.

Second, if products are frequently ordered together and already in the same warehouse, these are easy candidates for reorganization in the warehouse. I.e. moving the two products closer together to enhance picking efficiency.


This is quite a long query, I'll first display the SELECT portion of the query then follow with the rest.

![Common-Orders-Query-Part1.png](attachment:938dc96b-e6ee-4f25-a5e0-08f5cc4a735d.png)
![Common-Orders-Query-Part2.png](attachment:fbe9321f-2dd8-4b91-a05e-4055c9d1a188.png)

Quite the doozy! Here is a sample of the results:

![Results-Common-Orders-Query.png](attachment:b616597a-f9a9-4499-8263-ef6a31f01cc8.png)

This proved to be a very interesting query as we can see some unsual common product combinations. Also of note was that the most common product combination was not at the same warehouse.

These results are sure to yeild some high-specific recommendations during our analysis.

### Query 7-9: Warehouse Utilization

## Mistake Log

When calculating profitabilty I initally just tried to use MSRP subtracted by the buy price for each item. This was simple enough and fairly accurate. However, after further exploring the orderdetails table I realized that the MSRP did not accurately reflect what each person paid for an item. The price people paid varied by order. This would have proven to be a mistake that dramatically impacts the result we got. 

In fact, it probably would've been interesting to see which items had the biggest discrepency between projected MSRP and average actual price paid. This could be brought back to the supplier to potentially negociate a lower buy price for Mint Classics.


_____

When calculating the 'Actual Profitability per unit' I needed to get the average price paid for a specific product. My original query just got the average of the column that contained the priced paid information. However, I realized this wouldn't take into account the quantity of an item that was ordered. The average had to be accurately weighted by how much of a product was ordered at a certain pice.

Addressing these mistakes sucked because I already did a lot of the legwork in creating the queries. Fixing the formulas for both of these issues was also really challenging.



