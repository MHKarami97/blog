---
title: "پاک کردن توسط Func در متود RemoveAll"
categories:
  - Net
tags:
  - net
  - func
  - predicate
  - function
---

فرض کنید نیاز دارید که مقادیر مختلف را از یک List پاک کنید، البته با توجه به حالت‌های مختلف پاک کردن این موارد متفاوت است.  
یکی از راه‌ها استفاده از `Func` است که نمونه آن را در زیر می‌بینید.  
توسط این قابلیت می‌توانید شرط خود را در حالت‌های مختلف تعریف کنید و در انتها از آن استفاده کنید.  

مشکلی که وجود دارد این است که متود `RemoveAll` ورودی از نوع `Predicate` می‌گیرد. برای حل کردن این مشکل نیز می‌توانید شبیه خط آخر بصورت `Func(x)` استفاده کنید.  

```c#
void RemovePrevious(Order currentOrder)
{
    Func<Order, bool> func;
    
    if (currentOrder.AccountType == AccountTypeEnum.Others)
    {
        func = x => x.AccountType == AccountTypeEnum.Others && string.Compare(x.Priority,
            currentOrder.Priority, StringComparison.OrdinalIgnoreCase) <= 0;
    }
    else if (currentOrder.AccountType == AccountTypeEnum.Client)
    {
        func = x => x.AccountType == AccountTypeEnum.Client && string.Compare(x.Priority,
            currentOrder.Priority, StringComparison.OrdinalIgnoreCase) <= 0;
    }
    else
    {
        func = x =>
            string.Compare(x.Priority, currentOrder.Priority, StringComparison.OrdinalIgnoreCase) <= 0;
    }
    
    _orders.RemoveAll(x => func(x));
}
```