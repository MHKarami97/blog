---
title: "کم کردن حجم Log File دیتابیس در SQL Server"
categories:
  - SQL
tags:
  - sql_server
  - shrink
  - log
  - database
---

در پروژه ای که مدتی پیش روش کار میکرد، نیاز به وارد کردن دیتا تست زیادی بود که بعد از انجام پروژه هم پاک شدن، بعد از انجام پروژه متوجه شدن حجم درایو C خیلی کم شده
<br />
بعد از بررسی متوجه شدم که Log دیتابیس گفته شده با اینکه دیتابیس کلا خالی بود ولی حجم بسیار زیادی رو گرفته
<br />
بعد از جستجو داخل اینترنت و انجام کارهای گفته شده هم نتیجه ای حاصل نشد
<br />
بعضی از کارهایی که انجام دادم:
<br />

- استفاده از Shrink File
- استفاده از shrink در سه حالت مختلفی که داره
- تغییر Recovery به Simple و انجام دوباره کارهای بالا
- و ...

<br />
در نهایت با یکی از دوستان تیم دیتابیس تماس گرفتن و ایشون کدهای زیر رو وارد کردن که با موفقیت حجم لاگ کم شد
<br />

```sql
SELECT name, log_reuse_wait_desc
FROM sys.DATABASEs;
GO

sp_helpdb 'marketdepth';
GO

ALTER DATABASE MarketDepth SET RECOVERY SIMPLE;
GO

DBCC SHRINKFILE (MarketDepth_log,512)
GO

DBCC OPENTRAN;
CHECKPOINT;
CHECKPOINT;
CHECKPOINT;
GO

DBCC SHRINKFILE (MarketDepth_log,512)
GO

ALTER DATABASE MarketDepth SET RECOVERY FULL;
GO
```

داخل کوئری بالا:
<br />
دستور اول مشخصات دیتابیس ها رو نشون میده که از داخلش میشه اطلاعات مختلف مثل simple یا full بودن حالت ریکاوری و اطلاعات دیگه که داخل select هستن رو دید
<br />
دستور دوم اطلاعات خاص یه دیتابیس مثل محل و حجم فایل های دیتابیس رو میده
<br />
دستور بعدی حالت ریکاوری رو به simple تغییر میده
<br />
دستور بعدی فایل لاگ رو shrink میکنه
<br />
دستور بعدی که قسمت مهمش هست و مشکل من رو حل کرد، توضیحات زیادی داره که در ادامه میگم
<br />
دستور آخر هم حالت recovery رو دوباره به full تغییر میده
<br />
<br />
اول از همه داخل دیتابیس من چندتا تیبل In Memory هم بود و متوجه شدم دلیل کار نکردن دستورات قبلی هم این بوده
<br />
توضیحات کامل رو در این باره میتونید از لینک های زیر بخونید:
<br />

[لینک اول](https://www.dntips.ir/post/1731/%d8%a8%d8%a7%d8%b2%db%8c%d8%a7%d8%a8%db%8c-%d9%be%d8%a7%db%8c%da%af%d8%a7%d9%87-%d8%af%d8%a7%d8%af%d9%87-database-recovery)  
<br />
[لینک دوم](https://nikamooz.com/checkpoint-in-memory/)  
<br />
[لینک سوم](https://nikamooz.com/checkpoint-how-it-works-and-what-the-log/)  
<br />
بصورت خلاصه میشه دلیل زیر رو آورد
<br />
<br />
در Disk-Based Table به ازای Recovery Model Full بعد از گرفتن Transaction Log Backup لاگ فایل Truncate می‌شود.
<br />
اما اگر Memory Optimized Table داشته باشیم Transaction Log Backup لزوما باعث Truncate شدن لاگ فایل نمی‌شود بلکه Checkpoint هم نیاز است.
<br />
<br />
اگه لینک های گفته شده رو هم مطالعه کنید درک خیلی خوبی در این باره پیدا مکنید

