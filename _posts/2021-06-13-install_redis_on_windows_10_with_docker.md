---
title: "نصب redis در ویندوز 10 توسط docker"
date: 2021-06-13T21:49:00-00:00
categories:
  - Docker
tags:
  - redis
  - docker
  - windows
---

برای نصب و استفاده از دیتابیس Redis در ویندوز راه های مختلفی وجود داره که راحت ترین اونها استفاده از داکر هستش  
دلیل این کار هم این هست که ردیس بصورت مستقیم فایل قابل نصب بر روی ویندوز نداره و فقط یه نسخه قدیمی از اون رو میشه بطور مستقیم نصب کرد  
پس برای استفاده از ورژن های جدیدتر که روی لینوکس قابل نصب هستن، دوتا راه کلی هست:  

 - استفاده از داکر
 - استفاده از wsl

توی این آموزش روش اول رو دنبال میکنیم.  
برای این کار قبل از هر چیز نیاز به نصب کردن خود داکر هست که طبق راهنما سایت خودش میتونید پیش برید:  

[https://docs.docker.com/docker-for-windows/install](https://docs.docker.com/docker-for-windows/install/)  

حالا نیاز به نصب کانتینر ردیس هست که از لینک زیر میتونید اون رو نصب کنید:  

[https://hub.docker.com/_/redis](https://hub.docker.com/_/redis)  

بعد از نصب نوبت به اجرا میرسه، برای این کار داخل ترمینال ویندوز این دستور رو وارد کنید:  

```c#
docker run -d -p 6379:6379 redis
```

این دستور بعد از اجرا پورت 6379 از سیستم شما رو به پورت 6379 از ایمیج مورد نظر bind میکنه  
پس به راحتی میتونید از شبکه یا برنامه های لوکال خودتون بهش دسترسی داشته باشید  

اگه نیاز به استفاده از ردیس در .net داشتین، میتونید از پکیج زیر استفاده کنید:  

[https://github.com/StackExchange/StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis)  

برای مانیتور کردن دیتابیس هم میتونید از ابزار زیر استفاده کنید:  

[https://github.com/junegunn/redis-stat](https://github.com/junegunn/redis-stat)  

ابزار زیر هم برای مدیریت ردیس در دسترس هست:  

[https://github.com/uglide/RedisDesktopManager](https://github.com/uglide/RedisDesktopManager)  

