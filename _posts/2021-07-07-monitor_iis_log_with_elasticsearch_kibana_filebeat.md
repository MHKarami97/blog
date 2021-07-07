---
title: "مانیتور کردن لاگ های IIS توسط ElasticSearch و Kibana"
date: 2021-07-04T15:52:00-00:00
categories:
  - IIS
tags:
  - elasticSearch
  - kibana
  - log
  - iis
  - filebeat
---

با بزرگ شدن پروژه و گسترش اون، به تبع اطلاعات بدست اومده بصورت نمایی زیاد میشه و دیگه با ابزارهای قبلی نمیشه این حجم از اطلاعات رو پردازش کرد.  
بطور مثال اگه بخوایم لاگ های جمع شده از یه برنامه رو بررسی کنیم، ابزارهای قدیمی جوابگو نیستن.  
یکی از ابزارهای خیلی مفید و خوب برای این کار Elastic Search هستش.  
این ابزار پنل مدیریت گرافیکی بصورت پیش فرض نداره، پس ابزاری به اسم kibana تولید شده که امکانات گرافیکی رو در اختیار شما میزاره.  

در این مطلب میخوایم لاگ های تولید شده توسط IIS رو با این ابزارها مدیریت کنیم.  
برای این کار ابتدا از نصب بودن Java روی سیستم خودتون مطمئن بشید.  
توسط دستور زیر میشه از ورژن جاوا نصب شده مطمئن شد:  

```shell
java --version
```

برای اینکه مطمئن بشید قابلیت لاگ برای iis فعال هست هم میتونید از قسمت زیر اقدام کنید:  

<p align="center" >
  <img src="/assets/img/iis_log.png" alt="mhkarami97" width="600" />
</p>

[اطلاعات بیشتر درباره لاگ](https://docs.microsoft.com/en-us/iis/manage/provisioning-and-managing-iis/managing-iis-log-file-storage)  

حالا نوبت به نصب elasticsearch میرسه.  
برای این کار کافیه فایل msi رو از لینک زیر دانلود کنید و طبق داکیومنت نصب کنید:  

[elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html) 

اگه همه چیز درست پیش رفته باشه باید آدرس زیر رو ببینید

```shell
http://localhost:9200
```

برای نصب kibana میتویند از لینک زیر فایل zip برای windows رو دانلود کنید.  

[kibana](https://www.elastic.co/downloads/kibana) 

بعد از دانلود اون بطور مثال در درایو C از حالت فشرده خارج کنید و اسم پوشه رو به kibana تغییر بدید.  
حالا cmd رو در پوشه `C:\kibana\bin` باز کنید

با اجرا دستور زیر برنامه شروع به کار میکنه:  

```shell
kibana.bat
```

اگر کانفیگ پیش فرض elasticsearch رو تغییر داده بودید نیاز به تغییر فایل `config/kibana.yml` در kibana هستش.  

اگه همه چیز درست پیش رفته باشه آدرس زیر رو باید مشاهده کنید.

```shell
http://localhost:5601
```

حالا نوبت به نصب ابزار filebeat میرسه.  
برای این کار فایل windows zip رو از آدرس زیر دانلود کنید و اون رو در درایو c از حالت فشرده خارج کنید و اسمش رو به filebeat تغییر بدید.  

[filebeat](https://www.elastic.co/downloads/beats/filebeat) 

حالا powershell رو بصورت ادمین اجرا کنید و به آدرس `C:\filebeat` برید و دستور زیر رو اجرا کنید:  

```shell
.\install-service-filebeat.ps1
```

اگه کانفیگ های پیش فرض elasticsearch رو تغییر داده باشید نیاز به تغییر فایل `filebeat.yml` هستش.  

حالا دستور زیر رو داخل cmd و همون مسیر `C:\filebeat` وارد کنید.  

```shell
filebeat.exe modules enable iis
```

و بعدش دستور زیر رو وارد کنید:  

```shell
.\filebeat.exe setup
```

و بعد دستور زیر رو وارد کنید تا سرویس استارت بشه:  

```shell
Start filebeat
```

اگه همه چی درست پیش رفته باشه باید با کلیک روی کلید Check Data در لینک زیر متن سبز رنگ رو مشاهده کنید.  

[iisLogs](http://localhost:5601/app/home#/tutorial/iisLogs) 

اگه متن پیام سبز رنگ رو مشاهده کردید، با کلیک روی iis log dashboard لینک بالا میتونید پنل مانیتور لاگ ها رو مشاهده کنید.  

همچنین توسط این ابزار میتونید مانیتور ها مدیریت دیتا خیلی از ابزارهای دیگه رو انجام بدید که در لینک زیر راهنما اونها در دسترس هست:  

[tools](http://localhost:5601/app/home#/tutorial_directory) 

<p align="center" >
  <img src="/assets/img/kibana.png" alt="mhkarami97" width="600" />
</p>
