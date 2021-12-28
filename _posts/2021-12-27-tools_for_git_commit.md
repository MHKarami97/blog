---
title: "ابزارهای کاربردی Commit در Git"
date: 2021-12-27T14:07:00-00:00
categories:
  - Git
tags:
  - tools
  - format
  - husky
  - commitlint
  - dotnet_format
  - prettier
  - eslint
---

از ابزارهایی که برای مدیریت پروژه استفاده می‌شود، Git است که کاربردهای زیادی دارد.  
در زمان‌هایی که تعداد افرادی که بر روی یک پروژه کار می‌کنند زیاد می‌شود، نیاز به مدیریت کد و کامیت‌ها بیشتر احساس می‌‍شود تا هم خوانایی کدها و هم تاریخچه تغییرات بهتر حفظ شود.  

ابزارهای مختلفی برای این کار وجود دارد که در ادامه آنها را معرفی می‌کنیم.  

### commitlint
این ابزار برای مدیریت نام Commit ها است تا طبق یک فرمت خاص باشند تا خوانایی تاریخچه تغییرات را بهتر کند.  

[commit lint](https://github.com/conventional-changelog/commitlint)  

بطور مثال با دستور زیر کامیت‌ها بصورت کد دوم باید باشند.  

```s
type(scope?): subject  #scope is optional; multiple scopes are supported (current delimiter options: "/", "\" and ",")
```

```s
fix(server): send cors headers
```

از دیگر کاربردهای این ابزار می‌توان به موارد زیر اشاره کرد:  

  - ایجاد خودکار ChangeLogs
  - ایجاد خودکار ورژن‌ها
  - دانستن نوع تغییرات انجام شده در کامیت
  - راه‌اندازی پابلیش خودکار با توجه به نام کامیت


### husky
این ابزار برای انجام خودکار کارها پس از هر کامیت است.  
بطور مثال توسط آن می‌توانید دستور اجرا تست را بعد از هر کامیت اجرا کنید.  

[husky](https://github.com/typicode/husky)  

روش کار نیز بصورت زیر است:  

```s
npx husky add .husky/pre-commit "npm test"
git add .husky/pre-commit
```

```s
git commit -m "Keep calm and commit"

# `npm test` will run every time you commit
```

### prettier
این ابزار برای مرتب سازی کدها بصورت خودکار است که از زبان‌های مختلف برنامه نویسی پشتیبانی می‌کند.  
بطور مثال فرض کنید در شرکت شما افراد از IDE های مختلف استفاده می‌کنند که کدها را به روش‌های مختلف مرتب می‌کند.  
توسط این ابزار و ابزار بالا می‌توانید قبل از هر کامیت کدها را طبق یک استاندارد مرتب کنید.  

[prettier](https://github.com/prettier/prettier)  

```s
npm install --save-dev --save-exact prettier
```

```s
npx prettier --write .
```

### dotnet format
این ابزار شبیه ابزار بالا برای زبان .net است.  

[format](https://github.com/dotnet/format)  

```s
dotnet tool install -g dotnet-format
```

```s
dotnet format whitespace
dotnet format style
dotnet format analyzers
```

### eslint
این ابزار نیز برای بررسی ساختار در زبان JavaScript است.  

[eslint](https://github.com/eslint/eslint)  

```s
npm install eslint --save-dev
```