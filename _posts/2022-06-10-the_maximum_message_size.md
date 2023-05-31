---
title: "خطا WCF Max message size exceeded در WCF"
categories:
  - Net
tags:
  - net
  - wcf
  - wcfTestClient
  - maxReceivedMessageSize
---

در تکنولوژی‌های قدیمی مانند wcf اگر بخواهید به یک متود که خروجی آن زیاد است ریکوست بزنید، با خطا زیر مواجه می‌شود.  

> The maximum message size quota for incoming messages (65536) has been exceeded. To increase the quota, use the MaxReceivedMessageSize property on the appropriate binding element.

برای حل کردن این مشکل در WCF Test Client کافی است بعد از اضافه کردن آدرس خود بر روی `Config` راست کلیک کنید.  

![mhkarami97](/assets/img/wcfConfig1.jpg)  

سپس گزینه `Edit with SvcConfigEditor` را انتخاب کنید.  

![mhkarami97](/assets/img/wcfConfig2.jpg)  

اکنون در صفحه باز شده کافی است به تب `Binding` بروید و موارد مشخص شده را افزایش دهید.  

![mhkarami97](/assets/img/wcfConfig3.jpg)  

اگر با انجام کار بالا مشکل شما حل نشد بر روی همان فایل راست کلیک کنید و اینبار گزینه `Copy Full Path` را بزنید تا آدرس فایل کانفیگ را بدست آورید.  

![mhkarami97](/assets/img/wcfConfig4.jpg)  

سپس فایل مورد نظر را باز کنید و از وجود داشتن موارد زیر بر روی هردو بایندیگ مطمئن شوید.  

![mhkarami97](/assets/img/wcfConfig5.jpg)  

```c#
maxBufferPoolSize="2147483647" maxBufferSize="2147483647" maxReceivedMessageSize="2147483647"
```

اگر باز هم مشکل شما حل نشد از سربرگ Tools گزینه `Options` را انتخاب کنید و تیک گزینه زیر را بردارید.  

![mhkarami97](/assets/img/wcfConfig6.jpg)  

همچنین تیک گزینه زیر را فعال کنید:  

![mhkarami97](/assets/img/wcfConfig7.jpg)  