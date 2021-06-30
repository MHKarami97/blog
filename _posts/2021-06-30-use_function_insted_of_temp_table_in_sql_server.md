---
title: "بهبود عملکرد استفاده از Temp Table با استفاده از Function در دیتابیس"
date: 2021-06-30T09:32:00-00:00
categories:
  - SQL
tags:
  - sql
  - stored_procedure
  - function
  - temporary_table
---

گاهی مواقع در SP های نوشته شده شما نیاز به دریافت یک لیست دارید. بطور مثال میخواهید فقط مطالبی که در چند دسته بندی خاص هستند را دریافت کنید.  
برای این کار میتوانید شناسه دسته های مورد نظر را با یک کاراکتر بطور مثال (,) از هم جدا کرده و در قالب یک string ارسال کنید.  
برای اینکه در SP دوباره متن ارسال شده را به را به شناسه های مورد نظر تبدیل کنید، راحت ترین راه استفاده از TempTable میباشد.  
برای این کار میتوان بصورت زیر عمل کرد:  


```sql
CREATE TABLE #TmpTable
	(
		Id INT NOT NULL
	);

IF (@MarketIds IS NOT NULL)
	BEGIN
		INSERT INTO #TmpTable
           SELECT value
           FROM STRING_SPLIT(@CustomId, ',');
     END
```

و در قسمت شرط کوئری SELECT کد زیر را وارد کرد:  

```sql
WHERE (@MarketIds IS NULL OR (S.MarketId IN (SELECT Id
                                             FROM #TmpTable)));
```

اما روش بالا بصورت بهنیه نمیباشد و کمی سربار اضافه دارد.  
روش بهینه استفاده از یک function بصورت زیر میباشد که توسط خود دیتابیس کوئری پلن آن مدیریت میشود و بصورت بهینه اجرا میشود.  

```sql
CREATE FUNCTION [dbo].[FnStringToTable](
    @InputString Varchar(max),
    @SeparatedChar CHAR(1)
)
    RETURNS @Table TABLE
                   (
                       ColumnData VARCHAR(100)
                   )
AS
BEGIN
    IF RIGHT(@InputString, 1) <> @SeparatedChar
        SELECT @InputString = @InputString + @SeparatedChar;

    DECLARE @Pos BIGINT,
            @OldPos BIGINT
       SELECT @Pos = 1,
           @OldPos = 1;

    WHILE @Pos < LEN(@InputString)
        BEGIN
            SELECT @Pos = CHARINDEX(@SeparatedChar, @InputString, @OldPos);

            INSERT INTO @Table
            SELECT LTRIM(RTRIM(SUBSTRING(@InputString, @OldPos, @Pos - @OldPos))) Col001;

            SELECT @OldPos = @Pos + 1;
        END

    RETURN
END
GO
```

برای استفاده از این کد نیز کافی است بصورت زیر عمل کرد:  


```sql
 SELECT * FROM
	dbo.MyTable AS m WITH (NOLOCK) LEFT JOIN
     dbo.FnStringToTable(@CustomId, ',') AS s ON m.CategoryId = s.ColumnData
```

لازم به ذکر است که در SQL نیاز به تغییر نوع متغیر نمیباشد. بطور مثال در کدهای بالا CategoryId از نوع INT میباشد اما خروجی فانکشن از نوع VARCHAR اما کد بدرستی کار میکند.
