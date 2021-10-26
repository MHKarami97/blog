---
title: "انتقال یک کامیت به برنچ دیگر در Git با Cherry Pick"
date: 2021-10-18T11:10:00-00:00
categories:
  - Git
tags:
  - cherry-pick
  - git
  - jetbrains
---

در بعضی مواقع شما نیاز دارید که فقط یک Commit را از یک Branch به یک Branch دیگر منتقل کنید و نمی‌خواهید تمام تغییرات آن برنچ را با برنچ خود Merge کنید.  
برای این مواقع امکان خوبی به اسم Cherry Pick در Git وجود دارد که این امکان را به شما می‌دهد که یک تک کامیت را بر روی برنچ خود اعمال کنید.  
برای این کار از دستورات گیت در cmd نیز می‌توانید استفاده کنید ولی در این آموزش ما از IDE Rider شرکت Jetbrains استفاده می‌کنیم.  

برای این کار ابتدا برنچ مقصد خود که می‌خواهید تغییر را به آن منتقل کنید را با گزینه CheckOut انتخاب کنید.  

<p align="center" >
  <img src="/assets/img/cherryPick1.png" alt="mhkarami97" width="600" />
</p>

اکنون از قسمت پایین چپ و یا کلید `Alt+F9` بخش `Git` را باز کنید و سپس بر روی کامیتی که می‌خواهید راست کلیک کنید و گزینه `Cherry Pick` را انتخاب کنید.  

<p align="center" >
  <img src="/assets/img/cherryPick2.png" alt="mhkarami97" width="600" />
</p>

اکنون اگر به بخش Push تغییرات بروید، کامیت هایی که انتخاب کرده اید را مشاهده می‌کنید.  اگر می‌خواهید دوباره تغییری بر روی این کامیت ها بدهید، کافی است گزینه Cancel را انتخاب کنید تا تغییرات به بخش ChangeList ها اضافه شود.  

<p align="center" >
  <img src="/assets/img/cherryPick3.png" alt="mhkarami97" width="600" />
</p>

اطلاعات بیشتر:  

[jetbrains](https://www.jetbrains.com/help/rider/Apply_changes_from_one_branch_to_another.html#cherry-pick)  

[git](https://git-scm.com/docs/git-cherry-pick)  