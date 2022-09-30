---
title: "اضافه شدن خودکار Schema در DataGrip"
date: 2022-07-16T00:00:00-00:00
categories:
  - Trick
tags:
  - datagrip
  - sql
  - jetbrains
  - schema
---

بصورت پیش‌فرض در حالت `AutoComplete` نرم افزار `DataGrip` اسکیما جدول مورد نظر اضافه نمی‌شود و در صورت زدن کلید Tab فقط جدول مورد نظر اضافه می‌شود.  

```sql
SELECT *
FROM Request
```

در صورتی که چند Schema مختلف در دیتابیس خود داشته باشید این روش باعث مشکل می‌شود و شما مجبور هستید اسکیما را دستی وارد کنید.  
برای حل این مشکل می‌توانید به بخش Setting بروید و شبیه به عکس زیر از بخش `Editor > General > Code Completion` و سپس بخش `Qualify object with` از DropDown مورد نظر گزینه Always را انتخاب کنید.  
با این کار کوئری شبیه به صورت زیر به شما نشان داده می‌شود:  

```sql
SELECT *
FROM [mySchema].Request R
```

<p align="center" >
  <img src="/assets/img/datagripSchema.jpg" alt="mhkarami97" />
</p>