---
title: "تابع FIRST_VALUE در SQL"
date: 2021-12-25T13:50:00-00:00
categories:
  - SQL
tags:
  - windows_function
  - sql
  - sql_server
---

این تابع که از نوع `Window Function` است، برای دریافت اولین سطر در حالت `Ordered Partition` استفاده می‌شود.  

بطور مثال فرض کنید یک جدول سفارشات و یک جدول خطاهای سفارشات دارید و می‌خواهید به ازای هر سفارش آخرین خطا را نیز نشان دهید.  
در این حالت می‌توانید این تابع استفاده کنید.  

روش استفاده نیز بصورت زیر است:  

```sql
SELECT DISTINCT od.Id,
                od.CreationDate,
                od.CustomerId,
                od.UserId,
                od.InstrumentId,
                od.Side,
                od.Price,
                od.Quantity,
                od.Status,
                od.Tags,
                COUNT(ode.Id) OVER (PARTITION BY od.Id)                                      AS ErrorCount,
                FIRST_VALUE(ode.Description) OVER (PARTITION BY od.Id ORDER BY ode.Id DESC)  AS ErrorDescription,
                FIRST_VALUE(ode.CreationDate) OVER (PARTITION BY od.Id ORDER BY ode.Id DESC) AS ErrorCreationDate
FROM [trm].[OrderDraft] od
          LEFT JOIN [trm].[OrderDraftError] ode ON ode.OrderDraftId = od.Id
WHERE od.UserId = @UserId
  AND od.Status < 5;
```

در کد بالا توسط `COUNT` تعداد خطاها به ازای هر سفارش و توسط `FIRST_VALUE` آخرین خطا هر سفارش را بدست آورده ایم.  

به موارد PARTITION BY و ORDER BY دقت کنید که اولی با توجه به شناسه جدول سفارشات و دومی بر اساس شناسه جدول خطاها است.  

نمونه مثال دیگر برای دریافت ارزانترین کالا:  

```sql
SELECT Name, ListPrice,   
       FIRST_VALUE(Name) OVER (ORDER BY ListPrice ASC) AS LeastExpensive   
FROM Production.Product  
WHERE ProductSubcategoryId = 10; 
```

نمونه مثال دیگر برای دریافت کارمندان با کمترین ساعت مرخصی با شغل یکسان:  

```sql
SELECT JobTitle, LastName, VacationHours,   
       FIRST_VALUE(LastName) OVER (PARTITION BY JobTitle   
                                   ORDER BY VacationHours ASC  
                                   ROWS UNBOUNDED PRECEDING  
                                  ) AS FewestVacationHours  
FROM HumanResources.Employee AS e  
INNER JOIN Person.Person AS p   
    ON e.BusinessEntityId = p.BusinessEntityId  
ORDER BY JobTitle;  
```

اگر از `PARTITION BY` برای این تابع استفاده نشود، این تابع تمام موارد را در یک پارتیشن حساب می‌کند.  

`اطلاعات بیشتر` : [first value](https://docs.microsoft.com/en-us/sql/t-sql/functions/first-value-transact-sql?view=sql-server-ver15)  