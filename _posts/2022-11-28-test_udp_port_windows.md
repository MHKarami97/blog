---
title: "تست باز بودن پورت UDP و TCP در ویندوز"
categories:
  - Windows
tags:
  - port
  - udp
  - tcp
---

برای تست در دسترس بودن یک Port خاص توسط سرور دیگر آسان‌ترین راه استفاده از دستور `telnet` در CMD است که بصورت زیر می‌توانید از آن استفاده کنید:  

```s
telnet 172.22.2.1 1233
```

که در آن بخش اول آدرس سرور و بخش دوم پورت مورد نظر است.  
دقت کنید که نیاز به هیچ کاراکتر اضافه بین آنها نیست.  

توسط دستور بالا فقط اتصال `TCP` تست می‌شود و راه مستقیمی برای تست `UDP` نیست. برای تست باز بودن UDP می‌توانید از ابزار `PortQry` که توسط خود Microsoft ارائه شده است استفاده کنید.  
این ابزار دارای UI است و کافی است آن را در سرور اجرا کنید و سپس نوع TCP/UDP/BOTH را انتخاب کنید تا دسترسی مورد نظر را بررسی کنید.  

[PortQry](https://www.microsoft.com/en-us/download/details.aspx?id=17148)  
[portqry-command-line-port-scanner-v2](https://learn.microsoft.com/en-US/troubleshoot/windows-server/networking/portqry-command-line-port-scanner-v2)  
