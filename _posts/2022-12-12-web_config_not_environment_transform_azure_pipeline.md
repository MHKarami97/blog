---
title: "عدم کارکرد درست Environment در Web.config"
date: 2022-12-12T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - pipe_line
  - environment
---

اگر پروژه شما دارای فایل `web.config` است و برای محیط‌های مختلف آن فایلی مانند `web.Prod.config` تعریف کرده‌اید و همچنین از `Azure PipeLine` برای CI/CD استفاده می‌کنید، اگر از `DependentUpon` در آن استفاده کرده باشید با خطا مواجه می‌شوید و در واقع تمام کانفیگ‌ها منتقل نمی‌شود و خطا پیدا نشدن فایل را دریافت می‌کنید.  
راه‌حل آن است که مقدار `<DependentUpon>Web.config</DependentUpon>` را از آن پاک کنید.  

```csharp
<Content Include="Web.config">
    <SubType>Designer</SubType>
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</Content>
<Content Include="Web.Alpha.config">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <IsTransformFile>true</IsTransformFile>
    <DependentUpon>Web.config</DependentUpon>
</Content>
<Content Include="Web.Staging.config">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <IsTransformFile>true</IsTransformFile>
    <DependentUpon>Web.config</DependentUpon>
</Content>
<Content Include="Web.Prod.config">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <IsTransformFile>true</IsTransformFile>
    <DependentUpon>Web.config</DependentUpon>
</Content>
```

این مشکل در دایومنت خود ماکروسافت هم توضیح داده شده است:  

[transforms-variable-substitution](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/transforms-variable-substitution?view=azure-devops&tabs=Classic#xml-transformation-notes)