---
title: "خطا WCF Max message size exceeded در WCF"
date: 2022-06-11T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - wcf
  - wcfTestClient
  - MaxReceivedMessageSize
---

در تکنولوژی‌های قدیمی مانند wcf اگر بخواهید به یک متود که خروجی آن زیاد است ریکوست بزنید، با خطا زیر مواجه می‌شود.  

> The maximum message size quota for incoming messages (65536) has been exceeded. To increase the quota, use the MaxReceivedMessageSize property on the appropriate binding element.

برای حل کردن این مشکل در WCF Test Client کافی است بعد از اضافه کردن آدرس خود بر روی `Config` راست کلیک کنید.  

<img src="/assets/img/wcfConfig1.png" alt="mhkarami97" />

سپس گزینه `Edit with SvcConfigEditor` را انتخاب کنید.  

<img src="/assets/img/wcfConfig2.png" alt="mhkarami97" />

اکنون در صفحه باز شده کافی است به تب `Binding` بروید و موارد مشخص شده را افزایش دهید.  

<img src="/assets/img/wcfConfig3.png" alt="mhkarami97" />

اگر با انجام کار بالا مشکل شما حل نشد بر روی همان فایل راست کلیک کنید و اینبار گزینه `Copy Full Path` را بزنید تا آدرس فایل کانفیگ را بدست آورید.  

<img src="/assets/img/wcfConfig4.png" alt="mhkarami97" />

سپس فایل مورد نظر را باز کنید و از وجود داشتن موارد زیر بر روی هردو بایندیگ مطمئن شوید.  

<img src="/assets/img/wcfConfig5.png" alt="mhkarami97" />

```c#
maxBufferPoolSize="2147483647" maxBufferSize="2147483647" maxReceivedMessageSize="2147483647"
```

اگر باز هم مشکل شما حل نشد از سربرگ Tools گزینه `Options` را انتخاب کنید و تیک گزینه زیر را بردارید.  

<img src="/assets/img/wcfConfig6.png" alt="mhkarami97" />

همچنین تیک گزینه زیر را فعال کنید:  

<img src="/assets/img/wcfConfig7.png" alt="mhkarami97" />
