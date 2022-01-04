---
title: "دانستن کوئری‌های زمان‌بر در SQL Server"
date: 2022-04-22T11:07:00-00:00
categories:
  - SQL
tags:
  - long
  - log
  - sql_server
---

بهینه‌سازی دیتابیس و کوئری‌ها یکی از موارد مهم در دیتابیس است تا بتوان از منابع موجود بصورت بهینه استفاده کرد.  
یکی از روش‌ها برای دانستن کوئری‌های زمان‌بر برای بهینه‌سازی، اجرا کردن کوئری زیر در دیتابیس است که اطلاعات کوئری‌های در حال انجام را نشان می‌دهد.  
توسط این کوئری شما می‌توانید کوئری‌هایی که زمان زیادی برای اجرا می‌برند را شناسایی کنید.  

```sql
SELECT DISTINCT TOP 10 t.TEXT                                                                    QueryName,
                       s.execution_count                                                      AS ExecutionCount,
                       s.max_elapsed_time                                                     AS MaxElapsedTime,
                       ISNULL(s.total_elapsed_time / s.execution_count, 0)                    AS AvgElapsedTime,
                       s.creation_time                                                        AS LogCreatedOn,
                       ISNULL(s.execution_count / DATEDIFF(S, s.creation_time, GETDATE()), 0) AS FrequencyPerSec
FROM sys.dm_exec_query_stats s
         CROSS APPLY sys.dm_exec_sql_text(s.sql_handle) t
ORDER BY s.max_elapsed_time DESC
```

روش دیگر استفاده از `Active Monitor` در خود SQL Server Management Studio است.  
برای دسترسی به آن کافی است بر روی دیتابیس خود راست کلیک کنید و آن را انتخاب کند.  

<p align="center" >
  <img src="/assets/img/activeMonitor.jpg" alt="mhkarami97" width="600" />
</p>

ابزار دیگر برای مانیتور، `Query Store` است که تاریخچه کوئری‌ها را نگهداری می‌کند.  
برای فعال سازی این ابزار کافی است بر روی دیتابیس خود راست کلیک کنید و بر روی Properties کلیک کنید و سپس به بخش Query Store بروید.  
سپس در این قسمت مقدار Operation Mode (Requested) را بر روی آیتمی که می‌خواهید قرار دهید.  

<p align="center" >
  <img src="/assets/img/queryStore.jpg" alt="mhkarami97" width="600" />
</p>

<p align="center" >
  <img src="/assets/img/queryStoreItems.jpg" alt="mhkarami97" width="600" />
</p>