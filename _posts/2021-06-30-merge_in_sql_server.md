---
title: "استفاده از دستور Merge در دیتابیس"
categories:
  - SQL
tags:
  - sql
  - merge
  - updte
---

یکی از دستورات مفیدی که در SQL Server وجود داره، دستوری به اسم merge هست که کاربردهای مفید زیادی داره.  
مثلا وقتی میخواید اطلاعات یه جدول رو با جدول دیگه ای سینک کنید و یا اطلاعات جدولی رو با استفاده از ورودی های کاربر آپدیت کنید.  

بصورت کلی این دستور عملیات Insert, Delete, Update رو بصورت همزمان برای شما انجام میده.  
ساختار کلی این دستور بصورت زیر هست:  

```sql
MERGE target_table USING source_table
ON merge_condition
WHEN MATCHED
    THEN update_statement
WHEN NOT MATCHED
    THEN insert_statement
WHEN NOT MATCHED BY SOURCE
    THEN DELETE;
```

روش استفاده هم به این صورت هست که شما دوتا جدول رو بهش معرفی میکنید و بعد مشخص میکنید در سناریوهای مختلف یه کاری بکنه.  
نکته ای که هست اینه که (;) حتما باید در انتها بیاد.  

یه نمونه کامل تر:  

```sql
--Synchronize the target table with refreshed data from source table
MERGE target_table AS TARGET
USING source_table AS SOURCE 
ON (TARGET.Id = SOURCE.Id) 

--When records are matched, update the records if there is any change
WHEN MATCHED AND TARGET.Name <> SOURCE.Name OR TARGET.Code <> SOURCE.Code 
THEN UPDATE SET TARGET.Name = SOURCE.Name, TARGET.Code = SOURCE.Code 

--When no records are matched, insert the incoming records from source table to target table
WHEN NOT MATCHED BY TARGET 
THEN INSERT (Id, Name, Code) VALUES (SOURCE.Id, SOURCE.Name, SOURCE.Code)

--When there is a row that exists in target and same record does not exist in source then delete this record target
WHEN NOT MATCHED BY SOURCE 
THEN DELETE 

--$action specifies a column of type nvarchar(10) in the OUTPUT clause that returns 
--one of three values for each row: 'INSERT', 'UPDATE', 'DELETE' according to the action that was performed on that row
OUTPUT $action, 
DELETED.Id AS TargetId, 
DELETED.Name AS TargetName, 
DELETED.Code AS TargetCode, 
INSERTED.Id AS SourceId, 
INSERTED.Name AS SourceName, 
INSERTED.Code AS SourceCode; 
```
-	در کد بالا اگه سطری پیدا بشه که در هر دو جدول یکی باشن، مقدار جدول Target آپدیت میشه.  
-	اگه آیتمی در جدول Source باشه که در Target نباشه، بهش اضافه میشه.  
-	اگه آیتمی داخل جدول Target باشه که داخل source نباشه از جدول Target حذف میشه.  

در نهایت هم خروجی عملیات نشون داده میشه.  

اگر از این روش داخل SP میخواید استفاده کنید، میتونید پارامترهای وردوی رو داخل Temp Table بریزید و بعد بصورت بالا عمل کنید.

لینک برای مطالعه بیشتر:  
[merge-transact-sql](https://docs.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-ver15)
