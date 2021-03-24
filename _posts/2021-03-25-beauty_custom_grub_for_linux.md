---
title: "استفاده از GRUB شخصی سازی شده و زیبا در حالت Dual Boot"
date: 2021-03-25T01:20:00-00:00
categories:
  - Linux
tags:
  - linux
  - windows
  - grub
---

وقتی از چندتا سیستم عامل استفاده میکنید، در زمان روشن کردن سیستم صفحه ای نشون داده میشه تا بتونید سیستم عامل خودتون رو انتخاب کنید
<br />
بطور پیش فرض این صفحه ظاهر خیلی ساده ای داره اما میشه اون رو تغییر داد
<br />
بطور مثال من از سیستم عامل های Fedora و Windows 10 استفاده میکنم
<br />
برای تغییر این صفحه اول از همه به لینک زیر برید و تم دلخواه خودتون رو دانلود کنید

[https://www.gnome-look.org/browse/cat/109/order/latest](https://www.gnome-look.org/browse/cat/109/order/latest)  

بعد از دانلود اون رو از حالت فشرده خارج کنید، داخل پوشه مورد نظر فایلی با اسم install.sh هستش
<br />
کافیه ترمینال رو در محلی که این فایل هست باز کنید و با دستور زیر اون رو نصب کنید

```shell
sudo ./install.sh
```

<p align="center" >
  <img src="https://i.postimg.cc/gkyMq0RZ/Screenshot-from-2021-03-25-01-00-41-min.png" alt="mhkarami97" width="400" />
</p>

در بیشتر مواقع همین دستور کافی هست ولی اگه صفحه مورد نظر تغییر نکرد ادامه آموزش رو مطالعه کنید
<br />
<br />
برای راحتی کار میتونید از نرم افزار Grub Customizer استفاده کنید که البته با این نرم افزار میتونید تم دلخواه هم طراحی کنید

<p align="center" >
  <img src="https://i.postimg.cc/dQyN36Vn/Screenshot-from-2021-03-25-00-59-51-min.png" alt="mhkarami97" width="400" />
</p>

قبل از هر چیز مطمن بشید که پوشه theme در مسیر زیر موجود باشه

```shell
/boot/grub/themes
```

بعدش توسط نرم افزار گفته شده یا فایل grub در مسیر زیر خط گفته شده رو کامنت یا پاک کنید

```shell
/etc/default/grub
```

```shell
GRUB_TERMINAL_OUTPUT="console"
```

بعد از این کار اگه سیستم شما uefi هست کد زیر رو داخل ترمینال اجرا کنید

```shell
grub2-mkconfig -o /boot/efi/EFI/fedora/grub.cfg
```

با این کار Grub سیستم شما تغییر میکنه
