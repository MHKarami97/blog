---
title: "قرار دادن دامنه شخصی بروی صفحه شخصی گیتهاب بدون نیاز به هاست"
categories:
  - Github
tags:
  - github
  - github_page
  - domain
  - cloud
---

یکی از امکاناتی که گیتهاب در اختیار برنامه نویسان قرار میده، امکان ایجاد یک سایت رایگان با دامنه شخصی هستش، بطور مثال:
<br />
mhkarami97.github.io
<br />
که قسمت اول اون، یوزرنیم شما در گیتهاب هستش
<br />
از امکانات خوب دیگه، امکان قرار دادن دامنه شخصی بر روی این سایت هست. در این پست آموزش قرار دادن این دامنه رو بدون نیاز به داشتن هاست برای تنظیم دامنه آموزش میدم
<br />
اگر هم میخواید چگونگی ساخت دامنه شخصی رو از پایه یاد بگیرید، میتونید از آموزش زیر استفاده کنید
<br />
[Apart](https://www.aparat.com/v/wYdgL)  
<br />
<br />
در ابتدا نیاز به خرید دامنه هست، من برای این کار از دامنه ir استفاده کردم که البته شما میتونید از هر پسوندی استفاده کنید
<br />
برای تنظیم دامنه به سایت گیتهاب نیاز به تنظیم cname بر روی دامنه هستش که این امکان صرف داشتن دامنه محیا نیست ولی در ادامه روشی رو میگم که این کار رو انجام بدید و نیازی به خرید هاست هم برای تنظیم مورد گفته شده نباشه
<br />
[داکیومنت گیتهاب در مورد موارد گفته شده](https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)
<br />
برای تنظیم رکورد بر روی دامنه بدون نیاز به هاست من از (ابرآوران) استفاده کردم، البته از ClouadFlare هم میشه استفاده کرد.
<br />
برای این کار در ابتدا نیاز هست تا شناسه های ابرآوران رو برروی دامنه قرار بدید

<p align="center" >
  <img src="https://i.postimg.cc/wxbTBV26/Screenshot-2021-02-19-221411-min.jpg" alt="mhkarami97" width="300" />
</p>

بعد از این کار دامنه رو داخل ابرآوران ثبت کنید و چند ساعت صبر کنید تا وضعیتش به (فعال) تغییر کنه

<p align="center" >
  <img src="https://i.postimg.cc/cCjs5xCK/Screenshot-2021-02-19-221305-min.jpg" alt="mhkarami97" width="300" />
</p>

بعد از فعال شدن دامنه باید رکورد ها رو بر روی دامنه ثبت کرد
<br />
اولین رکورد cname هست که عنوان www و مقدار اون هم دامنه شما داخل گیتهاب هستش
<br />
بعد باید رکوردهای A رو ثبت کرد که عنوانش @ و مقدارش برابر با IP های گیتهاب هست که داخل لینک داکیومنت بالا هم اعلام شده
<br />
نتیجه:

<p align="center" >
  <img src="https://i.postimg.cc/pXhV5FcK/Screenshot-2021-02-19-221435-min.jpg" alt="mhkarami97" width="300" />
</p>

بعد از انجام موارد بالا نیاز هست تا دامنه بر روی صفحه گیتهاب هم قرار داده بشه.
<br />
پس به قسمت Setting صفحه مورد نظر در گیتهاب برید و دامنه رو در قسمت Custom Domain وارد کنید

<p align="center" >
  <img src="https://i.postimg.cc/ZRzHhpBZ/Screenshot-2021-02-19-223351-min.jpg" alt="mhkarami97" width="300" />
</p>

بعد از چند ساعت از انجام تمام موارد گفته شده، سایت شما با دامنه ای که وارد کردید بالا میاد
<br />
یکی از خوبی های استفاده از زیرساخت ابری، امکانات بیشتری هست که در اختیار شما میزاره
<br />
بطور مثلا امکان فعال کردن https بر روی دامنه های ir وجود نداشت، ولی از قسمت آروان کلود براحتی میشه این گواهی رو بصورت رایگان فعال کرد
<br />
امکان خوب دیگه قالبیت مانیتور کامل بازدید کننده ها هست که جزئیات خیلی خوبی رو در اختیار شما میزاره
