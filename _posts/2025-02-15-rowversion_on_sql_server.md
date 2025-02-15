---
title: "زمان مقداردهی ID , RowVersion در SQL Server"
categories:
  - SQL
tags:
  - sql
  - row_version
  - transaction
---

در صورتی که در سیستم خود بیزینسی شبیه به حالت زیر دارید به نکته‌ای که در ادامه آن توضیح داده می‌شود نیاز است که دقت کنید:  
داشتن یک SP که با یک اینتروال زمانی تمام سطرهایی که در شرط `Id > x` یا `RowVersion > x` را دریافت کند که در آن x آخرین شناسه خوانده شده در زمان قبلی است.  
بطور مثال در زمان 1 شما sp را با x=0 فراخوانی می‌کنید و 10 رکورد به شما برمی‌گرداند.  
در زمان 2 شما دوباره sp را با x=10 فراخوانی می‌کنید.  
در این حالت ممکن است بعضی سطرها را کلا دریافت نکنید که دلیل این مشکل در ادامه بیان می‌شود.  

مقداردهی به Id یا RowVersion در زمان فراخوانی دستور INSERT است نه زمان COMMIT تراکنش. بطور مثال در کد زیر به محض فراخوانی INSERT مقدار Id دریافت می‌شود.  

```sql
BEGIN TRANSACTION T1;

INSERT INTO #TempProducts (Name, Price)
VALUES ('Product T1', 1000);


COMMIT TRANSACTION T1;
```

در صورتی که تعداد تغییرات در جدول دیتابیس شما زیاد است و بصورت همزمان اینزت‌های مختلف در تراکنش‌های مختلف اتفاق می‌افتد، در صورتی که تراکنش زیاد طول بکشد ممکن است رکورد آن در سناریو گفته شده در بالا دریافت نشود.  
بطور مثال کد زیر را در نظر بگیرد.  


```sql
-- ایجاد جدول موقت با ستون RowVersion
CREATE TABLE #TempProducts
(
    Id         INT IDENTITY PRIMARY KEY,
    Name       NVARCHAR(100),
    Price      DECIMAL(10, 2),
    RowVersion ROWVERSION
);

-- 🔹 شروع تراکنش T1
BEGIN TRANSACTION T1;

INSERT INTO #TempProducts (Name, Price)
VALUES ('Product T1', 1000);

SELECT SCOPE_IDENTITY() AS NewId, RowVersion
FROM #TempProducts
WHERE Name = 'Product T1';

-- مقدار RowVersion در همین تراکنش مقداردهی شده ولی هنوز COMMIT نشده است

-- 🔹 شروع تراکنش T2 (قبل از COMMIT شدن T1)
BEGIN TRANSACTION T2;

INSERT INTO #TempProducts (Name, Price)
VALUES ('Product T2', 2000);
-- مقدار RowVersion برای این رکورد مقداردهی شده ولی هنوز COMMIT نشده

-- 🔹 COMMIT کردن T1 قبل از T2
COMMIT TRANSACTION T2;

-- ⏳ تأخیر ۱ ثانیه
WAITFOR DELAY '00:00:01';

-- 🔹 مقدار RowVersion فعلی را ذخیره می‌کنیم
DECLARE @CurrentRowVersion VARBINARY(8);
SELECT @CurrentRowVersion = MAX(RowVersion)
FROM #TempProducts;

-- 🔹 دریافت تمام سطرهایی که RowVersion جدیدتری دارند
SELECT *
FROM #TempProducts
WHERE RowVersion > @CurrentRowVersion;
-- در اینجا مقدار مربوط به T1 نمایش داده نمی‌شود چون هنوز COMMIT نشده

-- 🔹 حالا T1 را COMMIT می‌کنیم
COMMIT TRANSACTION T1;

-- 🔹 بعد از COMMIT شدن T1 دوباره چک می‌کنیم
SELECT *
FROM #TempProducts
WHERE RowVersion > @CurrentRowVersion;

-- 🗑 پاک کردن جدول موقت
DROP TABLE #TempProducts;
```

در کد بالا دو تراکنش T1, T2 را داریم که هر دو در جدول یک سطر جدید ایجاد می‌کنند. تراکنش T1 قبل از T2 و زودتر شروع می‌شود اما بعد از تراکنش T2 در دیتابیس COMMIT می‌شود.  
در این حالت شناسه تخصیص داده شده به آن کوچکتر است. بطور مثال:  
 - T1 => Id=1
 - T2 => Id=2

در صورتی که تغییرات را قبل از کامیت تراکنش T1 و بعد از کامیت شدن تراکنش T2 بخوانید سطر حاصل از T2 را دریافت می‌کنید ولی سطر T1 دریافت نمی‌شود.  
همچنین با توجه به اینکه شناسه T2 را برای فراخوانی‌های آینده ذخیره کرده‌اید دیگر سطر T1 را در آینده دریافت نخواهید کرد.  