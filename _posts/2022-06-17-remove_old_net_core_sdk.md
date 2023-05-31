---
title: "پاک کردن ورژن‌های قدیمی .Net Core SDK"
categories:
  - Net
tags:
  - net
  - sdk
  - net_core
---

با منتشر شدن ورژن‌های جدیدتر .Net Core SDK / Runtime و نصب آنها بر روی سیستم، پس از مدتی حجم زیادی از درایو C اشغال می‌شود که دلیل آن باقی ماندن ورژن‌های قدیمی بر روی سیستم است.  
ابزار `dotnet-core-uninstall` که توسط خود ماکروسافت ارائه شده است این امکان را به شما می‌دهد تا ورژن‌های قدیمی را پاک کنید.  

ابتدا آخرین نسخه را از لینک زیر دانلود و نصب کنید (فایل msi) :  

[cli-lab releases](https://github.com/dotnet/cli-lab/releases)  

اکنون می‌توانید با دستور زیر جزئیات ورژن‌های نصب شده را بدست بیاورید:  

```s
dotnet-core-uninstall list
```

![mhkarami97](/assets/img/uninstallTools.jpg)  

برای پاک کردن هم دستورات مختلفی وجود دارد که در لینک آخر می‌توانید آنها را مشاهده کنید.  
دستورهای زیر هم برای پاک کردن تمام ورژن‌ها بجز ورژن آخر است:  

![mhkarami97](/assets/img/uninstallTools1.jpg)  

```s
dotnet-core-uninstall remove --all-below 6.0.301 --sdk
```

```s
dotnet-core-uninstall remove --all-below 2.1.30 --runtime
```

```s
dotnet-core-uninstall remove --all-below 6.0.6 --aspnet-runtime
```

لینک‌ها:  

[cli-lab](https://github.com/dotnet/cli-lab)  

[uninstall-tool](https://docs.microsoft.com/en-us/dotnet/core/additional-tools/uninstall-tool)  