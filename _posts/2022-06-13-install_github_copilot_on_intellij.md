---
title: "نصب Github Copilot در Jetbrains"
categories:
  - IDE
tags:
  - ide
  - intellij
  - github
  - copilot
---

یکی از ابزارهای خوب که جدیدا معرفی شده است و توسط هوش مصنوعی کدزنی را بسیار آسانتر می‌کند، ابزاری به اسم `copilot` است که توسط github ارائه شده که بر پایه AI به شما متودها و کدها را پیشنهاد می‌کند.  

[copilot](https://copilot.github.com/)  

نصب نسخه جدید این ابزار بر روی Rider و دیگر IDE های شرکت Jetbrains با خطا موجه می‌شود و شما در مرحله `waiting for github authentication` می‌مانید.  
در لینک‌های زیر هم درباره این مشکل و همچنین راه‌حل‌های آن بحث شده است. راه حل اول نصب نسخه `1.1.20.1417` بصورت دستی است که این روش در نسخه‌های جدید دیگر جوابگو نیست و افزونه دیگر کار نمی‌کند و پیام آپدیت را به شما نشان می‌دهد.  

[version 1.1.20.1417](https://plugins.jetbrains.com/plugin/17718-github-copilot/versions/stable)  

راه حلی که به درستی کار می‌کند بصورت زیر است:  

```s
Update your github-copilot to the latest version then close the idea

Download this version of github-copilot (1.1.20.1417):
https://plugins.jetbrains.com/plugin/download?rel=true&updateId=172765 and Extract it

Navigate to ...\github-copilot-intellij-1.1.20.1417\github-copilot-intellij\lib

Copy "core-1.1.20" file

Navigate to
    For IntelliJ IDEA
    ...\AppData\Roaming\JetBrains\IdeaIC2022.1\plugins\github-copilot-intellij\lib
    
    For AndroidStudio
    ...\AppData\Roaming\Google\AndroidStudio2021.2\plugins\github-copilot-intellij\lib
    
    For Rider
    C:\Users\______\AppData\Roaming\JetBrains\Rider2022.1\plugins\github-copilot-intellij\lib

Replace "core-1.1.2X" with "core-1.1.20".
```

در واقع شما باید فایل `core-1.1.20` از ورژنی که درست است را در پوشه ی `lib` از این افزونه کپی و جایگزین فایل قبلی کنید.  


[discussions 1](https://github.com/orgs/github-community/discussions/18132)  

[discussions 2](https://github.com/orgs/github-community/discussions/16230#discussioncomment-2750640)  

[discussions 3](https://github.com/axios/axios/issues/3384)  

[discussions 4](https://github.com/github-community/community/discussions/16960)  

[discussions 5](https://github.com/github-community/community/discussions/8333)  

[github-copilot](https://plugins.jetbrains.com/plugin/17718-github-copilot/reviews#review=68155-68157)  

راهنما نصب :  

[gettingstarted](https://github.com/github/copilot-docs/blob/main/docs/jetbrains/gettingstarted.md)  

## حل خطا self signed certificate
خطایی که ممکن است بعد از نصب این افزونه با آن مواجه شوید، خطایی با متن زیر است:  

> Sign in failed. Reason: Request signInInitiate failed with message: self signed certificate in certificate chain, request id: 3, error code: -32603

![mhkarami97](/assets/img/copilate_github06.jpg)  

برای رفع این خطا بصورت دائم کافی است در مرورگر خود سایت github.com را باز کنید و آیکن قفل کنار آدرس سایت را بزنید و سپس به بخش `Connection is secure` بروید.  

![mhkarami97](/assets/img/copilate_github01.jpg)  

در این بخش بر بروی آیکن مشخص شده کلیک کنید:  

![mhkarami97](/assets/img/copilate_github02.jpg)  

در صفحه باز شده به تب دوم که `Details` است بروید و سپس بر روی `Export` کلیک کنید.  

![mhkarami97](/assets/img/copilate_github03.jpg)  

در این بخش از بخش پایین گزینه `Base64-encoded ASCII, certificate chain` را انتخاب کنید و فرمت فایل را هم خودتان از `crt` به `pem` تغییر دهید.  

![mhkarami97](/assets/img/copilate_github04.jpg)  

اکنون در سیستم عامل خود به بخش `Advanced System Settings` و سپس `Environment Variables` بروید و در بخش `System Variable` یک کلید جدید با نام `NODE_EXTRA_CA_CERTS` و مقدار `آدرس فایل مورد نظر` بسازید.  

![mhkarami97](/assets/img/copilate_github05.jpg)  

اکنون اگر یکبار IDE خود را ریست کنید می‌توانید بدون مشکل به افزونه گفته شده وارد شوید و از آن استفاده کنید.  