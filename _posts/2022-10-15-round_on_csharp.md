---
title: "عملکرد Round در سی شارپ"
date: 2022-10-15T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - round
  - decimal
---

یکی از نکاتی که در زبان C# در زمان استفاده از `Round` باید دقت کنید، تفاوت عملکرد اعدادی که به 5 ختم می‌شوند با دیگر زبان‌ها یا Excel است.  
بطور مثال در اکسل عدد 3.35 همیشه به 3.30 رند می‌شود اما در سی شارپ این حالت متفاوت است و در حالت پیش‌فرض عدد 2.35 به 2.40 و عدد 2.45 به 2.40 رند می‌شود. در واقع عدد به نزدیک‌ترین عدد زوج رند می‌شود که اگر به این دقت نکنید ممکن است باعث اشتباه در برنامه بشود.  

این تابع 3 ورودی می‌تواند بگیرد که اولی همان عدد مورد نظر، دومی به معنی رند شدن تا رقم چندم و سومی همان چگونگی عملکرد در مواقع گفته شده است.  

```c#
3.4 <    Math.Round(3.45, 1, MidpointRounding.ToEven)
3.4 <    Math.Round(3.47, 1, MidpointRounding.ToZero)
3.5 <    Math.Round(3.45, 1, MidpointRounding.AwayFromZero)

-3.4 <   Math.Round(-3.45, 1, MidpointRounding.ToEven)
-3.4 <   Math.Round(-3.47, 1, MidpointRounding.ToZero)
-3.5 <   Math.Round(-3.45, 1, MidpointRounding.AwayFromZero)
```


>> AwayFromZero  > 	1	 >  The strategy of rounding to the nearest number, and when a number is halfway between two others, it's rounded toward the nearest number that's away from zero.

>> ToEven  > 	0	 >  The strategy of rounding to the nearest number, and when a number is halfway between two others, it's rounded toward the nearest even number.

>> ToNegativeInfinity  > 	3	 >  The strategy of downwards-directed rounding, with the result closest to and no greater than the infinitely precise result.

>> ToPositiveInfinity  > 	4	 >  The strategy of upwards-directed rounding, with the result closest to and no less than the infinitely precise result.

>> ToZero  > 	2	 >  The strategy of directed rounding toward zero, with the result closest to and no greater in magnitude than the infinitely precise result.

در لینک‌هایی زیر می‌توانید جزئیات کامل را مشاهده کنید:  

[round](https://learn.microsoft.com/en-us/dotnet/api/system.math.round?view=net-6.0)  

[midpointrounding](https://learn.microsoft.com/en-us/dotnet/api/system.midpointrounding?view=net-6.0)  