---
title: "پیدا کردن خط‌های تکراری در C#"
date: 2023-01-29T00:00:00-00:00
categories:
  - Net
tags:
  - trick
  - dublicate
  - group_by
  - linq
---

اگر در برنامه خود نیاز دارید که خط‌های تکراری در یک فایل را پیدا کنید می‌توانید از کد زیر استفاده کنید.  

ابتدا تمام فایل را خوانده و در یک متغیر می‌ریزیم.  
سپس بر روی آن GroupBy انجام می‌دهیم و توسط شرط Where خط‌هایی که تکراری هستند را پیدا می‌کنیم.  

```csharp
const string path = "text.txt";

var lines = File.ReadAllLines(path);
var totalData = lines.GroupBy(x => x).Where(g => g.Count() > 1).ToList();
var duplicateData = totalData.Select(g => g.Key).ToArray();
var countDuplicateData = totalData.Select(g => g.Count()).ToArray();
```

افزونه زیر هم برای کارهایی مانند پیدا کردن خط‌های تکراری و ... کاربردر دارد.  

[transformer](https://marketplace.visualstudio.com/items?itemName=dakara.transformer#overview)  