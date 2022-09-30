---
title: "استفاده از IF در دستور WHERE SQL"
date: 2022-04-25T12:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - update
  - select
  - join
---

بصورت پیش فرض امکان استفاده از شرط IF در بخش WHERE یک کوئری وجود ندارد و اگر نیاز به بررسی شرط در حالت‌های مختلف داشتید، باید از ترفندهای مختلف استفاده کنید.  
یکی از این ترفندها را در کوئری زیر مشاهده می‌کنید. با استفاده از این کوئری شما می‌توانید شرط‌های خود را شبیه سازی کنید.  
بطور مثال اگر accountType برابر با othersAccountTypeValue بود شرط اول و اگر مقدار دیگری داشت شرط دوم اجرا شود.  


```sql
DECLARE @accountType TINYINT;
DECLARE @price BIGINT;
DECLARE @buySideValue TINYINT = 1;
DECLARE @othersAccountTypeValue TINYINT = 5;
DECLARE @clientAccountTypeValue TINYINT = 1;

SELECT *
FROM Decision
WHERE Quantity > 0
  AND (
        (
                    @accountType = @othersAccountTypeValue
                AND
                    side = @buySideValue
        )
        OR
        (
                    @accountType = @clientAccountTypeValue
                AND
                    Price > @price
        )
    );
```