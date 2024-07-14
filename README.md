# Sales Data Analysis Project

In this data analysis project, we will examine the sales data from `AdventureWorks`, an online retailer, to extract valuable insights.

<img src="https://github.com/sssingh/sales-customer-product-analysis-powerbi/blob/main/images/title.png?raw=true" width="1000" height="800" />

## Tools Utilized
- **Microsoft SQL Server**: Building the data source
- **Power BI Desktop**: Creating the dashboard/report
- **Power Query Editor**: Data transformation and modeling
- **Power BI Service**: Web accessibility without Power BI login
- **Interactive Report**: Multi-page, interactive for insights and analysis

## Table of Contents
- [Introduction](#introduction) 
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Solution Approach](#solution-approach)
- [Usage Instructions](#usage-instructions)
- [License](#license)
- [Credits](#credits)
- [Contact](#contact)

## Introduction

`AdventureWorks` is an online store specializing in bicycles and related products like bike parts, protective gear, and apparel. Sales transactions, inventory, financials, and customer information are recorded in a transaction database in real time. At the end of each day, this data is extracted, formatted, and transferred to a data warehouse for analysis.

## Objectives
AdventureWorks has requested an in-depth analysis of sales data for 2016 and 2017. The goal is to provide insights into sales performance, customer behavior, and product trends to help formulate strategies for increasing revenue and profits. The specific requirements are:

| Requirement ID | Stakeholder   | Description                                                                        |
|----------------|---------------|------------------------------------------------------------------------------------|
| AW-DA01-REQ-1  | Head of Sales | High-level overview of internet sales by various dimensions (customers, products, cities, quarters) |
| AW-DA01-REQ-2  | Head of Sales | Track sales performance over time against budget/target                           |
| AW-DA01-REQ-3  | Head of Sales | Dynamic slicing/dicing/filtering by year, month, product attributes                |
| AW-DA01-REQ-4  | Sales Rep     | Detailed overview of sales by customers                                             |
| AW-DA01-REQ-5  | Sales Rep     | Detailed overview of sales by products                                              |
| AW-DA01-REQ-6  | Sales Rep     | Dynamic slicing/dicing/filtering by year, month, product, and customer attributes   |

## Dataset
The analysis uses data from the AdventureWorks data warehouse. Real-time transaction data is not directly accessible.

### AdventureWorks Data Warehouse
The data warehouse schema is displayed below:

<img src="https://github.com/sssingh/sales-customer-product-analysis-powerbi/blob/main/images/DW%20Schema.png?raw=true" width="400" height="600" />

The complete database backup can be downloaded from [here](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=ssms). Refer to the [Usage Instructions](#usage-instructions) for details on restoring the database.

### Budget Data
AdventureWorks sets a monthly sales target. The budget data, available as an XLS file, is used to measure performance against targets. A snapshot of the 2016/2017 budget is shown below:

<img src="https://github.com/sssingh/sales-customer-product-analysis-powerbi/blob/main/images/budget.png?raw=true" width="400" height="600" />

## Solution Approach
  
| Requirement ID    | Solution ID     | Proposed Solution                                                                                                                                                        |
|-------------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| AW-DA01-REQ-1 <br> AW-DA01-REQ-2 <br> AW-DA01-REQ-3 | AW-DA01-SOL-1 | An `Executive Summary` Power BI dashboard will provide a high-level sales overview. Year, month slicers, and required filters will enable dynamic data filtering.       |
| AW-DA01-REQ-4 <br> AW-DA01-REQ-6    | AW-DA01-SOL-2    | A `Customer Analysis` Power BI dashboard will segment sales data by various customer attributes. Year, month slicers, and required filters will be available.             |
| AW-DA01-REQ-5 <br> AW-DA01-REQ-6    | AW-DA01-SOL-3    | A `Product Analysis` Power BI dashboard will segment sales data by various product attributes. Year, month slicers, and required filters will be available.              |

### Exploratory Data Analysis (EDA) and Data Preparation [SQL]
We begin with EDA to understand and check the data sanity. In this project, EDA was performed using SQL. 

#### EDA
We explored the data warehouse to identify necessary dimension and fact tables. The primary tables identified are:

| Table                | Description                                               |
|----------------------|-----------------------------------------------------------|
| `DimDate`            | Contains date-related information                         |
| `DimCustomer`        | Contains customer-related information                     |
| `DimGeography`       | Contains customer geography-related information           |
| `DimProduct`         | Contains product-related information                      |
| `DimProductCategory` | Contains product category-related information             |
| `DimProductSubcategory` | Contains product subcategory-related information        |
| `FactInternetSales`  | Contains sales-related information                        |

#### Data Preparation
Data is imported directly from the database into Power BI. We create database views to encapsulate SQL queries, simplifying data import into Power BI. This approach allows for dynamic data refresh and simplifies Power BI management.

##### 1. View: vw_date 
```sql
-- Dimension: Date - All date-related attributes are encapsulated by this view
DROP VIEW IF EXISTS vw_date;
GO

CREATE VIEW vw_date AS
SELECT
    [DateKey],
    [FullDateAlternateKey] AS [Date],
    [DayNumberOfWeek],
    [EnglishDayNameOfWeek] AS [Day],
    [DayNumberOfMonth] AS [Day Nr],
    [EnglishMonthName] AS [Month],
    [MonthNumberOfYear] AS [Month Nr],
    [CalendarQuarter] AS [Quarter],
    [CalendarYear] AS [Year]
FROM
    [AdventureWorksDW2019].[dbo].[DimDate];
GO
```

##### 2. View: vw_customer 
```sql
-- Dimension: Customer - All customer-related attributes are encapsulated by this view
DROP VIEW IF EXISTS vw_customer;
GO

CREATE VIEW vw_customer AS
SELECT
    [CustomerKey],
    CONCAT([FirstName], ', '[LastName]) AS [Full Name],
    CASE [MaritalStatus] WHEN 'M' THEN 'Married' WHEN 'S' THEN 'Single' END AS [Marital status],
    CASE [Gender] WHEN 'M' THEN 'Male' ELSE 'Female' END AS [Gender],
    [YearlyIncome],
    [TotalChildren],
    [EnglishEducation],
    [EnglishOccupation],
    [HouseOwnerFlag],
    [NumberCarsOwned],
    [DateFirstPurchase],
    GEOG.City AS [City],
    GEOG.EnglishCountryRegionName AS [Country]
FROM
    [AdventureWorksDW2019].[dbo].[DimCustomer] AS CUST
    LEFT JOIN [AdventureWorksDW2019].[dbo].[DimGeography] AS GEOG ON GEOG.GeographyKey = CUST.GeographyKey;
GO
```

##### 3. View: vw_product
```sql
-- Dimension: Product - All product-related attributes are encapsulated by this view 
DROP VIEW IF EXISTS vw_product;
GO
 
CREATE VIEW vw_product AS 
SELECT
    [ProductKey],
    CATG.EnglishProductCategoryName AS [Category],
    SUBC.EnglishProductSubcategoryName AS [Sub Category],
    [EnglishProductName],
    [Color],
    [ListPrice],
    [ProductLine],
    [Class],
    [Style],
    [ModelName],
    [EnglishDescription],
    [StartDate],
    [EndDate],
    [Status]
FROM
    [AdventureWorksDW2019].[dbo].[DimProduct] AS PROD
    LEFT JOIN [AdventureWorksDW2019].[dbo].DimProductSubcategory AS SUBC ON SUBC.ProductSubcategoryKey = PROD.ProductSubcategoryKey
    LEFT JOIN [AdventureWorksDW2019].[dbo].DimProductCategory AS CATG ON CATG.ProductCategoryKey = SUBC.ProductCategoryKey
WHERE
    PROD.FinishedGoodsFlag = 1;
GO  
```

##### 4. View: vw_internet_sales
```sql
-- Fact: FactInternetSales - All internet sales details for the years 2016 & 2017 are encapsulated by this view
DROP VIEW IF EXISTS vw_internet_sales;
GO 

CREATE VIEW vw_internet_sales AS 
SELECT
    [ProductKey],
    [OrderDateKey],
    [DueDateKey],
    [ShipDateKey],
    [CustomerKey],
    [SalesOrderNumber],
    [SalesAmount]
FROM
    [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE
    LEFT([OrderDateKey], 4) IN (2017, 2016);
GO
```
The SQL scripts for these views are available in the `sales-analysis.sql` file in this repository.

**Note:** Creating views is an elegant approach, provided the team has the necessary database permissions. If not, the next best approach is to move the SQL queries to Power BI.

### Data Cleaning and Transformation [Power Query Editor]
1. Import `vw_customer`, `vw_product`, `vw_date`, and `vw_internet_sales` as `Dim_Customer`, `Dim_Product`, `Dim_Date

`, and `Fact_InternetSales`, respectively.
2. Import the `Budget` sheet from the provided Excel file.

### Data Modeling
1. Rename the `Date` column in `vw_date` to `Order Date`.
2. Merge queries:
   - `vw_internet_sales` with `vw_customer` (join on `CustomerKey`) and select appropriate columns to form a single fact table.
   - `vw_internet_sales` with `vw_product` (join on `ProductKey`) and select appropriate columns to form a single fact table.
   - `vw_internet_sales` with `vw_date` (join on `OrderDateKey`) and select appropriate columns to form a single fact table.
3. Join the final fact table with the `Budget` table using the `Month Year` column.
4. Remove unnecessary columns from the final fact table.

### Data Visualization [Power BI Desktop]
1. Create a dashboard with multiple pages, including:
   - Executive Summary
   - Customer Analysis
   - Product Analysis
2. Use slicers and filters to allow dynamic data interaction.
3. Visualize key metrics like sales performance against targets, top-selling products, and customer demographics.

## Usage Instructions
1. Restore the AdventureWorks database from the provided backup file.
2. Create views in the SQL Server database using the provided SQL scripts.
3. Import data into Power BI from the created views.
4. Perform data cleaning and transformation in Power Query Editor.
5. Create the Power BI dashboard and publish it to Power BI Service for web accessibility.

## License
This project is licensed under the MIT License.

## Credits
This project was inspired by the sales data analysis needs of AdventureWorks.

## Contact
[![email](https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:gauravgiri959@gmail.com)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/giri-gaurav/)

---

This comprehensive project demonstrates the use of SQL for data extraction and preparation, Power BI for data visualization, and effective communication of insights derived from sales data analysis.
