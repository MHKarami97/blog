---
title: "دستور CROSS APPLY در SQL"
date: 2021-12-14T08:09:00-00:00
categories:
  - SQL
tags:
  - cross_apply
  - cross
  - sql_server
---

قبل از شروع این مطلب نیاز است که ابتدا با `CROSS` بیشتر آشنا شویم.  
روش استفاده از این نوع `JOIN` بصورت زیر است:  

```sql
SELECT
	T1,Id
  T2.Name
FROM
	T1
CROSS JOIN T2;
```

این دستور نیاز به شرط `ON` ندارد زیرا به ازای تمام سطرهای هر دو جدول انجام می‌شود.  
به این صورت که ابتدا به ازای سطر اول از حدول اول، آن را با تمام سطرهای جدول دوم جوین می‌زند و ...  
تا در نهایت یک خروجی `n*m` که در آن n برابر تعداد سطرهای حدول اول و m تعداد سطرهای جدول دوم است ایجاد شود.  

همانطور که از خروجی آن معلوم است، تعداد سطرهای خروجی آن زیاد است و نباید خیلی از این نوع join استفاده کنید و در صورت استفاده نیز بهتر است با دستور WHERE تعداد خروجی‌های آن را محدود کنید.  

از نمونه استفاده‌های این نوع join می‌توان به شرایطی اشاره کرد که شما در یک جدول لیست کالاها و در یک جدول دیگر لیست فروشگاه‌هایی که کالاها را دارند ذخیره می‌کنید و می‌خواهید به ازای هر کالا فروشگاهی که آن کالا را دارد بدست آوردید:  

```sql
SELECT
    p.ProductId,
    p.ProductName,
    s.StoreId
FROM
    production.products p
CROSS JOIN sales.stores s
WHERE s.Quantity > 0
ORDER BY
    ProductName;
```

