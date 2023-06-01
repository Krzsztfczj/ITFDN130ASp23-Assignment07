# Module07 Website
---
[Google Homepage](https://www.google.com "Google's Homepage")
[GitHub Webpage Code CheatSheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

##

Name: Kris Czaja    
Date: 05/31/2023
Course: IT FDN 130 A Sp 23: 
Foundations Of Databases & SQL Programming


Assignment 07 – Functions

Introduction

Module 07 continues with the subject of functions. Course is full of new information, such as aggregate functions: Max, Min, Avg, Sum, and Count; Group by and Having; Isnull and GetDate.
We learn how to select and operate functions, using Cast, Convert, String, Convert,  Concat, IIF and Case, to name a few.
In the later parts of the module we learn how to use new functions to create reports - part which I found very useful and practical and about user defined functions, or UDFs.
Additionally we learned more about the use of Github.

Explain when you would use a SQL UDF

UDFs are user created functions, custom to our needs and allow us to achieve unique commands. If functions existing in the system are not sufficient to perform desired action, we can define a custom function. They are similar to previously explored views and are named commands. I find them useful when nested in a larger, more complex, select statement.

Explain are the differences between Scalar, Inline, and Multi-Statement Functions

Scalar functions return a single value as an expression. Inline, or simple table-valued functions return a table - single set of rows. Multi-Statement functions, in contrast, also returns a table, but after some processing: can have user defined structure and contain multiple statements. These types of functions have slightly different syntax (begin, end, return).

Summary

Module 07 is the most condensed so far and, although functions were introduced before, introduces a lot of new material. Knowing and mastering those concepts allows for creation of much more advanced queries than before.

###

--*************************************************************************--
-- Title: Assignment07
-- Author: KrisCzaja
-- Desc: This file demonstrates how to use Functions
-- Change Log: When,Who,What
-- 2023-05-29,KrisCzaja,Created File
--**************************************************************************--
Begin Try
	Use Master;
	If Exists(Select Name From SysDatabases Where Name = 'Assignment07DB_KrisCzaja')
	 Begin 
	  Alter Database [Assignment07DB_KrisCzaja] set Single_user With Rollback Immediate;
	  Drop Database Assignment07DB_KrisCzaja;
	 End
	Create Database Assignment07DB_KrisCzaja;
End Try
Begin Catch
	Print Error_Number();
End Catch
go
Use Assignment07DB_KrisCzaja;

-- Create Tables (Module 01)-- 
Create Table Categories
([CategoryID] [int] IDENTITY(1,1) NOT NULL 
,[CategoryName] [nvarchar](100) NOT NULL
);
go

Create Table Products
([ProductID] [int] IDENTITY(1,1) NOT NULL 
,[ProductName] [nvarchar](100) NOT NULL 
,[CategoryID] [int] NULL  
,[UnitPrice] [money] NOT NULL
);
go

Create Table Employees -- New Table
([EmployeeID] [int] IDENTITY(1,1) NOT NULL 
,[EmployeeFirstName] [nvarchar](100) NOT NULL
,[EmployeeLastName] [nvarchar](100) NOT NULL 
,[ManagerID] [int] NULL  
);
go

Create Table Inventories
([InventoryID] [int] IDENTITY(1,1) NOT NULL
,[InventoryDate] [Date] NOT NULL
,[EmployeeID] [int] NOT NULL
,[ProductID] [int] NOT NULL
,[ReorderLevel] int NOT NULL -- New Column 
,[Count] [int] NOT NULL
);
go

-- Add Constraints (Module 02) -- 
Begin  -- Categories
	Alter Table Categories 
	 Add Constraint pkCategories 
	  Primary Key (CategoryId);

	Alter Table Categories 
	 Add Constraint ukCategories 
	  Unique (CategoryName);
End
go 

Begin -- Products
	Alter Table Products 
	 Add Constraint pkProducts 
	  Primary Key (ProductId);

	Alter Table Products 
	 Add Constraint ukProducts 
	  Unique (ProductName);

	Alter Table Products 
	 Add Constraint fkProductsToCategories 
	  Foreign Key (CategoryId) References Categories(CategoryId);

	Alter Table Products 
	 Add Constraint ckProductUnitPriceZeroOrHigher 
	  Check (UnitPrice >= 0);
End
go

Begin -- Employees
	Alter Table Employees
	 Add Constraint pkEmployees 
	  Primary Key (EmployeeId);

	Alter Table Employees 
	 Add Constraint fkEmployeesToEmployeesManager 
	  Foreign Key (ManagerId) References Employees(EmployeeId);
End
go

Begin -- Inventories
	Alter Table Inventories 
	 Add Constraint pkInventories 
	  Primary Key (InventoryId);

	Alter Table Inventories
	 Add Constraint dfInventoryDate
	  Default GetDate() For InventoryDate;

	Alter Table Inventories
	 Add Constraint fkInventoriesToProducts
	  Foreign Key (ProductId) References Products(ProductId);

	Alter Table Inventories 
	 Add Constraint ckInventoryCountZeroOrHigher 
	  Check ([Count] >= 0);

	Alter Table Inventories
	 Add Constraint fkInventoriesToEmployees
	  Foreign Key (EmployeeId) References Employees(EmployeeId);
End 
go

-- Adding Data (Module 04) -- 
Insert Into Categories 
(CategoryName)
Select CategoryName 
 From Northwind.dbo.Categories
 Order By CategoryID;
go

Insert Into Products
(ProductName, CategoryID, UnitPrice)
Select ProductName,CategoryID, UnitPrice 
 From Northwind.dbo.Products
  Order By ProductID;
go

Insert Into Employees
(EmployeeFirstName, EmployeeLastName, ManagerID)
Select E.FirstName, E.LastName, IsNull(E.ReportsTo, E.EmployeeID) 
 From Northwind.dbo.Employees as E
  Order By E.EmployeeID;
go

Insert Into Inventories
(InventoryDate, EmployeeID, ProductID, [Count], [ReorderLevel]) -- New column added this week
Select '20170101' as InventoryDate, 5 as EmployeeID, ProductID, UnitsInStock, ReorderLevel
From Northwind.dbo.Products
UNIOn
Select '20170201' as InventoryDate, 7 as EmployeeID, ProductID, UnitsInStock + 10, ReorderLevel -- Using this is to create a made up value
From Northwind.dbo.Products
UNIOn
Select '20170301' as InventoryDate, 9 as EmployeeID, ProductID, abs(UnitsInStock - 10), ReorderLevel -- Using this is to create a made up value
From Northwind.dbo.Products
Order By 1, 2
go


-- Adding Views (Module 06) -- 
Create View vCategories With SchemaBinding
 AS
  Select CategoryID, CategoryName From dbo.Categories;
go
Create View vProducts With SchemaBinding
 AS
  Select ProductID, ProductName, CategoryID, UnitPrice From dbo.Products;
go
Create View vEmployees With SchemaBinding
 AS
  Select EmployeeID, EmployeeFirstName, EmployeeLastName, ManagerID From dbo.Employees;
go
Create View vInventories With SchemaBinding 
 AS
  Select InventoryID, InventoryDate, EmployeeID, ProductID, ReorderLevel, [Count] From dbo.Inventories;
go

-- Show the Current data in the Categories, Products, and Inventories Tables
Select * From vCategories;
go
Select * From vProducts;
go
Select * From vEmployees;
go
Select * From vInventories;
go

/********************************* Questions and Answers *********************************/
Print
'NOTES------------------------------------------------------------------------------------ 
 1) You must use the BASIC views for each table.
 2) Remember that Inventory Counts are Randomly Generated. So, your counts may not match mine
 3) To make sure the Dates are sorted correctly, you can use Functions in the Order By clause!
