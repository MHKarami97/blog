---
title: "تغییر ظاهر صفحه ورود در اوبونتو لینوکس"
date: 2021-04-03T10:23:00-00:00
categories:
  - Linux
tags:
  - linux
  - login
  - purple
  - ubuntu
---

اگه میخواستید برای صفحه ورود لینوکس که بطور پیشفرض در اوبونتو رنگ بنفش داره، عکس یا رنگ دیگه ای قرار بدید، میتونید آموزش زیر رو دنبال کنید
<br />
اول از همه پکیج زیر رو نصب کنید

```shell
wget -qO - https://github.com/PRATAP-KUMAR/focalgdm3/archive/TrailRun.tar.gz | tar zx --strip-components=1 focalgdm3-TrailRun/focalgdm3

sudo apt install libglib2.0-dev
```

برای قرار دادن عکس میتونید از دستور زیر استفاده کنید

```shell
sudo ./focalgdm3 /absolute/path/to/image
```

اگر هم میخواستید رنگ دلخواه بزارید میتونید از دستور زیر استفاده کنید که قسمت #282a36 رنگ شما هست


```shell
sudo ./focalgdm3 \#282a36
```
