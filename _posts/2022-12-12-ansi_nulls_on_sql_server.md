---
title: "ANSI_NULLS در SQL Server"
date: 2022-12-12T00:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - sql_server
---

یکی از پارامترها که شاید بیشتر مواقع به آن دقت نکرد باشید، `ANSI_NULLS` است.  
بیشتر مواقع که یک کوئری را بصورت خودکار توسط ابزارها می‌سازید مقدار آن و همچنین مقدار `QUOTED_IDENTIFIER` برابر با ON می‌شود.  
اگر مقدار آنها را OFF کنید ممکن است با مشکلاتی در اجرا کوئری‌ها مواجه شوید.  

```sql
SET ANSI_NULLS ON

SET QUOTED_IDENTIFIER ON
```

در واقع متغیر گفته شده چگونگی برخورد با `= / <>` را تغییر می‌دهد که بصورت خلاصه در جدول زیر آمده است:  

|Boolean Expression|SET ANSI_NULLS ON|SET ANSI_NULLS OFF|  
|---------------|---------------|------------|  
|NULL = NULL|UNKNOWN|TRUE|  
|1 = NULL|UNKNOWN|FALSE|  
|NULL <> NULL|UNKNOWN|FALSE|  
|1 <> NULL|UNKNOWN|TRUE|  
|NULL > NULL|UNKNOWN|UNKNOWN|  
|1 > NULL|UNKNOWN|UNKNOWN|  
|NULL IS NULL|TRUE|TRUE|  
|1 IS NULL|FALSE|FALSE|  
|NULL IS NOT NULL|FALSE|FALSE|  
|1 IS NOT NULL|TRUE|TRUE|  

مقدار متغیر دوم هم چگونگی برخورد با ` "" ` و ` '' ` را مشخص می‌کند

[set-ansi-nulls](https://learn.microsoft.com/en-us/sql/t-sql/statements/set-ansi-nulls-transact-sql?view=sql-server-ver16)  

[set-quoted-identifier](https://learn.microsoft.com/en-us/sql/t-sql/statements/set-quoted-identifier-transact-sql?view=sql-server-ver16)  