------------------------------------------------------------------------------------------'
-- Question 1 (5% of pts):
-- Show a list of Product names and the price of each product.
-- Use a function to format the price as US dollars.
-- Order the result by the product name.
go


Select
	ProductName, Format (UnitPrice, 'C', 'en-US') as 'UnitPrice'

	From vProducts

Order by ProductName

go
--select * from vProductPrice


-- Question 2 (10% of pts): 
-- Show a list of Category and Product names, and the price of each product.
-- Use a function to format the price as US dollars.
-- Order the result by the Category and Product.
-- <Put Your Code Here> --


Select
	CategoryName, ProductName,  Format (UnitPrice, 'C', 'en-US') as 'UnitPrice' 

From vCategories
Join vProducts
on vCategories.CategoryID = vProducts.CategoryID

Order by CategoryName, ProductName
go

--Select * from vProductsByCategories


-- Question 3 (10% of pts): 
-- Use functions to show a list of Product names, each Inventory Date, and the Inventory Count.
-- Format the date like 'January, 2017'.
-- Order the results by the Product and Date.

-- <Put Your Code Here> --

 Select
	ProductName, 
	Datename(mm, InventoryDate) + ', ' + Ltrim(str(Year(InventoryDate)))  as 'InventoryDate', 
	Count

 from vProducts
 Join vInventories
 on vProducts.ProductID = vInventories.ProductID

Order by ProductName, InventoryDate
go


-- Question 4 (10% of pts): 
-- CREATE A VIEW called vProductInventories. 
-- Shows a list of Product names, each Inventory Date, and the Inventory Count. 
-- Format the date like 'January, 2017'.
-- Order the results by the Product and Date.

-- <Put Your Code Here> --

