---
title: "دیباگ کردن یک برنامه پابلیش شده توسط Attach To Debug"
date: 2022-03-08T12:00:00-00:00
categories:
  - Net
tags:
  - net
  - debug
  - publish
  - attach_process
---

فرض کنید برنامه‌ای که نوشته‌اید در محیط لوکال به درستی کار می‌کند و فقط زمانی که آن را پابلیش می‌دهید، در زمان اجرا با خطا مواجه می‌شود.  
یکی از قابلیت‌های خوب .net و Visual Studio قابلیت دیباگ برنامه‌ها و Process ها است که می‌توانید آنها را دیباگ کنید.  
بدین منظور از محیط Visual Studio از سربرگ Debug گزینه Attach To Process را انتخاب کنید.  
سپس تیک گزینه Show Processes fpr all users را انتخاب کنید.  
اکنون از لیست برنامه های سرویس مورد نظر خود را انتخاب کنید، بطور مثال w3wp.exe برای IIS است.  

<p align="center" >
  <img src="/assets/img/attach_to_debug.jpg" alt="mhkarami97" width="600" />
</p>

البته حتما باید Visual Studio را بصورت Admin اجرا کنید. در غیر این صورت با پیام زیر روبرو می‌شوید که باید بر روی Restart کلیک کنید.  

<p align="center" >
  <img src="/assets/img/attach_to_debug2.jpg" alt="mhkarami97" width="600" />
</p>

اکنون بر روی Attach کلیک کنید.  

<p align="center" >
  <img src="/assets/img/attach_to_debug3.jpg" alt="mhkarami97" width="600" />
</p>

با اینکار در هر نقطه از کد که Break Point بزارید می‌توانید آن را دیباگ کنید.  
البته دقت کنید قابلیت دیباگ در نسخه پابلیش شده وجود داشته باشد. در غیر این صورت Break Point ها غیر فعال می‌شوند.  

لینک‌های مفید:  

[attach-to-running-processes-with-the-visual-studio-debugger](https://docs.microsoft.com/en-us/visualstudio/debugger/attach-to-running-processes-with-the-visual-studio-debugger?view=vs-2022)  

[troubleshooting-breakpoints](https://docs.microsoft.com/en-us/visualstudio/debugger/troubleshooting-breakpoints?view=vs-2022)  

[remote-debugging-aspnet-on-a-remote-iis](https://docs.microsoft.com/en-us/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-7-5-computer?view=vs-2022)  

[how-to-enable-debugging-for-aspnet-applications](https://docs.microsoft.com/en-us/visualstudio/debugger/how-to-enable-debugging-for-aspnet-applications?view=vs-2022)  