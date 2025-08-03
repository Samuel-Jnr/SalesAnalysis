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

2. Customers who have made more than two purchases
   ``` sql
   WITH AllCustomers AS (
    SELECT 
        BusinessEntityID,
        CONCAT(FirstName, ' ',
               ISNULL(MiddleName, ''), ' ',
               LastName) AS CustomerName
    FROM Person.Person

    UNION ALL

    SELECT 
        BusinessEntityID,
        Name AS CustomerName
    FROM Sales.Store
   )
   SELECT
    soh.CustomerID,
	ac.CustomerName,
	COUNT(soh.SalesOrderID) AS TotalOrders,
    MAX(soh.OrderDate) AS FirstPurchase,
	MIN(soh.OrderDate) AS LastPurchase
   FROM Sales.SalesOrderHeader AS soh
   LEFT JOIN AllCustomers AS ac
    ON soh.CustomerID = ac.BusinessEntityID
   WHERE soh.OrderDate >= DATEADD(YEAR, -1, 2014) AND ac.CustomerName IS NOT NULL
   GROUP BY soh.customerID, ac.CustomerName
   HAVING COUNT(DISTINCT soh.SalesOrderID) > 1
   ORDER BY TotalOrders DESC;
   ```

   RECOMENDATION

   xxx

3. Sales Volume and Revenue Pattern
    ``` sql
    SELECT 
	YEAR(soh.OrderDate) AS Yr,
	MONTH(soh.OrderDate) AS Mth,
	SUM(sod.OrderQty) AS TotalOrder,
	SUM(soh.TotalDue) AS Revenue
    FROM Sales.SalesOrderHeader AS soh
    LEFT JOIN Sales.SalesOrderDetail AS sod 
	ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY YEAR(soh.OrderDate), MONTH(soh.OrderDate)
    ORDER BY Yr DESC, Mth DESC;
    ```

4. Product Sold by Season
   ``` sql
   SELECT
	pc.Name AS category,
	ps.Name AS subcategory,
	DATENAME(QUARTER, soh.OrderDate) AS quarter,
	SUM(sod.OrderQty) AS units_sold,
	SUM(sod.LineTotal) AS salesamount
   FROM Sales.SalesOrderDetail AS sod
   LEFT JOIN Sales.SalesOrderHeader AS soh
   ON sod.SalesOrderID = soh.SalesOrderID
	LEFT JOIN Production.Product AS pp
	ON sod.ProductID = pp.ProductID
		LEFT JOIN Production.ProductSubcategory ps
		ON pp.ProductSubcategoryID = ps.ProductSubcategoryID
			LEFT JOIN Production.ProductCategory AS pc
			ON ps.ProductCategoryID = pc.ProductCategoryID
   GROUP BY pc.Name, DATENAME(QUARTER, soh.OrderDate), pp.Name, ps.Name
   ORDER BY category ASC, quarter ASC;
   ```


5. Does delay in shipping vary with month?
   ``` sql
   SELECT
	MONTH(soh.OrderDate) AS month,
	DATENAME(QUARTER, soh.OrderDate) AS Qtr,
	AVG(DATEDIFF(DAY, soh.OrderDate, soh.ShipDate)) AS avgshipdelay,
	AVG(soh.Freight) AS avgfrieghtcost
   FROM Sales.SalesOrderHeader AS soh
   GROUP BY MONTH(soh.OrderDate), DATENAME(QUARTER, soh.OrderDate)
   ORDER BY Qtr, month;
   ```

6. Sales by Region and Channel
   ``` sql
   SELECT
	st.Name AS territory,
	st.[Group] AS territorygroup,
	DATENAME(QUARTER, soh.OrderDate) AS Qtr,
	SUM(sod.OrderQty) AS unitsold,
	SUM(soh.TotalDue) AS revenue,
	COUNT(CASE WHEN soh.OnlineOrderFlag = 0 THEN 0 END) AS saleperson_channel,
	COUNT(CASE WHEN soh.OnlineOrderFlag = 1 THEN 1 END) AS online_order_channel
   FROM Sales.SalesOrderDetail AS sod
   LEFT JOIN Sales.SalesOrderHeader AS soh
	ON sod.SalesOrderID = soh.SalesOrderID
	LEFT JOIN Sales.SalesTerritory AS st
	ON soh.TerritoryID = st.TerritoryID
   GROUP BY st.Name, st.[Group], DATENAME(QUARTER, soh.OrderDate)
   ORDER BY territory, Qtr, revenue;
   ```

7. Sales Performance amongst regions
   ``` sql
   SELECT
	st.Name AS Region,
	st.[Group] AS Regiongroup,
	DATENAME(QUARTER, soh.OrderDate) AS Qtr,
	SUM(soh.TotalDue) AS revenue
   FROM Sales.SalesOrderHeader AS soh
	LEFT JOIN Sales.SalesTerritory AS st
	ON soh.TerritoryID = st.TerritoryID
   GROUP BY st.Name, st.[Group], DATENAME(QUARTER, soh.OrderDate)
   ORDER BY revenue DESC;
   ```
