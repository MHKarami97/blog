---
title: "مقداردهی به Variable در SQL توسط SELECT"
date: 2021-12-26T11:07:00-00:00
categories:
  - SQL
tags:
  - variable
  - sql
  - sql_server
---

گاهی مواقع نیاز است تا یک متغیر را توسط دستور SELECT مقداردهی کنید و از آن متغیر در کدهای خود استفاده کنید.  
راحت ترین راه کوئری زیر است که فقط یک مورد را مقداردهی می‌کند:  

```sql
DECLARE @var1 INT;

SET var1 = (SELECT Id FROM MyTable WHERE Title = 'Mohammad');
```

اگر شرط کد بالا خروجی بیشتر از یک مورد داشته باشد، کد بالا با خطا روبرو می‌شود.  
همچنین توسط کد بالا فقط یک متغیر را می‌توانید مقدار‌دهی کنید.  
اگر نیاز به مقداردهی چند متغیر داشتید و یا اگر می‌خواستید تعداد کوئری‌های اجرا شده  را کم کنید، می‌توانید از کوئری زیر استفاده کنید:  

```sql
DECLARE @var1 INT,
        @var2 INT,
        @var3 INT;

SELECT @var1 = field1,
       @var2 = field2,
       @var3 = field3
FROM MyTable
WHERE Id = 1;
```

کد بالا در صورتی که شرط اعمال شده باعث چند خروجی شود، خطا نمی‌دهد ولی مقدار معتبری را به متغیر نمی‌دهد.  

روش درست استفاده بصورت زیر است تا با خطا یا مقداردهی اشتباه روبرو نشوید:  

```sql
DECLARE @var1 INT,
        @var2 INT,
        @var3 INT;

SELECT TOP 1 @var1 = field1,
             @var2 = field2,
             @var3 = field3
FROM MyTable
WHERE Id = 1
ORDER BY Title DESC;
```

```sql
DECLARE @var1 INT;

SET var1 = (SELECT TOP 1 Id FROM MyTable WHERE Title = 'Mohammad' ORDER BY Title DESC);
```

در حالت دوم دقت کنید که حتما `()` را قرار دهید تا با خطا روبرو نشوید.  