---
title: "استفاده از unique index در ستون با مقدار null توسط Filter Index"
date: 2021-03-15T11:35:00-00:00
categories:
  - SQL
tags:
  - sql
  - index
  - sqlServer
---

توی تغییری که لازم بود در دیتابیس بدیم، نیاز بود که یه ستون جدید به جدول اضافه بشه که مقدارش GUID بود
<br />
پس نوعش رو Unique در نظر گرفته بودیم
<br />
این فیلد میتونستن مقدار Null هم داشته باشه
<br />
همه چی درست بود تا اینکه توی محیط تست خطاها شروع شدن
<br />
خطاها هم میگفتن که نمیشه ستون جدیدی با مقدار نال در این فیلد جدید به دیتابیس اضافه کرد
<br />
در واقع مقدار Null رو Unique در نظر میگرفت و برای مقادیر جدید جلوی ثبت سطر جدید با مقدار Null رو میگرفت
<br />
<br />
با کمی جستجو به Filter Index رسیدیم که راه حل مشکل ما بود
<br />
میتونید دربارش در داکیومنت خودش مطالعه کنید

[https://docs.microsoft.com/en-us/sql/relational-databases/indexes/create-filtered-indexes?view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/relational-databases/indexes/create-filtered-indexes?view=sql-server-ver15) 

توسط این نوع از Index شما میتونید شرط بزارید که ایندکس در چه زمان هایی ساخته بشه
<br />
پس ما شرطی گذشتیم تا فقط برای مقادیر غیر Null در این فیلد جدید ایندکس ساخته بشه
<br />
برای اینکار میتونید شبیه عکس زیر تو قسمت Filter هر Index شرطی که میخواید رو بزارید
<br />
البته قبلش لینک بالا رو مطالعه کنید تا از پشتیبانی این نوع ایندکس در جدولی که میخواید مطمئن بشید

<p align="center" >
  <img src="https://i.postimg.cc/XYrxLkBZ/Screenshot-2021-03-15-104908-min.png" alt="mhkarami97" width="400" />
</p>

تو این قسمت کافی هست شرطی که میخواید رو بزارید
<br />
بطور مثال میتونید بگید اگه فیلد مقدار خاصی رو داشتن ایندکس بزار


```sql
[ISR] IS NOT NULL
```

یا اگه خواستید بصورت کوئری عمل کنید میتونید بصورت زیر انجام بدید

```sql
CREATE UNIQUE NONCLUSTERED INDEX [FUIX_New_Request(ISR)] ON [new].[Request] ([ISR]) WHERE ([ISR] IS NOT NULL) ON [Tse]
```
