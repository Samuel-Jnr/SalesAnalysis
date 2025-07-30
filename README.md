# SalesAnalysis
## Project Overview
Adventure Works is a global manufacturer of bicycles and other cycling related products, their database is organized into various schema that logically group tables of data/records from various department of the company. This project aims to identify trends, inefficiencies, opportunities in the sales data to help the management make informed decisions.

## Data Sources
Data is sourced from the AdventureWorks Database, which Includes extensive schemas covering sales, purchasing, product management, HR, and manufacturing provided by [Microsoft](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver17).

## Tools
SQL – SELECT, WHERE, UNION ALL, DATENAME, CTEs, HAVING

## Insights
1.	Identify the top 20 and bottom 20 selling product based on revenue
2.	Identify Customers who have made repeated purchases in the past 12 months
3.	Overall Sales Volume and Revenue Patterns
4.	Trend of sales by product category
5.	Does discount affect sales?
6.	Determine if delay in shipping vary by season
7.	Sales per region and channel
8.	Evaluate revenue performance by region
9.	Evaluate profit margin and discount per product

## Data Analysis
1. Top 20 selling product based on revenue
   ``` sql
   SELECT 
	      DISTINCT sod.ProductID, 
	      pp.Name,
        pp.ProductNumber,
	      pp.ProductLine,
	      sod.OrderQty,
	      sod.UnitPrice,
	      sod.UnitPrice * (1 - sod.UnitPriceDiscount) * sod.OrderQty  AS TotalRevenue
   FROM Sales.SalesOrderDetail AS sod
   LEFT JOIN Production.Product AS pp ON sod.ProductID=pp.ProductID
   ORDER BY TotalRevenue DESC
   OFFSET 0 ROWS FETCH NEXT 20 ROWS ONLY;
   ```
   The top selling products are the touring-1000 and the mountain-100, mountain-200 and Road-350-W, specifically the yellow, black, blue and silver colours.

   RECOMMENDATION
   
   Sales and production department should carry out user experience research/survey on this  products so, we can optimize the product for better experience which in turn will increase sales of this products.
