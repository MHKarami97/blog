---
title: "موجیم، که آرامش ما عدم ماست (مهاجرت به لینوکس)(03)(شروع کد نویسی)"
categories:
  - Linux
tags:
  - linux
  - windows
  - sql_server
  - net_core
  - elementary_os
---

اگه قسمت های قبل رو دنبال کرده باشید، میدونید که چندتا مشکل ساده رو حل کردیم و تونستیم سیستم رو بالا بیاریم
<br />
همه چی خوب پیش میرفت و من از همین تعجب میکردم، اما انگار سکوت قبل از طوفان بودش!!!
<br />
من میخواستم با .net core برنامه بنویسم و از دیتابیس sql server استفاده کنم، پس شروع کردم به نصب .net core
<br />

طبق لینک خود مستنداتش میرفتم جلو:

[https://docs.microsoft.com/en-us/dotnet/core/install/linux](https://docs.microsoft.com/en-us/dotnet/core/install/linux)  

تو لینک بالا از همون پکیج منجر snap هم گفته میشه نصب کرد، پس من هم دستوری که گفته بود رو زدم و با موفقیت نصب شد.
<br />
برای برنامه نویسی هم چند ماهی هستش که از Jetbrains Rirder استفاده میکنم و واقعا هم ازش راضی هست. نکته خوبش اینه که توی لینوکس هم میشه نصبش کرد
<br />
یکی از پروژ هایی که داشتم رو با Rider بازش کردم که اینجا اولین مشکل بوجود اومد. dependencies های پروژه نصب نمیشدن و ظاهرا Nuget نمیتونست پکیج ها رو دریافت کنه
<br />
یه مقدار جستجو کردم (البته این یه مقدار اسمش هست و بیشتر کار برد!!) که به نتیجه ای نرسیدم، گفتم بزار ببینم میشه Nuget رو جدا نصب کرد که دیدم بله !!!
<br />
پس طبق مطلب زیر شروع به نصبش کردم:

[https://stackoverflow.com/a/40209368/10233323](https://stackoverflow.com/a/40209368/10233323)  

مشکل اول حل شد و پکیج ها نصب شدن و پروژه هم با موفقیت بیلد شد
<br />
<br />
غول بعدی که منتظرم بود SQl Server هستش
<br />
داکیومنت این قسمت در لینک زیر دردسترس هستش:

[https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup?view=sql-server-ver15](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup?view=sql-server-ver15)  

با دستورهای زیر میتونید sql رو توی لینوکس نصب کنید:

```shell
sudo apt-get update
sudo apt-get install mssql-server
```

بعد باید خطهای زیر رو طبق عکس به فایل .bashrc اضافه کنید

<p align="center" >
  <img src="https://i.postimg.cc/yNLzMXXV/Screenshot-from-2021-03-11-16-27-05-2x-min.png" alt="mhkarami97" width="400" />
</p>

تو عکس بالا از قسمت export تا انتها رو باید اضاف کرد. البته بعضی قسمتهاش برای .net core هست و از اول تا mssql-tools/bin برای sql server هست

```shell
export PATH="$PATH:/opt/mssql-tools/bin
```

یکی از مشکلاتی که بعد از اضافه کردن این قسمت داشتم، خطا دادن terminal بود. یه چند ساعتی وقت گرفت تا بدونم مشکل چیه. اشتباها کلمه export رو به esac چسبونده بودم
<br />
بعد از جستجو متوجه شدم که برای فایل .bashrc یه فایل دیفالت هم توی پوشه rtc هستش. بعد از نگاه کردن به این فایل متوجه اشتباهم شدم 🥱
<br />
بعد از انجام کار بالا توی ترمینال باید دستور زیر رو وارد کنید تا کافنیگ sql server شروع بشه


```shell
sudo /opt/mssql/bin/mssql-conf setup
```

با انجام کار بالا قسمت زیر میاد که من گزینه 2 که نسخه رایگان sql server هست رو انتخاب کردم

<p align="center" >
  <img src="https://i.postimg.cc/G2N6rb72/Screenshot-from-2021-03-11-14-22-35-2x-min.png" alt="mhkarami97" width="400" />
</p>

بعد از انجام کارهای بالا باید دستور sqlcmd توی ترمینال شناخته بشه، پس دستور زیر رو وارد کنید:

```shell
sqlcmd -S localhost -U SA -Q 'select @@VERSION'
```

خب یکی دیگه از مشکلات رو حل کردم و میخواستم برنامه رو Start کنم که مشکلات بعدی بوجود اومد
<br />
تو قسمت بعدی به ادامه شاخ غول شکستن میپردازیم 😀
