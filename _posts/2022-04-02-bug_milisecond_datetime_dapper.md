---
title: "باگ مقدار MilliSecond در DateTime توسط dapper"
categories:
  - SQL
tags:
  - sql
  - dapper
  - datetime
  - time
---

در صورتی که از Dapper برای پاس دادن مقادیر DateTime2 به دیتابیس و یا فراخوانی SP استفاده می‌کنید، حواستان باشد که Dapper بصورت پیش‌فرض مقدار Millisecond را دقیق ارسال نمی‌کند.  

بطور مثال فرض کنید SP شما یک متغیر از نوع `DATETIME2(3)` دریافت می‌کند و شما `DateTime.Now` را به آن ارسال کرده‌اید. همچنین مقدار ارسال شده `2021-10-23 12:45:37.320` است.  
در این صورت مقدار ارسال شده فرق دارد و می‌تواند مقدار تقریبی 2 میلی‌ثانیه بیشتر یا کمتر باشد.  

راه‌حل موجود استفاده از کد زیر است:  

```c#
// dapper assumes C# DateTime is SQL DateTime, but we want DateTime2
SqlMapper.AddTypeMap(typeof(DateTime), DbType.DateTime2);
```

لینک‌های مفید:  

[github](https://github.com/DapperLib/Dapper/issues/1511)  