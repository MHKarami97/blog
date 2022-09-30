---
title: "بررسی Constraint در زمان Bulk Insert"
date: 2022-04-26T12:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - net
  - bulk
  - constraint
---

بصورت پیش‌فرض در صورتی که از `Bulk` در دیتابیس استفاده کنید، `Constraint` جدول مورد نظر قبل از عمل Insert پاک شده و پس از اتمام بصورت اتومات ساخته می‌شوند.  
پس باید کاربر فراخوانی کننده دسترسی `Alter Table` را داشته باشد. در غیر این صورت با خطا مواجه می‌شوید.  
راه حل پاک کردن این constraint ها یا دادن دسترسی مورد نظر به کاربر است.  

نمونه متن خطا :  

```sql
Bulk copy failed. User does not have ALTER TABLE permission on table 'dbo.Table'. ALTER TABLE permission is required on the target table of a bulk copy operation if the table has triggers or check constraints, but 'FIRE_TRIGGERS' or 'CHECK_CONSTRAINTS' bulk hints are not specified as options to the bulk copy command.</Message>
  <Source>.
```

[check_constraints](https://docs.microsoft.com/en-us/sql/t-sql/statements/bulk-insert-transact-sql?view=sql-server-ver15#check_constraints)  

[sqlbulkcopy](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlbulkcopy?view=dotnet-plat-ext-6.0)  

[sqlbulkcopyoptions](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlbulkcopyoptions?view=dotnet-plat-ext-6.0)  