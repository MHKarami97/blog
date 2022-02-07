---
title: "پیدا کردن وابستگی‌های یک شی در SQL Server"
date: 2022-02-06T12:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - entity
  - referencing
  - dm
---

از ویژگی‌های خوب SQL Server وجود داشتن فانکشن‌ها و آبجکت‌های مختلف برای راحت‌سازی کارها است.  
بطور مثال دو شی زیر که از نوع `Dynamic Management Views` هستند، برای پیدا کردن وابستگی‌ها استفاده می‌شوند.  

توسط این دو کوئری به‌ترتیب می‌توانید اشیائی که از آبجکت ورودی استفاده کرده‌اند و اشیائی که در آبجکت شما از آنها استفاده شده است را بدست بیاورید.  

```sql
SELECT * FROM sys.dm_sql_referencing_entities('[mySchema].myEntity', 'Object')
```

```sql
SELECT * FROM sys.dm_sql_referenced_entities('[mySchema].myEntity', 'Object')
```

جزئیات بیشتر:  

[sys-dm-sql-referenced-entities-transact-sql](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-sql-referenced-entities-transact-sql?view=sql-server-ver15)  

[sys-dm-sql-referencing-entities-transact-sql](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-sql-referencing-entities-transact-sql?view=sql-server-ver15)  

[system-dynamic-management-views](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views?view=sql-server-ver15)  