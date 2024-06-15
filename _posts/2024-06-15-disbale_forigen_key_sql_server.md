---
title: "غیرفعال سازی چک کردن کلید خارجی در SQL Server"
categories:
  - Sql
tags:
  - test
  - sql
  - mock
---

بعضی مواقع بطور مثال در زمان نوشتن Integration Test نیاز است بدون اینکه دیتا را در جدولی که به آن کلید خارجی داریم Insert کنیم، فقط در جدلی که به آن نیاز هست اضافه کنیم.  
در این مواقع توسط دستور زیر می‌توانید چک کردن کلید خارجی را غیرفعال کنید و سپس توسط کوئری دوم بعد از انجام کار آن را دوباره فعال کنید.  

```sql
ALTER TABLE [mySchema].[MyTable] NOCHECK CONSTRAINT [MyConstraintName]
```

```sql
ALTER TABLE [mySchema].[MyTable] CHECK CONSTRAINT [MyConstraintName]
```