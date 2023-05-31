---
title: "مقایسه ProjectTo و Map در AutoMapper"
categories:
  - Net
tags:
  - net
  - automapper
  - nhibernate
  - Queryable
---

یکی از کتابخانه های کاربردی در .net کتابخانه Automapper است که در زمان های مپ کردن دیتا کاربرد فراوان دارد.  
روش استفاده پیشفرض از این کتابخانه بصورت زیر است که یک مدل را به مدل دیگر تبدیل می‌کند:  

```c#
var result = _mapper.Map<TraderVm>(items);
```

بعضی مواقع مدل شما بصورت IQueryable می‌باشد.  مانند زمان‌هایی که یک دیتا را می‌خواهید از دیتابیس توسط `LINQ` و کتابخانه هایی مانند `Entry FrameWork` ذریافت کنید.  
در این مواقع اگر از کد بالا استفاده کنید، تمام آیتم ها از دیتابیس دریافت می‌شود و سپس بصورت `In Memory` عملیات مپ انجام می‌شود.  
بطور مثال اگر شما می‌خواهید فقط بعضی از ستون های دیتابیس را دریافت کنید، راه ساده استفاده از کد زیر است:  

```c#
result
    .Where(a => a.Title.Contains(title))
    .OrderBy(a => a.Id)
    .Select(a => new SearchDto
    {
        Id = a.Id,
        Title = a.Title
    });
```

اگر از کتابخانه Autommaper و دستور `Map` برای بجای کد بالا استفاده کنید، تمام ستون های دیتابیس دریافت می‌شود و سپس در مموری دو ستون Id, Title برای خروجی انتخاب می‌شود.  

برای جلوگیری از اضافه بار بالا، متود دیگری در این کتابخانه با نام `ProjectTo` وجود دارد که ورودی آن یک لیست از نوع IQueryable است و در حالت کوئری زدن به دیتابیس، بر روی دستور `SELECT` اعمال می‌شود و فقط خروجی هایی مورد نیاز شما است از دیتابیس دریافت می‌شود.  

روش استفاده از این متود نیز بصورت زیر است.  
در بخش قبل از where شما می‌توانید تمام موارد که در حالت عادی برای دستورهای دیتابیس می‌خواهید را بنویسید و در انتها به دستور ProjectTo خروجی مورد نظر خود را از کوئری بخوا‌هید.  

```c#
var result = context.Trader.Where(ol => ol.TraderId == traderId)
                     .ProjectTo<TraderDTO>(configuration).ToList();
```

اطلاعات بیشتر:  

[automapper](https://docs.automapper.org/en/stable/Queryable-Extensions.html)  