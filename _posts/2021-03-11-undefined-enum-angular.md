---
title: "خطا undifiend برای select در انگولار"
date: 2021-03-11T22:55:00-00:00
categories:
  - Angular
tags:
  - angular
  - undifiend
  - html-select
---

آخرین مشکلی که باهاش روبرو شدم و یه مدتی وقتم رو گرفت، مشکلی با این مضمون بود که وقتی توسط تگ select در html مقدارش تو فایل ts نمیومد
<br />
نمونه کدی که استفاده کرده بودم:

```html
<select class="form-control"
                    #RoleName="ngModel"
                    name="Nationality"
                    id="UserInfo_Nationality"
                    [ngModel]="model.roleName">
                <option value="0" selected disabled>انتخاب نقش</option>
                <option value="5">دکتر</option>
                <option value="3">پرستار</option>
                <option value="4">منشی</option>
                <option value="1">مطب</option>
</select>
```

به ظاهر مشکلی در کد نیست و بقیه موارد input به همین صورت درست مقداردهی میشدن
<br />
ولی این مورد که نوعش هم یک enum بود مقدار undifiend میگرفت
<br />
روش حل مشکل آسون بود و بیشتر به این دلیل بودن که روش کارکرد [ngModel] رو فراموش کرده بودم
<br />
این مورد بصورت یک طرفه کار میکنه و نمیشه باهاش مقدار رو در ui تغییر داد و در ts دریافت کرد
<br />
به همین دلیل از [(ngModel)] استفاده کردم تا بشه مقدار رو ارسال کرد

```html
<select class="form-control"
                    #RoleName="ngModel"
                    name="Nationality"
                    id="UserInfo_Nationality"
                    [(ngModel)]="model.roleName">
                <option value="0" selected disabled>انتخاب نقش</option>
                <option value="5">دکتر</option>
                <option value="3">پرستار</option>
                <option value="4">منشی</option>
                <option value="1">مطب</option>
</select>
```
با این کار مشکل حل شد و مقدار متغیر به درستی ارسال شد