Create View vProductInventories
As
	Select Top 100000000
		ProductName, 
		Datename(mm, InventoryDate) + ', ' + Ltrim(str(Year(InventoryDate)))  as 'InventoryDate', 
		 Count

	from vProducts
	Join vInventories
	on vProducts.ProductID = vInventories.ProductID

Order by ProductName, InventoryDate

go

select * from vProductInventories

-- Check that it works: Select * From vProductInventories;
go

-- Question 5 (10% of pts): 
-- CREATE A VIEW called vCategoryInventories. 
-- Shows a list of Category names, Inventory Dates, and a TOTAL Inventory Count BY CATEGORY
-- Format the date like 'January, 2017'.
-- Order the results by the Product and Date.

-- <Put Your Code Here> --

Create View vCategoryInventories
As 
	Select Distinct Top 10000000
		CategoryName,
		Datename(mm, InventoryDate) + ', ' + Ltrim(str(Year(InventoryDate)))  as 'InventoryDate',
		Sum(Count) as 'Count'

	from vCategories
	Join vProducts
	on vCategories.CategoryID = vProducts.CategoryID
	Join vInventories
	on vProducts.ProductID = vInventories.ProductID

Group by CategoryName, InventoryDate
Order by CategoryName, InventoryDate
go

Select * From vCategoryInventories
-- Check that it works: Select * From vCategoryInventories;

go
-- Question 6 (10% of pts): 
-- CREATE ANOTHER VIEW called vProductInventoriesWithPreviouMonthCounts. 
-- Show a list of Product names, Inventory Dates, Inventory Count, AND the Previous Month Count.
-- Use functions to set any January NULL counts to zero. 
-- Order the results by the Product and Date. 
-- This new view must use your vProductInventories view.

-- <Put Your Code Here> --

Create View vProductInventoriesWithPreviouMonthCounts
As
	Select Top 100000000
		ProductName, 
		Datename(mm, InventoryDate) + ', ' + Ltrim(str(Year(InventoryDate)))  as 'InventoryDate', 
		Count,
		[PreviousMonthCount] = IIF(Month(InventoryDate) = 1, 0, Lag(Count) Over(Order By Month(InventoryDate)))

	from vProducts
	Join vInventories
	on vProducts.ProductID = vInventories.ProductID

Order by ProductName, InventoryDate
go

-- Check that it works: Select * From vProductInventoriesWithPreviousMonthCounts;
go

-- Question 7 (15% of pts): 
-- CREATE a VIEW called vProductInventoriesWithPreviousMonthCountsWithKPIs.
-- Show columns for the Product names, Inventory Dates, Inventory Count, Previous Month Count. 
-- The Previous Month Count is a KPI. The result can show only KPIs with a value of either 1, 0, or -1. 
-- Display months with increased counts as 1, same counts as 0, and decreased counts as -1. 
-- Varify that the results are ordered by the Product and Date.

-- <Put Your Code Here> --

Create view vProductInventoriesWithPreviousMonthCountsWithKPIs
As
	Select Top 10000000
		ProductName, InventoryDate, Count, PreviousMonthCount,
		Case
			When Count > PreviousMonthCount
				Then 1
			When Count = PreviousMonthCount
				Then 0
			When Count < PreviousMonthCount
				Then -1
			End
			as 'KPI'

	From
		vProductInventoriesWithPreviouMonthCounts
Order by ProductName, InventoryDate
Go

-- Important: This new view must use your vProductInventoriesWithPreviousMonthCounts view!
-- Check that it works: Select * From vProductInventoriesWithPreviousMonthCountsWithKPIs;
go

-- Question 8 (25% of pts): 
-- CREATE a User Defined Function (UDF) called fProductInventoriesWithPreviousMonthCountsWithKPIs.
-- Show columns for the Product names, Inventory Dates, Inventory Count, the Previous Month Count. 
-- The Previous Month Count is a KPI. The result can show only KPIs with a value of either 1, 0, or -1. 
-- Display months with increased counts as 1, same counts as 0, and decreased counts as -1. 
-- The function must use the ProductInventoriesWithPreviousMonthCountsWithKPIs view.
-- Varify that the results are ordered by the Product and Date.

-- <Put Your Code Here> --

Create Function dbo.fProductInventoriesWithPreviousMonthCountsWithKPIs(@KPI int)
Returns Table
As
	Return(
		Select Top 10000
			ProductName,
			InventoryDate,
			Count,
			PreviousMonthCount,
			KPI
			From vProductInventoriesWithPreviousMonthCountsWithKPIs
			Where KPI = @KPI
			Order by ProductName, InventoryDate
			)
go


/* Check that it works:
Select * From fProductInventoriesWithPreviousMonthCountsWithKPIs(1);
Select * From fProductInventoriesWithPreviousMonthCountsWithKPIs(0);
Select * From fProductInventoriesWithPreviousMonthCountsWithKPIs(-1);
*/
go

/***************************************************************************************/
