---
title: "پاس دادن لیست به عنوان ورودی SP"
categories:
  - SQL
tags:
  - sql
  - sql_server
  - dapper
  - user_defined_table_type
---

یکی از امکانات خوب دیتابیس SQL Server امکان پاس دادن لیست به عنوان ورودی به یک `stored procedure` است که کار شما را برای مواقعی که نیاز به پاس دادن تعداد زیادی پارامتر به SP دارید و یا از قابلیت Bulk Insert می‌خواهید استفاده کنید، راحت می‌کند.  
کتابخانه Dapper بصورت پیش فرض قابلیت Bulk Insert را ندارد که بجای آن می‌توانید از روش گفته شده در این آموزش استفاده کنید.  
همچنین مقایسه این دو امکان را می‌توانید در لینکی که در انتها آمده، مشاهده کنید.  

برای این کار نیاز است که ابتدا یک `user defined table type` بسازید که روش ساخت آن بصورت زیر است که در آن شبیه ساخت یک جدول، Property های آن را مشخص می‌کنید.  

```sql
CREATE TYPE [dbo].[UserDefinedSetting] AS TABLE
(
    [UserId] INT,
    [Scope]  VARCHAR(50),
    [Key]    VARCHAR(255),
    [Value]  VARCHAR(200)
);
```

اکنون کافی است Type ساخته شده در بالا را به عنوان ورودی به SP خود بدهید.  
اکنون از آن ورودی می‌توانید به صورت یک جدول عادی در SP خود استفاده کنید.  
بطور مثال در کد زیر تیبل اصلی با پارامتر ورودی Join زده شده است تا برای آپدیت کردن و یا ایجاد سطر جدید در دیتابیس استفاده شود.  

```sql
CREATE PROCEDURE [dbo].[SpSetOrUpdateUserSetting] @Settings UserDefinedSetting READONLY
AS
BEGIN
    SET NOCOUNT ON;

    UPDATE [dbo].UserSetting
    SET [Value] = s.[Value]
    FROM [dbo].Usersetting us
             JOIN
         @Settings s
         ON us.[UserId] = s.[UserId]
             AND us.[Scope] = s.[Scope]
             AND us.[Key] = s.[Key];

    INSERT INTO [dbo].UserSetting(UserId, Scope, [Key], [Value])
    SELECT s.[UserId], s.[Scope], s.[Key], s.[Value]
    FROM @Settings s
             LEFT JOIN [dbo].Usersetting us
                       ON us.[UserId] = s.[UserId]
                           AND us.[Scope] = s.[Scope]
                           AND us.[Key] = s.[Key]
    WHERE us.[Id] IS NULL;
END
```

برای پاس دادن این پارامتر به SP ساخته شده نیز کافی است بصورت زیر عمل کنید.  
در کد زیر از Dapper برای ارتباط با دیتابیس استفاده شده است که ورودی متود یک لیست می‌باشد.  
ابتدا یک `DataTable` با ستون های `user defined table type` که در ابتدا ساخته شده بود، ایجاد شده است.  
سپس مقادیر لیست ورودی به آن اضافه شده است.  
در انتها جدول ساخته شده به عنوان پارامتر تعیین شده است.  
نکته مهم در این قسمت `Settings` است که باید همنام با پارامتر ورودی SP که `@Settings` است، باشد.  
همچنین `UserDefinedSetting` نیز باید همنام با `user defined table type` باشد.  

```c#
private string Schema { get; } = "trm";
private const string UserDefinedSetting ="UserDefinedSetting";

public async Task<bool> SetOrUpdateSettings(List<Models.UserSetting> settings) 
{
    var procedure = $"[{Schema}].[SpSetOrUpdateUserSetting]";

    var data = new DataTable();
    data.Columns.Add(nameof(Models.UserSetting.UserId));
    data.Columns.Add(nameof(Models.UserSetting.Scope));
    data.Columns.Add(nameof(Models.UserSetting.Key));
    data.Columns.Add(nameof(Models.UserSetting.Value));

    foreach (var setting in settings)
    {
        data.Rows.Add(setting.UserId, setting.Scope, setting.Key, setting.Value);
    }

    var param = new
    {
        Settings = data.AsTableValuedParameter($"{Schema}.{UserDefinedSetting}")
    };

    var result = await Connection.ExecuteScalarAsync<bool>(procedure, param,
            commandType: CommandType.StoredProcedure);

    return true;
}
```

اطلاعات بیشتر:  

[microsoft](https://docs.microsoft.com/en-us/sql/relational-databases/tables/use-table-valued-parameters-database-engine?view=sql-server-ver15)  

[microsoft](https://docs.microsoft.com/en-us/sql/t-sql/statements/create-type-transact-sql?view=sql-server-ver15)  

[dapper](https://github.com/DapperLib/Dapper)  