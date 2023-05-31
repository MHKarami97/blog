---
title: "نصب redis در ویندوز 10 توسط docker"
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

[install docker](https://docs.docker.com/docker-for-windows/install/)  

حالا نیاز به نصب کانتینر ردیس هست که از لینک زیر میتونید اون رو نصب کنید:  

[redis](https://hub.docker.com/_/redis)  

بعد از نصب نوبت به اجرا میرسه، برای این کار داخل ترمینال ویندوز این دستور رو وارد کنید:  

```c#
docker run -d -p 6379:6379 redis
```

این دستور بعد از اجرا پورت 6379 از سیستم شما رو به پورت 6379 از ایمیج مورد نظر bind میکنه  
پس به راحتی میتونید از شبکه یا برنامه های لوکال خودتون بهش دسترسی داشته باشید  

اگه نیاز به استفاده از ردیس در .net داشتین، میتونید از پکیج زیر استفاده کنید:  

[StackExchange](https://github.com/StackExchange/StackExchange.Redis)  

برای مانیتور کردن دیتابیس هم میتونید از ابزار زیر استفاده کنید:  

[redis-stat](https://github.com/junegunn/redis-stat)  

ابزار زیر هم برای مدیریت ردیس در دسترس هست:  

[RedisDesktopManager](https://github.com/uglide/RedisDesktopManager)  

[AnotherRedisDesktopManager](https://github.com/qishibo/AnotherRedisDesktopManager)  

اگر نیاز به قرار دادن رمز عبور برای ردیس داشتید میتونید از دستور زیر استفاده کنید:  

```c#
docker run --name redis -d -p 6379:6379 redis redis-server --requirepass "12345"
```

که بخش 12345 رمز عبور شما هستش.  
البته به این نکته هم دقت کنید:  
``
Warning: since Redis is pretty fast an outside user can try up to  
150k passwords per second against a good box. This means that you should  
use a very strong password otherwise it will be very easy to break
``

<p align="center" >
  <img src="https://i.postimg.cc/KzgWWFbq/redis.png" alt="mhkarami97" width="400" />
</p>
