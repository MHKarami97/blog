---
title: "مقایسه Scalar Valued و Table Valued در SQL"
date: 2021-12-14T08:09:00-00:00
categories:
  - SQL
tags:
  - scalar_valued
  - table_valued
  - sql_server
---

در دیتابیس فانکشن‌های موجود به 2 صورت Scalar و Table-Value هستند که در این مطلب تفاوت آنها را بیان می‌کنیم.  

## `Scalar Function`: 

که به آن `User Defined Functions` یا `UDFS` نیز می‌گویند، فقط یک خروجی دارد و خروجی آن `result set` نیست.  
پس برای مقداردهی به آن می‌توان از `SET` استفاده کرد.  
همچنین برای فراخوانی آن شبیه به Stored Procedures می‌توان از `EXEC` استفاده کرد.  
ورودی این فانکشن‌ها می‌تواند یک یا بیشتر باشد اما خروجی آنها همیشه یکی است.  
این نوع فانکشن نمی‌تواند دیتا را آپدیت کند و فقط قابلیت دستیابی به دیتا را دارد که البته خواندن دیتا در این نوع فانکشن نیز Good Practice نیست.  
این فانکشن قابلیت فراخوانی در ورودی دیگر فانکشن‌ها را نیز دارد.  

برای ساخت این نوع فانکشن می‌توانید بصورت زیر عمل کنید:  

```sql
CREATE FUNCTION [schema_name.]function_name (parameter_list)
RETURNS data_type AS
BEGIN
    statements
    RETURN value
END
```

بطور مثال فانکشن زیر متغیر ورودی را در 2 ضرب می‌کند و آن را برمی‌گرداند

```sql
CREATE FUNCTION dbo.Test(
    @quantity INT,
)
RETURNS INT
AS 
BEGIN
    RETURN @quantity * 2;
END;
```

فانکشن‌های ساخته شده در آدرس `Programmability > Functions > Scalar-valued Functions` ذخیره می‌شوند.  

روش فراخوانی آنها نیز می‌تواند بصورت زیر باشد:  

```sql
SELECT 
    dbo.Test(10) my;
```

لیست توابع از خود SQL که از این نوع هستند را می‌توانید در لینک زیر مشاهده کنید:  

[Categories of scalar functions](https://docs.microsoft.com/en-us/sql/t-sql/functions/functions?view=sql-server-ver15#categories-of-scalar-functions)  

## `Table-Valued Functions`

که به آن `TVFs` نیز می‌گویند، دارای خروجی از نوع `result sets` است و می‌تواند در FROM, JOIN, CROSS APPLY استفاده شود.  
اما قابلیت استفاده در INSERT, UPDATE, DELETE را ندارد.  


## `User-Defined Aggregates`

که به آن `UDA` نیز می‌گویند، برای استفاده حتما نیاز به `GROUP BY` دارند.  
مانند توابع SUM, MIN, COUNT  
ورودی آن مجموعه‌ای از داده‌ها است و فقط یک خروجی برمی‌گرداند.  
از آنها می‌توان در بخش SELECT و HAVING یک کوئری استفاده کرد.  
خروجی آن از نوع `deterministic` است، به این معنی که به ازای ورودی یکسان، همیشه خروجی یکسان می‌دهد.  

[Deterministic and Nondeterministic Functions](https://docs.microsoft.com/en-us/sql/relational-databases/user-defined-functions/deterministic-and-nondeterministic-functions?view=sql-server-ver15)  

لیست توابع از خود SQL که از این نوع هستند را می‌توانید در لینک زیر مشاهده کنید:  

[Aggregate Functions](https://docs.microsoft.com/en-us/sql/t-sql/functions/aggregate-functions-transact-sql?view=sql-server-ver15)  

## `Analytic functions`

شبیه به حالت `Aggregate functions` است اما خروجی آن به ازای هر GROUP می‌تواند چندین سطر است.  
مانند توابع `LAG` و `PERCENTILE_DISC`

[Analytic Functions](https://docs.microsoft.com/en-us/sql/t-sql/functions/analytic-functions-transact-sql?view=sql-server-ver15)  