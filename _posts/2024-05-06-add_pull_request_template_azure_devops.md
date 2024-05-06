---
title: "اضافه کردن Template به PullRequest در AzureDevops"
categories:
  - trick
tags:
  - azure
  - git
  - pull_request
  - devops
---

برای اطمینان بیشتر از اینکه در یک Pull Request تمام موارد مهم بررسی شده‌اند، می‌توانید یک CheckList به توضیحات آن اضافه کنید تا افراد قبل از زدن پول‌ریکوست آن چک لیست را بررسی کنند.  
بدین منظور کافی است یک فایل به اسم `pull_request_template.md` در برنچ پیش‌فرض پروژه خود بسازید. این برنچ بیشتر مواقع master است.  



```csharp
## Check: 
- [x] current branch naming
- [ ] clean solution and rebuild
- [ ] undo all config change
- [ ] add unitTest/IntegrationTest for changed/added code
- [ ] pass all tests
- [ ] remove all unused using on code
- [ ] remove all unnecessary white space
- [ ] clean coding
- [ ] remove unnecessary comments
- [ ] change documents if needed
- [ ] review by your self
- [ ] review by other team person

## What type of PR is this? (check all applicable)

- [ ] Refactor
- [ ] Feature
- [ ] Bug Fix
- [ ] Optimization
- [ ] Documentation Update
- [ ] Config

## Description
Please include a summary of the changes and the related issue. Please also include relevant motivation and context. List any dependencies that are required for this change `(remove this lines first)`
```

توضیحات بالا بصورت پیش‌فرض در تمام پول‌ریکوست‌ها نشان داده خواهند شد.  
همچنین می‌توانید چند فایل اضافه کنید. بطور مثال برای تغییرات از یک برنچ خاص یک توضیح دیگر نمایش دهید. بدین منظور می‌توانید از لینک زیر استفاده کنید.  

[azure-devops](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-request-templates?view=azure-devops)  