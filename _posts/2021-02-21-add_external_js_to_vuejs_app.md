---
title: "استفاده از فایل js خارجی در پروژه Vuejs"
date: 2021-02-21T10:00:00-00:00
categories:
  - Vuejs
tags:
  - vuejs
  - external_js
  - ui
  - javascript
---

در یکی از پروژه هایی که با فریم ورک Vuejs در حال نوشتن بودم و نیاز بود تا یک تم html به vue تبدیل شود، نیاز داشتم تا فایل های js قالب رو داخل پروژه استفاده کنم
<br />
اولین راهی که به ذهنم رسید، نصب کردن هر کدوم از فایهای js توسط npm بود، اما مشکلی که بود، این بود که تموم فایل ها کتابخونه نبودن و طراح قالب بعضی هاش رو نوشته بود
<br />
دومین راه که به ذهنم رسید اضافه کردن فایل ها به قسمت public/index.html بود. بصورت زیر:

```html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <script src="/js/myfile.js"></script>
  </body>
</html>
```

اولین مشکل شناخته نشدن فایل و خطا 404 فایل js در نسخه دولوپ و بعدش پابلیش شده بود
<br />
ابتدا توسط webpack کانفیگ زیر رو به vue.config.js اضافه کردم

```js
    configureWebpack: {
    plugins: [
      new CopyPlugin({
        patterns: [
          {from: 'src/assets/js', to: 'js'}
        ],
      }),
    ]
  }
```

البته بجای کار بالا بصورت دستی هم میشد کل فولدر js رو داخل پوشه public قرار داد اما این کار بنظرم بهتر بود
<br />
برای حل مشکل بعدی اون / رو از اول آدرس پاک کردم تا در نسخه پابلیش شده که در یه base address دیگه بود هم حل بشه

```html
    <script src="js/myfile.js"></script>
```

بعد انجام کارها فایل های js به درستی شناخته میشدن ولی با یه مشکل جدید روبرو شدم
<br />
چون بعضی از توابع فایل های js نیاز داشتن تا سایت کامل لود بشه و بعد روی قالب اعمال بشن، مشکل بهم ریختن قالب رو داشتم و بعضی قسمت ها بدرستی کار نمیکردن
<br />
شبیه این مشکل رو قبلا داخل انگولار هم داشتم، با کمی جستجو هم به نتیجه زیر رسیدم
<br />
باید با یه روشی فایل های js رو بعد از کامل لود شدن صفحه فراخونی میکردم و چون داخل قالب اصلی در آخرین قسمت body بودن، تو این قسمت هم باید آخرین قسمت باشن
<br />
برای این کار در کامپوننت بیس تابع جدید زیر رو نوشتم:

```js
  export default {
  components: {
    "menu-component": Menu,
    "search-component": Search,
    "nav-component": Topnav,
    "footer-component": Footer
  },
  methods: {
    addJs(address) {
      let externalScript = document.createElement('script');
      externalScript.src = address;
      document.body.appendChild(externalScript);
    }
  },
  mounted() {
    let items = [
      "js/jquery.min.js",
      "js/Bootstrap/bootstrap.bundle.min.js",
      "js/js-plugins/navigation.min.js",
      "js/js-plugins/material.min.js",
      "js/js-plugins/swiper.min.js",
      "js/js-plugins/smooth-scroll.min.js",
      "js/js-plugins/jquery.matchHeight.min.js",
      "js/main.js",
      "js/svg-loader.js"
    ];
    for (const i of items) {
      if (document.getElementById(i) != null) {
        document.getElementById(i).remove();
      }
      this.addJs(i);
    }
  },
};
```

در کد بالا تابع mounted برای خود vue هست و تابع addJs هم برای اضافه کردن اسکریپت ها به صفحه
<br />
[تفاوت mounted و created](https://medium.com/@akgarg007/vuejs-created-vs-mounted-life-cycle-hooks-74c522b9ceee)  
<br />
در تابع addJs هم توسط document.body گفته شده که آیتم مورد نظر به قسمت body اضافه بشه
<br />
قبل از نوشتن متود بالا کد رو بصورت زیر نوشته بودم که باعث اشتباه میشد:

```js
  export default {
  components: {
    "menu-component": Menu,
    "search-component": Search,
    "nav-component": Topnav,
    "footer-component": Footer
  },
  mounted() {
    let externalScript = document.createElement('script');     
      document.body.appendChild(externalScript);
      
      externalScript.src = "js/jquery.min.js";
      document.body.appendChild(externalScript);
      
      externalScript.src = "js/js-plugins/navigation.min.js";
      document.body.appendChild(externalScript);
  },
};
```

در کد بالا فقط آخرین js اضافه میشد که اشتباه بود، پس کد رو بصورت کد قسمت قبل تغییر دادم.
<br />
با انجام کارهای گفته شده پروژه به درستی کار کرد و ظاهر هم به هم نریخت
