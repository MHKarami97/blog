---
title: "اجرا کردن بخشی از کد خارج از Transaction"
date: 2022-11-01T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - transaction
  - suppress
---

فرض کنید در یک تراکنش بانکی نیاز دارید که در صورت به خطا خوردن بخشی از کد، تمام موارد انجام شده هم به حالت قبل برگردند. بطور مثال اگر انتقال پول به خطا خورد موجودی کاربر اولیه دوباره افزایش یابد. در این موارد می‌توانید از Transaction استفاده کنید.  
حال فرض کنید در داخل این عملیات نیاز دارید تا بخشی را خارج از ترنزکشن انجام دهید تا حتی اگر کد به خطا خورد هم انجام شود. در این موارد کافی است یک Transaction جدید باز کنید و حالت آن را `Suppress` قرار دهید:  

```csharp
using (var scope = new TransactionScope(TransactionScopeOption.Required, option, TransactionScopeAsyncFlowOption.Enabled))
{

// Some code

 using (var tx = new TransactionScope(TransactionScopeOption.Suppress))
{
    InsertMoneyAction(ActionState.Active);
    tx.Complete();
}

// Some Code

scope.Complete();
}
```

مورد مهم در کد بالا تفاوت `TransactionScopeOption` و `TransactionScopeAsyncFlowOption` است. هر دو مقدار `Suppress` را دارند اما مورد مورد نیاز ما `TransactionScopeOption.Suppress` است که عمل انجام شده را انجام می‌دهد. در واقع دو کد زیر کاملا متفاوت هستند و یک کار را انجام نمی‌دهند:  

```csharp
 using (var tx = new TransactionScope(TransactionScopeOption.Suppress))
{
    InsertMoneyAction(ActionState.Active);
    tx.Complete();
}
```

```csharp
 using (var tx = new TransactionScope(TransactionScopeAsyncFlowOption.Suppress))
{
    InsertMoneyAction(ActionState.Active);
    tx.Complete();
}
```

[transactionscopeasyncflowoption](https://learn.microsoft.com/en-us/dotnet/api/system.transactions.transactionscopeasyncflowoption?view=net-6.0)  

[](https://learn.microsoft.com/en-us/dotnet/api/system.transactions.transactionscopeoption?view=net-6.0)  