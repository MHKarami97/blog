---
title: "قلمرو دید در Derived Table"
date: 2021-12-15T08:09:00-00:00
categories:
  - SQL
tags:
  - derived_table
  - vision
  - sql_server
---

در زمان استفاده از SubQuery ها باید حواستان به قلمرو دید باشد.  
بصورت کلی تمام ستون‌های جدول اول در بخش دو دردسترس نیست.  

دستور زیر با خطا مواجه خواهد زیرا `a.Id` در SELECT دوم دردسترس نیست.  

```sql
SELECT a.Id,
       c.Name
FROM MyTable a
         JOIN (SELECT b.Name FROM MyTable2 b WHERE b.Id2 = a.Id) c
```

روش درست استفاده بصورت زیر است:  

```sql
SELECT a.Id,
       c.Name
FROM MyTable a
         JOIN (SELECT b.Name,
                            b.Id2
                     FROM MyTable2 b) c
WHERE c.Id2 = a.Id
```

تعریف `Derived Table` را نیز می‌توان بصورت زیر بیان کرد:  
یک Derived Table در واقع یک `Inner Query` است که در بخش `FROM` یا `JOIN` یک کوئری اصلی می‌آید.

اطلاعات بیشتر:  
[Subqueries](https://docs.microsoft.com/en-us/sql/relational-databases/performance/subqueries?view=sql-server-ver15)  