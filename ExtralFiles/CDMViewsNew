--CDMCustomer
%sql
CREATE  VIEW db_mgn.silversales.CDMCustomer AS
SELECT
    ROW_NUMBER() OVER (ORDER BY c.CustomerID) AS CustomerKey,
    c.CustomerID,
    c.NameStyle,
    c.Title,
    c.FirstName,
    c.MiddleName,
    c.LastName,
    c.Suffix,
    c.CompanyName,
    c.SalesPerson,
    c.EmailAddress,
    c.Phone

FROM db_mgn.saleslt.vwcustomer AS c;

--CDMDImProduct
--%sql
create  view db_mgn.silversales.CDMProduct as
SELECT
    ROW_NUMBER() OVER (ORDER BY p.ProductID) AS ProductKey,

    -- Product
    p.ProductID,
    p.Name AS ProductName,
    p.ProductNumber,
    p.Color,
    p.StandardCost,
    p.ListPrice,
    p.Size,
    p.Weight,
    -- Product Category
    p.ProductCategoryID,
    pc.Name AS ProductCategoryName,
    pc.ParentProductCategoryID,
    -- Product Model
    p.ProductModelID,
    pm.Name AS ProductModelName,
    pm.CatalogDescription AS ProductModelCatalogDescription,
    -- Product Description
    pd.ProductDescriptionID,
    pd.Description AS ProductDescription,
    pmpd.Culture AS DescriptionCulture,
    -- Product Dates
    p.SellStartDate,
    p.SellEndDate,
    p.DiscontinuedDate
FROM db_mgn.saleslt.vwproduct AS p
LEFT JOIN db_mgn.saleslt.vwproductcategory AS pc
    ON p.ProductCategoryID = pc.ProductCategoryID
LEFT JOIN db_mgn.saleslt.vwproductmodel AS pm
    ON p.ProductModelID = pm.ProductModelID
LEFT JOIN (
    SELECT
        ProductModelID,
        ProductDescriptionID,
        Culture,
        ROW_NUMBER() OVER (
            PARTITION BY ProductModelID
            ORDER BY ProductDescriptionID
        ) AS rn
    FROM db_mgn.saleslt.vwproductmodelproductdescription
    WHERE Culture = 'en'
) AS pmpd
    ON p.ProductModelID = pmpd.ProductModelID
   AND pmpd.rn = 1
LEFT JOIN db_mgn.saleslt.vwproductdescription AS pd
    ON pmpd.ProductDescriptionID = pd.ProductDescriptionID;


--DimAddress
%sql
create view db_mgn.silversales.CDMAddress as
SELECT
    ROW_NUMBER() OVER (ORDER BY a.AddressID) AS AddressKey,
    a.AddressID,
    a.AddressLine1,
    a.AddressLine2,
    a.City,
    a.StateProvince,
    a.CountryRegion,
    a.PostalCode

FROM db_mgn.saleslt.vwaddress AS a;

--DimDate
%sql
create view db_mgn.silversales.CDMDimDate as

WITH DateRange AS (
    SELECT
        CAST(MIN(OrderDate) AS DATE) AS MinDate,
        CAST(MAX(ShipDate) AS DATE) AS MaxDate
    FROM db_mgn.saleslt.vwsalesorderheader
),

DateSeries AS (
    SELECT
        explode(
            sequence(
                MinDate,
                MaxDate,
                interval 1 day
            )
        ) AS DateValue
    FROM DateRange
)

SELECT
    ROW_NUMBER() OVER (ORDER BY DateValue) AS DateKey,

    DateValue AS FullDate,

    YEAR(DateValue) AS CalendarYear,
    MONTH(DateValue) AS CalendarMonth,
    DAY(DateValue) AS CalendarDay,

    QUARTER(DateValue) AS CalendarQuarter,

    DATE_FORMAT(DateValue, 'MMMM') AS MonthName,
    DATE_FORMAT(DateValue, 'EEEE') AS DayName,

    WEEKOFYEAR(DateValue) AS WeekOfYear,
    DAYOFYEAR(DateValue) AS DayOfYear,

    CASE
        WHEN DAYOFWEEK(DateValue) IN (1, 7)
        THEN 0
        ELSE 1
    END AS IsWeekday

FROM DateSeries
ORDER BY DateValue;

--cdmfactsalesorderdetail
%sql

CREATE OR REPLACE TABLE db_mgn.silversales.cdmfactsalesorderdetail
AS

SELECT
    -- Business keys
    sod.SalesOrderDetailID,
    sod.SalesOrderID,

    -- Dimension keys
    dc.CustomerKey,
    dp.ProductKey,

    dod.DateKey AS OrderDateKey,
    ddu.DateKey AS DueDateKey,
    dsh.DateKey AS ShipDateKey,

    da_ship.AddressKey AS ShipToAddressKey,
    da_bill.AddressKey AS BillToAddressKey,

    -- Measures
    sod.OrderQty,
    sod.UnitPrice,
    sod.UnitPriceDiscount,
    sod.LineTotal

FROM db_mgn.saleslt.vwsalesorderdetail AS sod

INNER JOIN db_mgn.saleslt.vwsalesorderheader AS soh
    ON sod.SalesOrderID = soh.SalesOrderID

LEFT JOIN db_mgn.silversales.cdmcustomer AS dc
    ON soh.CustomerID = dc.CustomerID

LEFT JOIN db_mgn.silversales.cdmproduct AS dp
    ON sod.ProductID = dp.ProductID

LEFT JOIN db_mgn.silversales.cdmdimdate AS dod
    ON CAST(soh.OrderDate AS DATE) = dod.FullDate

LEFT JOIN db_mgn.silversales.cdmdimdate AS ddu
    ON CAST(soh.DueDate AS DATE) = ddu.FullDate

LEFT JOIN db_mgn.silversales.cdmdimdate AS dsh
    ON CAST(soh.ShipDate AS DATE) = dsh.FullDate

LEFT JOIN db_mgn.silversales.cdmaddress AS da_ship
    ON soh.ShipToAddressID = da_ship.AddressID

LEFT JOIN db_mgn.silversales.cdmaddress AS da_bill
    ON soh.BillToAddressID = da_bill.AddressID;
