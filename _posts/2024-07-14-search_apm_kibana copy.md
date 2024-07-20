---
title: "جستجو و فیلتر APM در Kibana"
categories:
  - Trick
tags:
  - kibana
  - apm
  - elastic
  - filter
---

از ابزارهای خوبی که برای Trace سیستم وجود دارد می‌توان به APM اشاره کرد. یکی از مشکلاتی که UI این ابزار در Kibana وجود دارد، کامل نبودن بخش Search آن است. بطور مثال اگر یک Span به خطا خورده باشد نمی‌توان آن را با بخش جستجو پیدا کرد و یا اگر یک Tag دلخواه اضافه کرده باشید و آن تگ در اولین Span نباشد امکان جستجو آن وجود ندارد.  
برای حل این مشکل می‌توان به صورت زیر عمل کرد:  
برای پیدا کردن کامل Transaction یک خطا ابتدا به بخش Error بروید و در بخش Metadata خطا خود مقدار trace.id را بدست آورید.  

![mhkarami97](/assets/img/apmSearch.jpg)  

سپس کافی است از بخش Services سمت راست دوباره به سیستمی که خطا به آن است بروید و به سربرگ Transactions بروید.  
اکنون اگر بصورت زیر مقدار پیدا شده را جستجو کنید Trace کامل مورد به خطا خورده نمایش داده می‌شود.  

```csharp
trace.id : "7207b815e8c77695eb66dfbff3bb9aae"
```

![mhkarami97](/assets/img/apmSearch1.jpg)  

برای پیدا کردن Trace کاملی که دارای تگ دلخواه شما است نیز می‌توانید بصورت زیر عمل کنید.  
به بخش Discover بروید و نام تگ خود را جستجو کنید:  

```csharp
labels.Order_ChangeAssetCustomerCode:  "4"
```

![mhkarami97](/assets/img/apmSearch2.jpg)  

اکنون آن را باز کنید و مقدار trace.id را پیدا کنید.  

![mhkarami97](/assets/img/apmSearch3.jpg)  

اکنون کافی است مانند بخش اول آن را در بخش سرویس خود جستجو کنید تا به یک Trace کامل برسید.  

دقت کنید که برای جستجو حتما از "" استفاده کنید نه ''