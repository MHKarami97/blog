---
title: "پابلیش تغییرات دیتابیسی در سرور توسط SQLProj و SQL Compare"
categories:
  - Sql
tags:
  - sql
  - sql_compare
  - datagrip
  - publish
  - sql_proj
---

برای ایجاد پروژه دیتابیسی می‌توانید از لینک زیر کمک بگیرید:  

[sqlproj]([/assets/img/sql-publish-1.jpg](https://learn.microsoft.com/en-us/sql/tools/sql-database-projects/tutorials/create-deploy-sql-project?view=sql-server-ver16&pivots=sq1-visual-studio))  

در ادامه فرض گرفته شده است که شما پروژه دیتابیسی دارید و اکنون نیاز به اعمال تغییرات جدید و پابلیش آن دارید.  


در اولین مرحله تغییرات خود را در پروژه اعمال کنید.  
دقت کنید که اگر چند پروژه دیتابیسی دارید و کوئری شما از هر دو استفاده می‌کند طبق لینک زیر می‌توانید به آن رفرنس بدهید و سپس در کد خود بطور مثال با [$(ArchiveDatabase)] به آن دسترسی داشته باشید.  

[SQLCMD]([/assets/img/sql-publish-1.jpg](https://learn.microsoft.com/en-us/sql/tools/sql-database-projects/concepts/database-references?view=sql-server-ver16&pivots=sq1-visual-studio))  

بعد از انجام تغییرات برای اطمینان از عدم وجود خطا یکبار پروژه خود را بیلد کنید.  
در صورتی که خطایی در پروژه باشد در این بخش نمایش داده می‌شود. دقت کنید که نیاز است حتما هم Error و هم warning برطرف شود.
 
![mhkarami97](/assets/img/sql-publish-1.jpg)  
![mhkarami97](/assets/img/sql-publish-2.jpg)  

برای پابلیش کافی است بر روی پروژه خود راست کلیک کنید و گزینه Publish را انتخاب کنید.  
روی Publish بر روی دیتابیس لوکال شما پابلیش می‌شود.  
دقت کنید که اگر فایلی از دیتابیس بطور مثال SP پاک شده باشد در این مورد اعمال نمی‌شود. بدین منظور یکبار دیتابیس لوکال خود را پاک کنید و عملیات پابلیش را یکبار دیگر انجام دهید.  
اگر مشکلی در کوئری‌ها باشد این بخش خطا نشان داده می‌شود. در غیر این صورت تیک سبز به معنی موفقیت آمیز بودن پابلیش است.  

![mhkarami97](/assets/img/sql-publish-3.jpg)  

در صورتی که با خطا مواجه شد کافی است بر روی View Result کلیک کنید. در بخش باز شده می‌توانید خطا را مشاهده کنید.  

![mhkarami97](/assets/img/sql-publish-4.jpg)  
![mhkarami97](/assets/img/sql-publish-5.jpg)  

برای بدست آوردن تغییرات در این مرحله نیاز به استفاده از SQL Compare است.  
برای این کار کافی است نرم افزار RedGate را دانلود کنید. سپس SQL Compare را از آن نصب کنید. دقت کنید برای کرک آن نیاز است اینترنت خود را قطع کنید و سپس با Serial آن را فعال کنید.  
بعد از باز کردن کافی است آن را مانند زیر کامل کنید. همچنین به تیک‌های زده شده نیز دقت کنید.  

![mhkarami97](/assets/img/sql-publish-6.jpg)  
  
همچنین در سربرگ Option تیک گزینه‌های زیر را بزنید.  

![mhkarami97](/assets/img/sql-publish-7.jpg)  
![mhkarami97](/assets/img/sql-publish-8.jpg)  

اکنون بر روی OK کلیک کنید تا مقایسه انجام شود. در این بخش از سربرگ File می‌توانید این تنظیمات را ذخیره کنید تا هردفعه نیاز به وارد کردن آنها نباشد.  
اکنون از بخش‌های نشان داده شده که به 4 قسمت (موجود در هر دو ولی متفاوت، فقط موجود در سورس، فقط موجود در مقصد، مشابه) تقسیم شده موارد مورد نیاز را انتخاب کنید. دقت کنید که نباید مواردی مانند Role را انتخاب کنید.  

![mhkarami97](/assets/img/sql-publish-8-1.jpg)  

اکنون بر روی Deploy کلیک کنید. در صفحه باز شده نیاز به تغییری نیست و کافی است بر روی Next کلیک کنید. دقت کنید که گزینه Create انتخاب شده باشد.  

![mhkarami97](/assets/img/sql-publish-9.jpg)  

ممکن است صفحه زیر اگر تغییرات باعث تغییر دیتا بشوند نشان داده شود. این بخش هم نیاز به تغییر ندارد و کافی است بر روی Next کلیک کنید.  

![mhkarami97](/assets/img/sql-publish-10.jpg)  

در صورتی که صفحه زیر نشان داده شد بعد از مطالعه آن کافی است به سربرگ Deployment Script بروید.  

![mhkarami97](/assets/img/sql-publish-11.jpg)  

در صفحه باز شده بر روی Save Script کلیک کنید و سپس می‌توانید این کوئری را برای اعمال تغییرات ارائه کنید.  

![mhkarami97](/assets/img/sql-publish-12.jpg)  

کوئری ساخته شده Transactional است و بصورت همه یا هیچ کدام اجرا می‌شود.  