---
title: "استفاده از Transaction متودهای async در Net"
categories:
  - Net
tags:
  - transaction
  - async
  - net
---

یکی از موارد مهم استفاده از ترنزکشن در .Net دقت کردن به استفاده از متود async / await در داخل آن است. بطور مثال کد زیر را در نظر بگیرید:  

```csharp
using (var scope = new TransactionScope(TransactionScopeOption.Required, new TransactionOptions
        {
            IsolationLevel = IsolationLevel.ReadCommitted
        }))
{
    await DoWorkAsync(model);

    scope.Complete();
}
```
در این مواقع ممکن است با خطا زیر مواجه شوید و هیچ اطلاعی از Exception واقعی اتفاق افتاده در متود await نیز ثبت نمی‌شود و فقط با دیباگ متوجه خطا اصلی می‌شوید.  

 > The operation is not valid for the state of the transaction

برای حل این مشکل کافی است از TransactionScopeAsyncFlowOption.Enabled استفاده کنید.  

```csharp
using (var scope = new TransactionScope(TransactionScopeOption.Required, new TransactionOptions
        {
            IsolationLevel = IsolationLevel.ReadCommitted
        }, TransactionScopeAsyncFlowOption.Enabled))
{
    await DoWorkAsync(model);

    scope.Complete();
}
```
در واقع استفاده از await باعث تغییر Thread می‌شود و TransactionScope کامل نمی‌شود. پس حتما پارامتر گفته شده باید پاس داده شود تا مشکل فوق پیش نیاید.  
