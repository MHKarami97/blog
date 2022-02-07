---
title: "بررسی CDC یا Change Data Capture در SQL Server"
date: 2022-02-07T09:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - cdc
  - data_capture
---

یکی از قابلیت‌های کاربردی در SQL Server امکانی با اسم `CDC` است که به شما قابلیت دانستن تغییرات بر روی یک جدول خاص را می‌دهد.  
فرض کنید در برنامه‌ای که بصورت MicroService نوشته شده است شما به دیتا یک سرویس دیگر نیاز دارید و فقط می‌خواهید آن بخش از دیتا که نیاز اصلی شما هست را در دیتابیس سرویس خود نگهداری کنید.  
قابلیت گفته شده این توانایی را به شما می‌دهد که از تغییرات انجام شده بر روی جدول اصلی مطلع شوید و جداول خود را آپدیت کنید.  

برای این کار نیاز است که در ابتدا `SQL Server Agent` را Start کنید.  

<p align="center" >
  <img src="/assets/img/cdc3.jpg" alt="mhkarami97" width="600" />
</p>

سپس برای فعال‌سازی این قابلیت کافی است کد زیر را اجرا کنید.  

```sql
USE [MyDb]
GO

EXEC sys.sp_cdc_enable_db
```

و سپس برای فعال‌سازی این قابلیت در سطح دیتابیس خود، کد زیر را فراخوانی کنید.  

```sql
USE [MyDb]
GO  

EXEC sys.sp_cdc_enable_table  
@source_schema = N'schema_name',  
@source_name   = N'tbl_name',  
@role_name     = NULL,  
@filegroup_name = NULL,  
@supports_net_changes = 0 
```

برای چک کردن موفقیت آمیز بودن کدهای بالا نیز می‌توانید از دستورهای زیر استفاده کنید.  

```sql
USE master
GO

select name,
       is_cdc_enabled
from sys.databases
where name = 'db_name'
```

```sql
USE [MyDb]
GO

select  name,
        type,
        type_desc,
        is_tracked_by_cdc
from sys.tables
where name = 'tbl_name'
```

در صورتی که تمام موارد به‌درستی پیش برود، قسمت‌های زیر به دیتابیس شما اضافه می‌شود:  

<p align="center" >
  <img src="/assets/img/cdc1.jpg" alt="mhkarami97" width="600" />
</p>

<p align="center" >
  <img src="/assets/img/cdc2.jpg" alt="mhkarami97" width="600" />
</p>

جاب‌های اضافه شده به سیستم کارهای زیر را انجام می‌دهند:  

  - Capture job: وظیفه جمع‌آوری اطلاعات را بر عهده دارد.
    - این job بعد از اجرای دستور بلافاصله آغاز به کار می‌کند.
    - پیوسته در حال اجراشدن است.
    - در هر اجرا نهایتاً 1000 تراکنش انجام می‌دهد.
    - بین هر اجرا 5 ثانیه تأخیر وجود دارد.


  - Clean up job: وظیفه حذف رکوردهای قدیمی را بر عهده دارد.
    - هر شب ساعت 2 اجرا می‌شود.
    - به‌صورت پیش‌فرض رکوردهای قدیمی‌تر از 3 روز را پاک می‌کند.
    - با هر دستور delete در حدود 5000 رکورد را حذف می‌کند.

فانکشن‌های اضافه شده به دیتابیس، دارای نام ثابت `fn_cdc_get_all_changes_` هستند که در انتهای آن نام دیتابیس شما می‌آید. بطور مثال `fn_cdc_get_all_changes_dbo_MyTable`.  
برای دریافت لیست تغییرات انجام شده بر روی دیتایس می‌توانید از کوئری زیر استفاده کنید:  

```sql
DECLARE @from_lsn binary(10), @to_lsn binary(10);

SET @from_lsn = sys.fn_cdc_get_min_lsn('dbo_MyTable');
SET @to_lsn   = sys.fn_cdc_get_max_lsn();

SELECT * FROM cdc.fn_cdc_get_all_changes_dbo_MyTable(@from_lsn, @to_lsn, N'all update old');
```

در کد بالا اگر مقدار `all update old` را قرار دهید، در دستور آپدیت مقادیر قبل از تغییر نیز که کد 3 دارند هم نمایش داده می‌شوند و اگر مقدار `all` را قرار دهید، مقادیر گفته شده نشان داده نمی‌شوند.  

[cdc-fn-cdc-get-all-changes-capture](https://docs.microsoft.com/en-us/sql/relational-databases/system-functions/cdc-fn-cdc-get-all-changes-capture-instance-transact-sql?view=sql-server-ver15)  

<p align="center" >
  <img src="/assets/img/cdc6.jpg" alt="mhkarami97" width="600" />
</p>

در خروجی نشان داده شده:  

  - __$start_lsn: نشان‌دهنده شماره Commit است. عملیاتی که در یک تراکنش انجام‌شده باشند، مقادیر این ستون برایشان یکسان ثبت می‌شود (LSN مخفف Log Sequence Number است)
  - __$seqval: شماره عملیات در یک تراکنش را نشان می‌دهد.
  - __$operation: نشان‌دهنده نوع عمل صورت گرفته است:
    - شماره 1 نشان‌دهنده delete
    - شماره 2 نشان‌دهنده insert
    - شماره 3 نشان‌دهنده update (پیش از تغییر مقادیر)
    - و شماره 4 نشان‌دهنده update (پس از تغییر مقادیر) است
  - __$update_mask: یک متغیر از جنس bit mask است که به ازای هر ستونی که مقدار گرفته باشد، 1 می‌گیرد. برای Insert و Update تمام بیت‌های آن 1 ثبت می‌شود ولی برای آپدیت، تنها ستون‌هایی که تغییر کرده‌اند مقدار 1 می‌گیرند.


> حواستان باشد که قابلیت دیگری به اسم `Change Tracking` نیز در SQL Server وجود دارد که با این قابلیت گفته شده تفاوت دارد.  


توضیحات بیشتر:  

[about-change-data-capture](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server?view=sql-server-ver15)  

[enable-and-disable-change-data-capture](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server?view=sql-server-ver15)  

[administer-and-monitor-change-data-capture](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/administer-and-monitor-change-data-capture-sql-server?view=sql-server-ver15)  

[work-with-change-data](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/work-with-change-data-sql-server?view=sql-server-ver15)  

[change-data-capture-and-other-sql-server-features](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/change-data-capture-and-other-sql-server-features?view=sql-server-ver15)  

[cdc-fn-cdc-get-net-changes-capture](https://docs.microsoft.com/en-us/sql/relational-databases/system-functions/cdc-fn-cdc-get-net-changes-capture-instance-transact-sql?view=sql-server-ver15)  