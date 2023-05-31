---
title: "دستور SYNONYM در SQL"
categories:
  - SQL
tags:
  - synonym
  - host
  - sql_server
---

شی Synonym در دیتابیس در واقع یک نام جایگزین برای اشیا دیگر است که نکته مهم در آن این است که این شی می‌تواند در یک دیتابیس و حتی سرور دیگر باشد. پس با این امکان شما می‌توانید از جداول و دیگر اشیا یک سرور دیگر در کوئری‌های خود استفاده کنید.  

یکی دیگر از کاربردهای آن در مواقعی هست که شما می‌خواهید وابستگی استفاده کنندگان از اشیا دیتابیس بطور مثال View را کم کنید و وابستگی‌ها را مستقیم به نام‌های استفاده شده در دیتابیس ندهید، در این مواقع می‌توانید توسط این امکان یک شی واسط ایجاد کنید.  

دستور ایجاد بصورت زیر است:  

```sql
CREATE SYNONYM MyNewTable   
FOR MyServer.MyDatabase.MySchema.MyTable;  
```

اگر شی مورد نظر در یک سرور دیگر باشد کافی است نام آن را بجای MyServer بیاورید تا به آن شی در سرور خود دسترسی داشته باشید.  
از این امکان بطور مثال در MicroService ها که هر بخش دیتابیس جدا دارد می‌توانید استفاده کنید.  

برای شی‌های زیر می‌توانید Synonym بسازید:  

 - Assembly (CLR) stored procedure
 - Assembly (CLR) table-valued function
 - Assembly (CLR) scalar function
 - Assembly (CLR) aggregate functions
 - Replication-filter-procedure
 - Extended stored procedure
 - SQL scalar function
 - SQL table-valued function
 - SQL inline-tabled-valued function
 - SQL stored procedure
 - View
 - User-defined table

برای پاک کردن آن نیز می‌توانید از دستور زیر استفاده کنید، در این حالت هیچ اروری مربوط که داشتن Reference به این شی در دیتابیس نمی‌گیرید. برای حالت ویرایش نیز به همین صورت است که شما شی اصلی را می‌توانید ویرایش کنید.  

```sql
DROP SYNONYM IF EXISTS MyNewTable;
```

در نرم افزار `Microsoft SQL Server Management Studio` در پوشه `Synonyms` می‌توانید اشیا‌ای که ساخته‌اید را مشاهده کنید.  
در این بخش می‌توانید دسترسی‌های شی ساخته شده را نیز کنترل کنید:  

 - CONTROL
 - DELETE
 - EXECUTE
 - INSERT
 - SELECT
 - TAKE OWNERSHIP
 - UPDATE
 - VIEW DEFINITION

```sql
CREATE USER [myUser] FOR LOGIN [myUser]
GO
GRANT SELECT ON [dbo].[MyNewTable] TO [myUser]
```

<p align="center" >
  <img src="/assets/img/synonym.png" alt="mhkarami97" width="600" />
</p>

بعد از ساخته شدن نیز شبیه به دیگر شی‌های دیتابیس می‌توانید از synonym ساخته شده استفاده کنید:  

```sql
SELECT TOP 10 * FROM MyNewTable;
```

بصورت خلاصه از Synonym در حالت‌های زیر می‌توانید استفاده کنید:  

  - یک لایه abstraction برای شی‌ها
  - کوتاه کردن نام شی‌ها
  - دسترسی به شی‌ها در دیگر سرورها
  - جلوگیری از وابستگی مستقیم به اشیا دیتابیس

اطلاعات بیشتر:  
[synonyms](https://docs.microsoft.com/en-us/sql/relational-databases/synonyms/synonyms-database-engine?view=sql-server-ver15)  