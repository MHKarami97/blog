---
title: "تغییر تمام فایل‌های LF به CRLF"
categories:
  - Trick
tags:
  - crlf
  - lf
  - unix
  - windows
---

اگر می‌خواستید فایل‌های خود را از LF به CRLF و یا برعکس تغییر دهید می‌توانید از دستورات زیر استفاده کنید:  

ابتدا در پوشه‌ای که می‌خواهید تغییرات را انجام دهید توسط cmd و دستور `git init` یک پروژه گیت ایجاد کنید.  
سپس تمام فایل‌ها را commit کنید.  
دقت کنید که اگر از قبل فایل‌ها در گیت موجود بودند هم نباید هیچ تغییر بازی داشته باشید و باید همه تغییرات را کامیت کنید.  

اکنون در پوشه اول پروژه cmd را باز کنید و دستور زیر را وارد کنید.  

```shell
git config core.autocrlf false
```
اگر می‌خواستید از LF به CRLF بروید دستور بالا باید false باشد.  
اگر می‌خواستید از CRLF به LF بروید دستور بالا باید true باشد.  

اکنون دو دستور زیر را به ترتیب وارد کنید تا تمام فایل‌های شما تغییر کنید.  

```shell
git rm --cached -r .
```

در دستور بالا به . دقت کنید.

```shell
git reset --hard
```


[more info](https://dhwaneetbhatt.com/blog/why-crlf-vs-lf/#:~:text=This%20issue%20arises%20because%20Windows,machines%20as%20their%20console%20devices.)  