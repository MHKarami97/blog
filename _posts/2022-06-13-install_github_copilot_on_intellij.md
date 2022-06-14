---
title: "نصب Github Copilot در Jetbrains"
date: 2022-06-13T00:00:00-00:00
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
