---
title: "دستور WITH در SQL"
categories:
  - SQL
tags:
  - with
  - sql
  - sql_server
---

این دستور در واقع یک result set را از نوع موقت ایجاد می‌کند که به آن `Common Table Expression` یا `CTE` نیز می‌گویند.  
روش کلی استفاده از آن بصورت زیر است:  

```sql
WITH query_name (column_name1, ...) AS
     (SELECT ...)
     
SELECT ...
```

قسمت `query_name` فقط در همان کوئری معنی دارد و در دیتابیس ذخیره نمی‌شود. همچنین بعد از آن حتما باید دستور SELECT باشد و دیگر دستورها مثل INSERT پشتیبانی نمی‌شود.  

به عنوان مثال کوئری زیر را درنظر بگیرید:  

```sql
-- Define the CTE expression name and column list.  
WITH Sales_CTE (SalesPersonID, SalesOrderID, SalesYear)  
AS  
-- Define the CTE query.  
(  
    SELECT  SalesPersonID,
            SalesOrderID,
            YEAR(OrderDate) AS SalesYear  
    FROM Sales.SalesOrderHeader  
    WHERE SalesPersonID IS NOT NULL  
)  

-- Define the outer query referencing the CTE name.  
SELECT  SalesPersonID,
        COUNT(SalesOrderID) AS TotalSales,
        SalesYear  
FROM Sales_CTE  
GROUP BY SalesYear, SalesPersonID  
ORDER BY SalesPersonID, SalesYear; 
```

داخل CTE به هیچ عنوان نمیتوانیم از ORDER BY استفاده کنیم مگر آنکه با TOP یا OFFSET FETCH بیاید.  
تمام فیلدهای داخل CTE باید دارای نام باشند به عبارتی no column name نباشند.  
تمام اسامی فیلدهای داخل CTE باید نام های منحصر به فرد داشته باشند.  

اطلاعات بیشتر:  
[with](https://docs.microsoft.com/en-us/sql/t-sql/queries/with-common-table-expression-transact-sql?view=sql-server-ver15)  