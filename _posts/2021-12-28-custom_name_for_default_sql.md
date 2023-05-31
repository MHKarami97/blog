---
title: "قرار دادن نام دلخواه برای DEFAULT در SQL"
categories:
  - SQL
tags:
  - default
  - sql
  - sql_server
  - identity
  - constraint
---

یکی از مواردی که در زمان طراحی دیتابیس باید رعایت شود، بحث نامگذاری درست آیتم‌ها است.  
بصورت پیش‌فرض SQL SERVER نام‌های پیش‌فرض بطور مثال برای کلید داخلی، کلید خارجی، مقدار پیش فرض و ... می‌دهد که بهتر است شما نام خود را طبق قوانینی که برای نامگذاری وجود دارد قرار دهید.  

بطور مثال کد زیر را در نظر بگیرید:  

```sql
CREATE TABLE [dbo].[MyDb]
(
	  [Id] INT IDENTITY (1, 1) NOT NULL, 
    [Title] NVARCHAR(250) NOT NULL,
    [CreationDateTime] DATETIME2(2) NOT NULL,
    [UserId] INT NOT NULL, 
    [IsFavorite] BIT NOT NULL CONSTRAINT [DF_Dbo_MyDb(IsFavorite)] DEFAULT (0),
    CONSTRAINT [PK_Dbo_MyDb] PRIMARY KEY CLUSTERED ([Id] ASC)
);
```

توسط بخش زیر نام دلخواه خود را که `DF_Dbo_MyDb(IsFavorite)` است، برای `DEFAULT` قرار داده‌ایم.  

```sql
[IsFavorite] BIT NOT NULL CONSTRAINT [DF_Dbo_MyDb(IsFavorite)] DEFAULT (0)
```

توسط کد زیر نیز نام دلخواه برای `کلید جدول` که `PK_Dbo_MyDb` است را قرار داده‌ایم

```sql
CONSTRAINT [PK_Dbo_MyDb] PRIMARY KEY CLUSTERED ([Id] ASC)
```

اگر کد بالا را بصورت زیر استفاده کنید، نام‌های پیش‌فرض برای موارد گفته شده قرار داده می‌شود.  

```sql
CREATE TABLE [dbo].[MyDb]
(
	  [Id] INT IDENTITY (1, 1) NOT NULL, 
    [Title] NVARCHAR(250) NOT NULL,
    [CreationDateTime] DATETIME2(2) NOT NULL,
    [UserId] INT NOT NULL, 
    [IsFavorite] BIT NOT NULL DEFAULT (0)
);
```
