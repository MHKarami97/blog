---
title: "غیرفعال سازی CORS در Chrome"
categories:
  - Trick
tags:
  - chrome
  - cors
  - edge
---

در محیط تست و سیستم Local اگر نیاز داشتید که `CORS` مرورگر را غیر فعال کنید می‌توانید از دستور زیر برای اجرای آن استفاده کنید:  

> "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disable-web-security  --user-data-dir=~/chromeTemp

در دستور بالا وجود داشتن `--disable-web-security` باعث غیرفعال سازی مورد گفته شده می‌شود.  