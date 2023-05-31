---
title: "آپدیت کردن یک جدول با استفاده از SELECT و JOIN"
categories:
  - SQL
tags:
  - sql
  - update
  - select
  - join
---

فرض کنید 2 جدول دارید که می‌خواهید اطلاعات یکی از آنها را با توجه به جدول دوم آپدیت کنید. یا به زبان دیگر این 2 جدول با یکدیگر کلید خارجی دارند و می‌خواهید فیلدهای یکی از آنها را آپدیت کنید.  
توسط کوئری زیر می‌توانید این کار را انجام بدهید.  

```sql
UPDATE c
SET c.[NationalCode] = cc.[NationalCode],
    c.[CustomerTitle]  = cc.[CustomerTitle]
FROM [bof].Customer AS c
         LEFT JOIN
     [tse].Customer2 AS cc
     ON c.Id = cc.CustomerId
WHERE
    cc.Type = 2
GO
```