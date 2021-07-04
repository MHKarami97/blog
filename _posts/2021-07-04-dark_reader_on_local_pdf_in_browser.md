---
title: "استفاده از Dark Mode برای فایل های pdf در مرورگر"
date: 2021-07-04T15:52:00-00:00
categories:
  - Trick
tags:
  - pdf
  - chrome
  - edge
  - browser
  - dark_reader
---

یکی از افزونه های خوبی که برای مروگرهای بر پایه Chromium وجود داره، افزونه Dark Reader هستش.  
با استفاده از این افزونه میتونید تم تاریک رو در داخل همه سایت ها استفاده کنید.  
مشکلی که وجود داره، بصورت پیش فرض نمیشه فایل های PDF که داخل سیستم خودتون هست رو با تم تاریک مشاهده کنید.  
برای حل این مشکل کافیه به بخش تنظیمات افزونه برید و گزینه Allow access to file URLs رو فعال کنید.  
حالت افزونه رو هم بر روی Filter+ قرار بدید.  

<p align="center" >
  <img src="/assets/img/darkReader.png" alt="mhkarami97" width="600" />
</p>

برای دانلود افزونه هم از لینک زیر میتونید اقدام کنید:  

[Dark Reader](https://microsoftedge.microsoft.com/addons/detail/dark-reader/ifoakfbpdcdoeenechcleahebpibofpc)  


اگه میخواستید از افزونه استفاده نکنید هم کافیه بعد از باز کردن صفحه مورد نظرتون کلید F12 رو بزنید و در قسمت Console کد زیر رو وارد کنید:  

```css
var cover = document.createElement("div");
let css = `
    position: fixed;
    pointer-events: none;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: white;
    mix-blend-mode: difference;
    z-index: 1;
`
cover.setAttribute("style", css);
document.body.appendChild(cover);
```