اطلاعات بیشتر:  
[Cross Join](https://docs.microsoft.com/en-us/u-sql/statements-and-expressions/select/from/joins/cross-join)  

اکنون به دستور `Cross Apply` می‌رسیم. حالت دیگر این دستور `OUTER APPLY` است.  
در حالت اول فقط به ازای وضعیت‌هایی که رکورد جدول اول با رکوردهای جدول دوم Match می‌شود، انجام می‌شود. شبیه به `INNER JOIN` اما حالت دوم به ازای تمام سطرها انجام می‌شود و در صورت نبود در جدول دوم، بجای آن NULL قرار می‌گیرد.  

```sql
SELECT * FROM Department D 
CROSS APPLY 
   ( 
   SELECT * FROM Employee E
   WHERE E.DepartmentID = D.DepartmentID 
   ) A 
GO
 
SELECT * FROM Department D
INNER JOIN Employee E ON D.DepartmentID = E.DepartmentID 
```

```sql
SELECT * FROM Department D 
OUTER APPLY 
   ( 
   SELECT * FROM Employee E 
   WHERE E.DepartmentID = D.DepartmentID 
   ) A 
GO
 
SELECT * FROM Department D 
LEFT OUTER JOIN Employee E ON D.DepartmentID = E.DepartmentID 
```

خروجی کوئری‌های نوشته شده در هر بخش با یکدیگر برابر است و execution plan نیز تقریبا برابر است.  
البته در تمام موارد موارد گفته شده خروجی یکسان ندارند، بطور مثال فرض کنید:  

ما 2 جدول زیر را داریم:  

```sql

MASTER TABLE

x------x--------------------x
| Id   |        Name        |
x------x--------------------x
|  1   |          A         |
|  2   |          B         |
|  3   |          C         |
x------x--------------------x


DETAILS TABLE

x------x--------------------x-------x
| Id   |      PERIOD        |   QTY |
x------x--------------------x-------x
|  1   |   2014-01-13       |   10  |
|  1   |   2014-01-11       |   15  |
|  1   |   2014-01-12       |   20  |
|  2   |   2014-01-06       |   30  |
|  2   |   2014-01-08       |   40  |
x------x--------------------x-------x
```

در حالت اول می‌خواهیم عملیات join را فقط به ازای آخرین 2 تاریخ جدول Details انجام دهیم. برای این کار از کوئری زیر استفاده می‌کنیم:  

```sql
SELECT M.ID,
            M.NAME,
            D.PERIOD,
            D.QTY
FROM MASTER M
INNER JOIN
(
    SELECT TOP 2 ID,
                         PERIOD,
                         QTY 
    FROM DETAILS D      
    ORDER BY CAST(PERIOD AS DATE)DESC
)D
ON M.ID=D.ID
```
که خروجی آن بصورت زیر است:  

```sql
x------x---------x--------------x-------x
|  Id  |   Name  |   PERIOD     |  QTY  |
x------x---------x--------------x-------x
|   1  |   A     | 2014-01-13   |  10   |
|   1  |   A     | 2014-01-12   |  20   |
x------x---------x--------------x-------x
```
سپس کوئری بالا را بصورت زیر بازنویسی می‌کنیم:  

```sql
SELECT M.ID,
            M.NAME,
            D.PERIOD,
            D.QTY
FROM MASTER M
CROSS APPLY
(
    SELECT TOP 2 ID,
                         PERIOD,
                         QTY 
    FROM DETAILS D  
    WHERE M.ID=D.ID
    ORDER BY CAST(PERIOD AS DATE)DESC
)D
```

که خروجی آن بصورت زیر است:  

```sql
x------x---------x--------------x-------x
|  Id  |   Name  |   PERIOD     |  QTY  |
x------x---------x--------------x-------x
|   1  |   A     | 2014-01-13   |  10   |
|   1  |   A     | 2014-01-12   |  20   |
|   2  |   B     | 2014-01-08   |  40   |
|   2  |   B     | 2014-01-06   |  30   |
x------x---------x--------------x-------x
```

همانطور که می‌بینید خروجی اول فقط 2 تاریخ که از همه بزرگتر هستند را گرفته است که با شرایط ما که می‌خواستیم به ازای هر سطر از جدول Master آخرین 2 تاریخ را بگیرد، فرق دارد که خروجی دلخواه ما در کوئری دوم آمده است.  

در حالت دوم ما از Cross Join می‌توانیم برای select از function داریم استفاده کنیم.  

```sql
SELECT M.ID,
            M.NAME,
            C.PERIOD,
            C.QTY
FROM MASTER M
CROSS APPLY dbo.MyFunc(M.ID) C
```

که در آن MyFunc بصورت زیر است:  

```sql
CREATE FUNCTION MyFunc 
(   
    @Id INT 
)
RETURNS TABLE 
AS
RETURN 
(
    SELECT ID,PERIOD,QTY 
    FROM DETAILS
    WHERE ID=@Id
)
```

به عنوان مثال دیگر فرض کنید که ما در جدول Post لیست Tag های آن را در یک ستون به اسم Tags ذخیره می‌کنیم که تگ‌ها با `,` از هم جدا شده‌اند و ما می‌خواهیم تگ‌های استفاده شده توسط یک کاربر را بدست آوریم:  

```sql
SELECT DISTINCT values AS UsedTags
FROM Post p
CROSS APPLY STRING_SPLIT(p.Tag, ',');
```

همچنین زمان اجرا شدن 2 کوئری نیز در بعضی مواقع یکسان نیست، بطور مثال 2 کوئری زیر اگرچه خروجی یکسان دارند اما استفاده از cross apply بسیار سریع‌تر است:  

```sql
WITH    t AS 
        (
        SELECT  1 AS id
        UNION ALL
        SELECT  2
        )
SELECT  *
FROM    t
CROSS APPLY
        (
        SELECT  TOP (t.id) m.*
        FROM    master m
        ORDER BY
                id
        ) q
```

```sql
WITH    q AS
        (
        SELECT  *,
                     ROW_NUMBER() OVER (ORDER BY id) AS rn
        FROM    master
        ),
        t AS 
        (
        SELECT  1 AS id
        UNION ALL
        SELECT  2
        )
SELECT  *
FROM    t
JOIN    q
ON      q.rn <= t.id
```

اطلاعات بیشتر:  
[inner join vs cross apply](https://explainextended.com/2009/07/16/inner-join-vs-cross-apply/)