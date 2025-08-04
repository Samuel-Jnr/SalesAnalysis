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

Result:


| ProductID |	Name	                | 	ProductNumber	 | ProductLine	  |  OrderQty      |	UnitPrice   |   TotalRevenue |
|:---------:|:-------------------------:|:----------------------:|:--------------:|:--------------:|:--------------:|:--------------:|
| 954	    |	Touring-1000 Yellow, 46	| BK-T79Y-46		 | 	T 	  | 	26	   | 	1192.035    |	27893.619    |
| 772	    |   Mountain-100 Silver, 42	|  BK-M82S-42		 |	M 	  |	14	   |	1971.9942   | 	27055.7602   |
|   969	    |	Touring-1000 Blue, 60	|	BK-T79U-60	 |	T 	  |	21	   |	1311.2385   |	26159.2086   |
|   775	    |	Mountain-100 Black, 38 	|	BK-M82B-38	 |	M 	  |	13	   |	1957.4942   |	24938.4759   |
|   966	    |	Touring-1000 Blue, 46	|	BK-T79U-46	 |	T 	  |	19	   |	1311.2385   |	23667.8554   |
|   773	    |	Mountain-100 Silver, 44	|	BK-M82S-44	 |	M 	  |	12	   |	1971.9942   |	23190.6516   |
|   776	    |	Mountain-100 Black, 42	|	BK-M82B-42	 |	M 	  |	12	   |	1957.4942   |	23020.1316   |
|   775	    |	Mountain-100 Black, 38	|	BK-M82B-38	 |	M 	  |	12	   |	1957.4942   |	23020.1316   |
|  777	    |	Mountain-100 Black, 44	|	BK-M82B-44	 |	M 	  |	12	   |	1957.4942   |	23020.1316   |
|  976	    |	Road-350-W Yellow, 48	|	BK-R79Y-48	 |	R 	  |	30	   |	850.495	    | 	22963.365    |
|  966	    |	Touring-1000 Blue, 46	|	BK-T79U-46	 |	T 	  |	18	   |	1311.2385   |	22422.1788   |
|  969	    |	Touring-1000 Blue, 60	|	BK-T79U-60	 |	T 	  |	18	   |	1311.2385   |	22422.1788   |
|  957	    |	Touring-1000 Yellow, 60	|	BK-T79Y-60	 |	T 	  |	17	   |	1311.2385   |	21176.5022   |
|  775	    |	Mountain-100 Black, 38	|	BK-M82B-38	 |	M 	  |	11	   |	1957.4942   |	21101.7873   |
|  777	    |	Mountain-100 Black, 44	|	BK-M82B-44	 |	M 	  |	11	   |	1957.4942   |	21101.7873   |
|  780	    |	Mountain-200 Silver, 42	|	BK-M68S-42	 |	M 	  |	17	   |	1275.9945   |	20607.3116   |
|  773	    |	Mountain-100 Silver, 44	|	BK-M82S-44	 |	M 	  |	10	   |	2039.994    |	20399.94     |
|  771	    |	Mountain-100 Silver, 38	|	BK-M82S-38	 |	M 	  |	10	   |	2039.994    |	20399.94     |
|  774	    |	Mountain-100 Silver, 48	|	BK-M82S-48	 |	M 	  |	10	   |	2039.994    |	20399.94     |
|  782	    |	Mountain-200 Black, 38	|	BK-M68B-38	 |	M 	  |	17	   |	1262.2445   |	20385.2491   |
    
The top selling products are the touring-1000 and the mountain-100, mountain-200 and Road-350-W, specifically the yellow, black, blue and silver colours.

   RECOMMENDATION
   
   Sales and production department should carry out user experience research/survey on this  products so, we can optimize the product for better experience which in turn will increase sales of this products.

3. Customers who have made more than two purchases
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

Result (section of the query result)

|     CustomerID	|	CustomerName	|	TotalOrders	|	FirstPurchase		|	LastPurchase		|
|:---------------------:|:---------------------:|:---------------------:|:-----------------------------:|:-----------------------------:|
|	11091		|   Jennifer  Taylor	|	28		|	2014-06-10 00:00:00.000	|	2013-07-01 00:00:00.000	|
|	11176		| Morgan C Miller	|	28		|	2014-06-29 00:00:00.000	|	2013-07-05 00:00:00.000	|
|	11331		|Grace J Lee		|	27		|	2014-06-26 00:00:00.000	|	2013-07-10 00:00:00.000	|
|	11330		|Grace R Lewis		|27			|2014-06-24 00:00:00.000	|	2013-07-15 00:00:00.000|
|11223			|Isabella B Moore	|27			|2014-06-08 00:00:00.000	|	2013-07-01 00:00:00.000|
|	11711		|Brianna  Jones		|27			|2014-06-28 00:00:00.000	|	2013-07-10 00:00:00.000|
|	11300		|Marshall M Shen	|27			|2014-06-02 00:00:00.000	|2013-07-03 00:00:00.000|
|11185	|Morgan P Jackson	|27	|2014-06-28 00:00:00.000	|2013-07-24 00:00:00.000|
|11262	|Natalie C Wilson	|27	|2014-06-24 00:00:00.000	|2013-07-30 00:00:00.000|
|11200	|Morgan  Lewis		|27	|2014-06-22 00:00:00.000	|2013-07-10 00:00:00.000|
|11276	|Natalie M Martin	|27	|2014-06-24 00:00:00.000	|2013-07-01 00:00:00.000|
|11287	|Ruben J Carlson	|27	|2014-06-30 00:00:00.000	|2013-07-01 00:00:00.000|
|11277	|Ruben B Vazquez	|27	|2014-06-18 00:00:00.000	|2013-07-05 00:00:00.000|
|11566	|Barbara H Zhu		|25	|2014-06-09 00:00:00.000	|2013-06-30 00:00:00.000|
|11215	|Isabella L Brown	|17	|2014-05-17 00:00:00.000	|2013-07-18 00:00:00.000|
|11212	|Marshall K Cai		|17	|2014-06-19 00:00:00.000	|2013-07-02 00:00:00.000|
|11078	|Joe  Sanz		|17	|2014-06-30 00:00:00.000	|2013-07-18 00:00:00.000|


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
