---
title: "اضافه کردن Live Template به DataGrip"
categories:
  - Trick
tags:
  - datagrip
  - template
  - auto_complete
---

یکی از موارد کاربردی در DataGrip بخش Live Template می‌باشد که توسط آن می‌توانید یک کلید خاص تعریف کنید تا با نوشتن آن بقیه موارد بصورت خودکار نوشته شود.  
بطور مثال فرض کنید می‌خواهید 20 سطر آخر یک جدول را بگیرید. برای این کار می‌توانید کلمه ای تهیه کنید تا با نوشتن آن در Editor فقط نیاز باشد نام جدول را وارد کنید و دیگر لازم نباشد یک کوئری SELECT را بصورت کامل بنویسید.  
بدین منظور به آدرس زیر بروید:  

 > Settings - Editor - Live Templates

در این بخش کافی است توسط آیکن + یک مورد جدید با اطلاعات زیر ایجاد کنید:  

 > Abbreviation : topdesc
 > Description : select top 20 order by desc
 > Template Text : select top 20 * from $table$ order by Id desc $END$;
 > Expand With : Tab

در واقع توسط این کار کوئری زیر نوشته می‌شود و بجای $table$ نام جدول را از شما می‌گیرد.  

```sql
select top 20 * from $table$ order by Id desc $END$;
```

برای استفاده نیز کافی است در زمان نوشتن کوئری کلمه کلیدی topdesc را بنویسید و سپس Tab را بزنید تا کوئری نوشته شود.  