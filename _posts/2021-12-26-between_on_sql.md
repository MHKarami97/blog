---
title: "دستور BETWEEN در SQL"
date: 2021-12-26T11:07:00-00:00
categories:
  - SQL
tags:
  - between
  - sql
  - sql_server
---

این دستور کوتاه شده `X <= Id AND X >= Id2` است که برای برقرار شدن شرط بین دو مقدار کاربرد دارد.  
بطور مثال در مواردی که می‌خواهید یک گزارش بین دو تاریخ را نمایش دهید.  
نکته مهم در این مورد این است که در هر دو طرف شرط حالت `EQUALS` نیز در خروجی حساب می‌شود.  

```sql
SELECT name 
FROM sys.database_principals
WHERE type = 'R'
AND principal_id BETWEEN 16385 AND 16390;
```

برای تاریخ نیز می‌توانید از حالت زیر که تاریخ وارد شده باید در فرمت `YYYYMMDD` باشد، استفاده کنید

```sql
...
WHERE
    OrderDate BETWEEN '20210115' AND '20210117'
ORDER BY
    OrderDate;
```

همچنین این دستور از NOT بصورت `NOT BETWEEN` نیز پشتیبانی می‌کند.  

[اطلاعات بیشتر](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/between-transact-sql?view=sql-server-ver15)  