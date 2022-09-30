---
title: "یکی کردن مقادیر یک ستون در SQL"
date: 2022-04-25T12:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - substring
  - xml
---

اگر نیاز داشتید که تمام سطرهای یک ستون خاص در دیتابیس را به صورت یک خروجی درآورید، شبیه به قابلیت Concat در زبان های برنامه نویسی، می‌توانید از کوئری زیر استفاده کنید.  


```sql
SELECT SUBSTRING((SELECT ',' + body
                  FROM LogStep
                  FOR XML PATH('')), 2, 9999) AS result
```