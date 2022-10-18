---
title: "Collation حساس به حروف بزرگ و کوچک در SQL Server"
date: 2022-10-17T00:00:00-00:00
categories:
  - Sql
tags:
  - sql
  - collation
  - case sensitive
---

در یکی از پروژه‌ها نیاز به انتقال تمام اطلاعات جداول به دیتابیس جدید بود که در انجام این کار که در مطلب قبلی کلیات آن گفته شد به چند خطا هم برخورد کردیم.  
خطا زیر را ابتدا مشاهده کردیم:  

>> Cannot insert duplicate key row in object 'dbo.Response' with unique index 'FUIX_Dbo_Response(MessageId)'. The duplicate key value is (331BAv80AAAAAAAA).

بعد از بررسی‌های بیشتر به این موارد رسیدیم:  
دیتابیس جدید توسط Generate Script از دیتابیس قدیم ساخته شده بود و نکته‌ای که وجود داشت این بود که این ابزار SQL Server Management Script مشکلی دارد که `Collation` جداول را منتقل نمی‌کند.  
خطا بالا هم دقیقا به همین دلیل بود. در دیتابیس قدیمی Collation مقدار `PERSIAN_CS_AI` داشت که حساس به حروف کوچک و بزرگ بود. پس مقدار `331BAv80AAAAAAAA` با `331BAV80AAAAAAAA` تفاوت می‌کرد اما در دیتابیس دوم این مورد رعایت نشده بود که باعث خطا بالا می‌شد.  
همچنین ابزار گفته شده Collation خود دیتابیس هم منتقل نمی‌کند که باعث شده بود برای دیتابیس جدید مقدار `SQL_Latin1_General_CP1_CI_AS` بگیرد که باعث خراب شدن دیتا و نیاز به انتقال دوباره تمام اطلاعات بود.  
همچنین بعد از انتقال اطلاعات امکان تغییر Collation دیتابیس بسیار سخت است و لازم است تمام Constraint های دیتابیس پاک شود که حتی بعد از این کار اطلاعاتی که قبلا منتقل شده بودند هم درست نمی‌شود و بطور مثال حروف فارسی را علامت سوال نشان می‌دهد.   

[collations](https://learn.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support?view=sql-server-ver16)  

[collations case sensitivity](https://learn.microsoft.com/en-us/ef/core/miscellaneous/collations-and-case-sensitivity)  