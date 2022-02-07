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

بطور مثال فرض کنید می‌خواهید یک دیتابیس بزرگ را پاکسازی کنید و فقط آبجکت‌هایی که در اسکیما شما استفاده شده است را نگهداری کنید، توسط کوئری‌های بالا می‌توانید وابستگی‌ها را بدست بیاورید.  

برای راحت انجام دادن کار بالا و پیدا کردن آبجکت‌هایی که در اسکیما شما استفاده نشده‌اند نیز می‌توانید از کوئری زیر استفاده کنید:  

```sql
SELECT DISTINCT 'mySchema.MyEntity',
                referenced_entity_name + '.' + referenced_schema_name
FROM sys.dm_sql_referenced_entities('[mySchema].MyEntity', 'Object') r
         LEFT JOIN sys.objects o ON o.object_id = r.referenced_id
WHERE o.object_id IS NULL;
```

کد بالا را نیز می‌توانید توسط کوئری زیر تولید کنید و در نهایت می‌توانید آیتم‌های موجود در Temp را پاک کنید.  

```sql
CREATE TABLE #Temp
(
    TargetName    VARCHAR(500),
    ReferenceName VARCHAR(500)
);

SELECT 'Insert Into #Temp
SELECT Distinct ''' + SCHEMA_NAME(o.schema_id) + '.' + o.name +
       ''',  referenced_entity_name + ''.'' + referenced_schema_name from sys.dm_sql_referenced_entities(''[' +
       SCHEMA_NAME(o.schema_id) + '].' + o.name + ''',''Object'') r
LEFT JOIN sys.objects o on o.object_id = r.referenced_id
where o.object_id is null'
FROM sys.objects o;
```

جزئیات بیشتر:  

[sys-dm-sql-referenced-entities-transact-sql](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-sql-referenced-entities-transact-sql?view=sql-server-ver15)  

[sys-dm-sql-referencing-entities-transact-sql](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-sql-referencing-entities-transact-sql?view=sql-server-ver15)  

[system-dynamic-management-views](https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views?view=sql-server-ver15)  

[sys-objects-transact-sql](https://docs.microsoft.com/en-us/sql/relational-databases/system-catalog-views/sys-objects-transact-sql?view=sql-server-ver15)  