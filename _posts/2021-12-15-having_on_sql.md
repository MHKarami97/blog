---
title: "دستور HAVING در SQL"
date: 2021-12-15T08:09:00-00:00
categories:
  - SQL
tags:
  - aggregate
  - having
  - sql_server
---

این دستور شبیه به `WHERE` عمل می‌کند با این تفاوت که دستور `HAVING` می‌تواند همراه با `aggregate functions` استفاده شود.  
این دستور بیشتر مواقع همراه با `GROUP BY` استفاده می‌شود و نتیجه آن را فیلتر می‌کند.  

نمونه استفاده:  

```sql
SELECT OrderKey,
       SUM(Amount) AS TotalSales
FROM MyTable
GROUP BY OrderKey
HAVING SUM(Amount) > 50000
ORDER BY OrderKey;
```
یک مثال دیگر:  

```sql
SELECT
    CustomerId,
    YEAR (OrderDate),
    COUNT (OrderId) OrderCount
FROM
    sales.orders
GROUP BY
    CustomerId,
    YEAR (OrderDate)
HAVING
    COUNT (OrderId) >= 2
ORDER BY
    CustomerId;
```

اگر یک `aggregate function` در بخش SELECT بیاید، حتما باید آن را در بخش HAVING هم بیاوردی. همانند مثال بالا که تابع COUNT آمده است.  

دلیل استفاده از این دستور این است که بطور مثال نمی‌توان بصورت زیر از دستور COUNT استفاده کرد:  

```sql
SELECT
    CustomerId,
    COUNT (OrderId) OrderCount
FROM
    sales.orders
WHERE COUNT(OrderId) > 100    
GROUP BY
    CustomerId
ORDER BY
    CustomerId;
```

یک مثال دیگر می‌تواند بصورت زیر باشد که می‌خواهیم لیست مشتریانی که جمع سفارش آنها بیشتر از 100000 است را پیدا کنیم، برای این کار می‌توان بصورت زیر عمل کرد:  

```sql
SELECT Customer,
       SUM(OrderPrice)
FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice) > 2000
```

اطلاعات بیشتر:  
[having](https://docs.microsoft.com/en-us/sql/t-sql/queries/select-having-transact-sql?view=sql-server-ver15)  

دستور HAVING بر روی فقط برروی دیتا grouped شده اعمال می‌شود اما دستور where بر روی دیتای تکی:  

[Use HAVING and WHERE Clauses in the Same Query](https://docs.microsoft.com/en-us/sql/ssms/visual-db-tools/use-having-and-where-clauses-in-the-same-query-visual-database-tools?view=sql-server-ver15)  