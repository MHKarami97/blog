---
title: "فراخوانی کردن WCF در Postman و jmeter"
date: 2021-06-29T10:18:00-00:00
categories:
  - Develop
tags:
  - wcf
  - postman
  - jmeter
---

یک از مشکلاتی که wcf داره، عدم امکان فراخوانی ساده اون داخل مرورگر،شبیه به rest api هست.  
هرچند که تکنولوژی wcf قدیمی شده اما بعضی مواقع نیاز به استفاده از اون هست.  
بطور مثال درگاه های بانکی هنوز از این تکنولوژی استفاده میکنن.  
  
اگر میخواید بر روی این تکنولوژی Performance Test توسط JMeter انجام بدید یا اون رو توسط Postman فراخونی کنید، به سادگی api نیست و باید کارهای زیر رو انجام بدید.  
ابتدا توسط ابزار  WCF Test Client که بعد از نصب Visual Studio داخل محل نصب اون اضافه میشه، سرویس خودتون رو اضافه کنید.

<p align="center" >
  <img src="/assets/img/wcf1.png" alt="mhkarami97" width="600" />
</p>

حالا یکبار بر روی Invoke کلیک کنید و بعد از قسمت پایین بر روی xml کلیک کنید.

<p align="center" >
  <img src="/assets/img/wcf2.png" alt="mhkarami97" width="600" />
</p>

حالا برای فراخونی این سرویس توسط Postman ابتدا آدرس اصلی سرویس خودتون رو اضافه کنید و نوع فراخونی رو هم بر روی Post قرار بدید.  
بعد به قسمت Headers برید و دو آیتم زیر رو اضافه کنید.  


```shell
Content-Type : text/xml
SOAPAction : http://tempuri.org/IService1/GetData
```

<p align="center" >
  <img src="/assets/img/wcf3.png" alt="mhkarami97" width="600" />
</p>

داخل عکس بالا مقدار SOAP Action از عکس دوم بخش Request و آیتم Action بدست اومده

```shell
<Action s:mustUnderstand="1" xmlns="http://schemas.microsoft.com/ws/2005/05/addressing/none">http://tempuri.org/IService1/GetData</Action>
```

حالا به قسمت Body برید و مقدار Request رو از ابزار WCF Test Client داخل اون قرار بدید.  
نوع Body رو هم بر روی Raw و سپس XML قرار بدید  
همچنین بخش Header رو هم از کد کپی شده پاک کنید.  

<p align="center" >
  <img src="/assets/img/wcf4.png" alt="mhkarami97" width="600" />
</p>

حالا با فراخونی این سرویس خروجی مورد نظر خودتون رو مشاهده میکنید.

<p align="center" >
  <img src="/assets/img/wcf5.png" alt="mhkarami97" width="600" />
</p>
