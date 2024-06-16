---
title: "استفاده از submodules در git"
categories:
  - Github
tags:
  - github
  - git
  - submodules
---

یکی از امکانات جالبی که در git موجود هست که من هم تازه باهاش آشنا شدم، امکانی به اسم Submodules هست.  
این ویژگی به شما این امکان رو میده که کدهای خود رو به پروژه های مختلف تقسیم کنید.  
بطور مثال فرض کنید شما یک کامپوننت نوشتید که در برنامه های مختلف شرکت از اون استفاده میشه. راه ساده ای که وجود داره اینه که اون کامپوننت رو در پروژ های مختلف کپی کنید.  
روشی که این قابلیت در اختیار شما میزاره به این صورت هست که میتونید کامپوننت خودتون رو بصورت جدا گسترش بدید تا هم تاریخچه جدا داشته باشه، هم در صورت آپدیت شدن نیاز نباشه در پروژه های مختلف اون رو بصورت دستی کپی کنید.  
  
برای اضافه کردن Submodules به یک پروژه نیاز هست دستور زیر رو وارد کنیم:  

```shell
git submodule add https://github.com/MHKarami97/Submodules
```

در کد بالا قسمت بعد از `add` آدرس ماژول شما هست که میخواید به پروژه فعلی اضافه کنید.  

<p align="center" >
  <img src="/assets/img/submodules2-min.jpg" alt="mhkarami97" width="600" />
</p>

با این کار پروژه ای که لینکش رو در بالا وارد کردید و همچنین فایل `gitmodules` به پروژه شما اضافه میشه.  
این فایل دوم در واقع یک مپ بین فایل لوکال و ریموت submodules هست.  

<p align="center" >
  <img src="/assets/img/submodules3-min.jpg" alt="mhkarami97" width="600" />
</p>

برای اضافه کردن یک ماژول دیگه هم میتونید به همین صورت عمل کنید:  

<p align="center" >
  <img src="/assets/img/submodules4-min.jpg" alt="mhkarami97" width="600" />
</p>

بعد از اینکار تغییرات گیت برای پروژه اصلی بصورت زیر نشون داده میشه که میتونید کامیت کنید:  

<p align="center" >
  <img src="/assets/img/submodules5-min.jpg" alt="mhkarami97" width="600" />
</p>

حالا اگه تغییری در فایل های ماژول ها بدید، تغییرات اونها هم در پروژه اصلی و هم پروژه های ماژول نشون داده میشه:  

<p align="center" >
  <img src="/assets/img/submodules6-min.jpg" alt="mhkarami97" width="600" />
</p>

تنها نکته ای که هست اینه که برای کامیت اول باید تغییرات رو در ماژول ها کامیت کنید و بعد اجازه کامیت در پروژه اصلی رو دارید.  

اگر یکی از ماژول تغییر کرده باشن، میتونید فقط اون پروژه رو pull کنید:  

<p align="center" >
  <img src="/assets/img/submodules7-min.jpg" alt="mhkarami97" width="600" />
</p>

برای کلون کردن پروژه اصلی که از ماژول های دیگه داخلش استفاده شده، علاوه بر دستور `git clone ...` که فایل های پروژه اصلی رو میگیره، نیاز به اجرای دستور زیر هم هست:  

`git submodule init`  

<p align="center" >
  <img src="/assets/img/submodules8-min.jpg" alt="mhkarami97" width="600" />
</p>

و سپس برای آپدیت کردن و دریافت آخرین نسخه ماژول ها دستور زیر رو وارد کنید:  

`git submodule update`

<p align="center" >
  <img src="/assets/img/submodules9-min.jpg" alt="mhkarami97" width="600" />
</p>

ماژول های در ادیتور Rider :  

<p align="center" >
  <img src="/assets/img/submodules10-min.jpg" alt="mhkarami97" width="600" />
</p>

با این کار پروژه شما آماده استفاده هست.  

داکیومنت اصلی:  

[Git-Tools-Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)  

لینک پروژه های ساخته شده:  

[Submodules](https://github.com/MHKarami97/Submodules)  

[Submodules 2](https://github.com/MHKarami97/Submodules2)  

[Main Submodules](https://github.com/MHKarami97/MainSubmodules)  