---
title: "منتقل کردن دیتا به دیتابیس جدید در SQL Server"
date: 2022-10-17T00:00:00-00:00
categories:
  - Sql
tags:
  - sql
  - transfer
  - count
---

در یکی از پروژه‌های شرکت نیاز به انتقال تمام دیتا دیتابیس به یک دیتابیس جدید بود. برای انجام این کار بعد از بررسی کوئری‌های مختلف به کوئری زیر رسیدیم. از راه‌های دیگر انتقال می‌توان به جوین زدن بین دو جدول برای جلوگیری از انتقال دیتا تکراری اشاره کرد که البته اینکار سنگین‌تر از این کوئری بود. کوئری زیر در واقع دیتا تکراری را بررسی نمی‌کند و تمام دیتا را منتقل می‌کند. ممکن است این کوئری به خطا بخورد و بخشی از دیتا منتقل نشود که برای حل این مشکل از کوئری دوم استفاده می‌شود و برای کارکرد درست آن هم `Id` جدول هم منتقل می‌شود تا در انتها بتوان صحت دیتا و همچنین انتقال دیتاهای جا مانده را انجام داد:  

>> SET IDENTITY_INSERT [DB2].[dbo].[Request] ON

یکی از موارد مهم در کوئری که در واقع از ویژگی‌های خود SQL می‌شود، ادامه دادن کوئری در مواقع به خطا خوردن است. بطور مثال در کوئری زیر خط 3 و 4 هم اجرا می‌شوند:  

```sql
SELECT 1
SELECT 2
SELECT 5/0
SELECT 3
SELECT 4
```

پس در این کوئری حتی اگر یک Batch هم به خطا بخورد بقیه دیتا منتقل می‌شود و فقط دیتا همان Batch جا می‌ماند.  
همچنین دقت کنید که اگر یک Batch را یک عدد بطور مثال یک میلیون قرار دهید، به معنی انتقال یک میلیون سطر نیست. زیرا ممکن است در خود Id ها گپ وجود داشته باشد که از دلایل آن می‌توان به پاک شدن دیتا اشاره کرد.  

قرار دادن `Batch Size` هم به دلیل جلوگیری از پر شدن Log دیتابیس است. در غیر این صورت به دلیل پر شدن تمام ترافیک شبکه و فایل لاگ دیتابیس به حالت Recovery می‌رود و هیچ کوئری دیگر را هم نمی‌توانید بر روی آن اجرا کنید. اندازه هر batch را هم با توجه به جدول اصلی، منابع سرور و .. می‌توانید تعیین کنید. بطور مثال اگر جدول اصلی 500 میلیون سطر داشته باشد می‌توانید از اندازه 5 میلیون و اگر 1 میلیارد سطر داشته باشد از مقدار 100 میلیون استفاده کنید.  

مقدار `WITH (NOLOCK)` برای این است که برای خواندن مقدار از جدول اولیه منتظر نمانیم و اگر کوئری سنگینی در حال انجام بود هم بتوان مقادیر را از جدول خواند.   

[nolock](https://learn.microsoft.com/en-us/sql/t-sql/queries/hints-transact-sql-table?view=sql-server-ver16)  

اجرا کردن این کوئری در زمان‌های متفاوت هم ممکن است سرعت‌های متفاوتی داشته باشد. بطور مثال اگر افراد دیگری هم در حال اجرا کوئری بر روی دیتابیس اولیه باشند با توجه به مشغول شدن بخشی از منابع سرور اولیه با کندی مواجه می‌شوید.  

```sql
DECLARE @batchSize BIGINT;
DECLARE @MinId BIGINT;
DECLARE @MaxId BIGINT;
SET @batchSize = 1000000;

SELECT @MinId = MIN(Id),
       @MaxId = MAX(Id)
FROM [DB1].[dbo].[Request]

SET IDENTITY_INSERT [DB2].[dbo].[Request] ON

WHILE (@MinId <= @MaxId)
    BEGIN
        INSERT INTO [DB2].[dbo].[Request]
        ( [Id]
        , [DecisionId]
        , [ErrorCode]
        , [Description]
        , [CreationDate])
        SELECT I.[Id]
             , I.[DecisionId]
             , I.[ErrorCode]
             , I.[Description]
             , I.[CreationDate]
        FROM [DB1].[dbo].[Request] I WITH (NOLOCK)
        WHERE I.Id >= @MinId
          AND I.Id < @MinId + @batchSize

        SET @MinId = @MinId + @batchSize
    END

SET IDENTITY_INSERT [DB2].[dbo].[Request] OFF
```

ممکن است کوئری بالا به دلایل مختلف به خطا بخورد. بطور مثال شبکه قطع شود، دیتا اشتباه باشد و ...
در این مواقع می‌توانید با کوئری زیر دیتایی که منتقل نشده است را منتقل کنید.  

یکی از خطاهای کوئری بالا خطا زیر ممکن است باشد:  

>> A transport-level error has occurred when receiving results from the server. (provider: TCP Provider, error: 0 - The semaphore timeout period has expired.)

این خطا در مواقعی پیش می‌آید که Session شما قطع می‌شود که از دلایل آن می‌توان به Policy های تیم امنیت شرکت برای جلوگیری از اجرا کوئری‌ها در ساعات خاصی از روز باشد.  

```sql
DECLARE @CurrentId BIGINT = 1;
DECLARE @BatchSize INT = 1000000;
DECLARE @MaxId BIGINT = (SELECT MAX(Id)
                         FROM [DB1].[dbo].[Request]);

WHILE (@CurrentId < @MaxId)
    BEGIN
        INSERT INTO [DB2].[dbo].[Request](Id, DecisionId, ErrorCode, Description, CreationDate)
        SELECT I1.Id, I1.DecisionId, I1.ErrorCode, I1.Description, I1.CreationDate
        FROM [DB1].[dbo].[Request] I1
                 LEFT JOIN
             [DB2].[dbo].[Request] I2 ON I1.Id = I2.Id
        WHERE I2.Id IS NULL
          AND I1.Id BETWEEN (@CurrentId + 1) AND (@CurrentId + @BatchSize)
        ORDER BY I.DecisionId ASC

        SET @CurrentId += @BatchSize;
    END
```

در کوئری بالا دقت کنید که `ORDER BY` مهم است در غیر این صورت ضمانتی برای منتقل شدن همه دیتا و همچنین ترتیب وجود ندارد. همچنین ممکن است به خطاهای ظاهرا نامشخص مثل `Duplicate Primary Key` برخورد کنید.  

برای مطمئن شدن از انتقال کامل دیتا هم می‌توانید از کوئری زیر استفاده کنید. استفاده از `COUNT` راحت‌ترین راه است که البته اگر دیتا زیاد باشد کمی زمانبر می‌شود.  
خط آخر که در واقع یک SP است بسیار سریع تعداد سطرهای جدول را نشان می‌دهد.  

```sql
SELECT COUNT(*) FROM [DB1].[dbo].[Request] WITH (NOLOCK)
SELECT COUNT(*) FROM [DB2].[dbo].[Request] WITH (NOLOCK)

EXEC sys.sp_spaceused @objname = N'[dbo].[Request]'
```

البته SP گفته شده در زمان انتقال مقدار را نشان نمی‌دهد. مخصوصا اگر batchSize بزرگ باشد. بطوری که کل جدول 500 میلیون رکورد داشت اما این SP 600 میلیون را نشان می‌داد.  

[spaceused](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-spaceused-transact-sql?view=sql-server-ver16)  