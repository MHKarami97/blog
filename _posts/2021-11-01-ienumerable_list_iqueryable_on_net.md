---
title: "مقایسه ienumerable و iqueryable و list"
date: 2021-11-01T14:39:00-00:00
categories:
  - Net
tags:
  - net
  - list
  - IQueryable
  - IEnumerable
---

در زبان برنامه نویسی .net کالکشن های مختلفی وجود دارد که دارای کاربردهای متفاوت هستند و استفاده درست از آنها، باعث جلوگیری از به وقوع پیوستن مشکلات در آینده می‌شود.  
سه عدد از معروف ترین این موارد `ienumerable` و `iqueryable` و `list` است که استفاده درست از آنها بخصوص در Repository ها مهم است.  

یکی از تفاوت های مهم ienumerable و list در دو تکه کد زیر بیان شده است.
فرض کنید لیستی با اسم items داریم که می‌خواهیم مواردی از آن که دارای طول بلندتر از 3 هستند را بدست آوریم و سپس آنها را چاپ کنیم.  
اما در بین این موارد یکی از آیتم ها را تغییر می‌دهیم:  

```c#
var items = new List<string> {"aa", "bbbb", "cc", "ddddd", "e"};
var moreItems = items.Where(w => w.Length > 3);
items[0] = "aaaa";

foreach (var item in moreItems)
{
    Console.WriteLine(item);
}
```

در کد بالا نوع متغیر moreItems از نوع `ienumerable` می‌باشد. به همین دلیل فراخوانی آن تا زمان درخواست مقادیر آن به تاخیر می‌افتد.  
پس خروجی آن مقادیر زیر است:  

  - aaaa
  - bbbb
  - ddddd

اما در کد زیر moreItems توسط `.ToList()` به `List` تبدیل شده است و تغییر مقادیر items بر آن اثر نمی‌گذارد.  
پس خروجی آن بصورت زیر است:  

  - bbbb
  - ddddd

```c#
var items = new List<string> {"aa", "bbbb", "cc", "ddddd", "e"};
var moreItems = items.Where(w => w.Length > 3).ToList();
items[0] = "aaaa";

foreach (var item in moreItems)
{
    Console.WriteLine(item);
}
```

---

استفاده درست از انواع Collections ها در زمانی که به دیتابیس کوئری می‌زنید، نقش پررنگ تری دارد.  

بطور مثال در کدهای زیر از 3 کالکشن نام برده شده، استفاده شد است.  
در کد زیر که خروجی آن از نوع `IQueryable` است، دریافت فقط 1 آیتم نیز به کوئری اعمال می‌شود و سپس به دیتابیس کوئری زده می‌شود.  

```c#
IQueryable<MyItems> list = db.MyItem.Where(p => p.Name.StartsWith("T"));
var items = list.Take<MyItems>(1);
return items;
```

اما در کد زیر تمام سطرها که شرط `where` را برقرار می‌کنند، دریافت می‌شوند و سپس فقط آیتم اول آن دریافت می‌شود.  

```c#
IEnumerable<MyItems> list = db.MyItem.Where(p => p.Name.StartsWith("T"));
var items = list.Take<MyItems>(1);
return items;
```

در کد زیر نیز همانند کد بالا عمل می‌شود و تفاوت آن نیز در اوایل پست ذکر شد.  
بهتر است خروجی Repository خود را از نوع List تعیین کنید و همان Enumerable را به لایه های دیگر پاس ندهید.  

```c#
IEnumerable<MyItems> list = db.MyItem.Where(p => p.Name.StartsWith("T"));
var items = list.ToList();
return items;
```

اطلاعات بیشتر درباره کالکشن ها:  

<p align="center" >
  <img src="/assets/img/collections1.png" alt="mhkarami97" width="600" />
</p>

زمان درست استفاده از انواع کالکشن ها:  

<p align="center" >
  <img src="/assets/img/collections2.png" alt="mhkarami97" width="600" />
</p>

اطلاعات بیشتر:  

[iqueryable](https://docs.microsoft.com/en-us/dotnet/api/system.linq.iqueryable?view=net-5.0)  
[ienumerable](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerable?view=net-5.0)  
[list](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1?view=net-5.0)  