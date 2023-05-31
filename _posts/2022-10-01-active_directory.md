---
title: "دریافت اطلاعات تمام کاربران Active Directory"
categories:
  - Trick
tags:
  - windows
  - active_directory
---

با نرم‌افزار زیر می‌توانید جزئیات را مشاهده کنید. برای پیدا کردن Domain هم به بخش زیر بروید و بخش `Full device name` را مشاهده کنید.البته دقت کنید که بخش `Device Name` را باید از آن کم کنید و فقط Domain را از بخش گفته شده بردارید.  

> Setting => System => About

[adexplorer](https://learn.microsoft.com/en-us/sysinternals/downloads/adexplorer)  

برای داشتن دسترسی بیشتر ابتدا نیاز است ابزارهای مدیریت active directory را بر روی سیستم خود نصب کنید. این ماژول‌ها بصورت پیش‌فرض در `Windows Server` فعال هستند اما بر روی ویندوز شخصی وجود ندارند و باید نصب شوند.  

[remote-server-administration-tools](https://learn.microsoft.com/en-US/troubleshoot/windows-server/system-management-components/remote-server-administration-tools)  

ابتدا به بخش زیر بروید:  

> Setting => Apps => Optional Features => Add

سپس این مورد را جستجو و نصب کنید:  

> Remote Server Administration Tools (RSAT)

بعد از نصب `PowerShell` را بصورت Admin باز کنید و خط زیر را در آن وارد کنید:  

```powershell
import-module ActiveDirectory
```

اکنون توسط دستور‌های مختلف می‌توانید اطلاعات مورد نظر خود را درآورید.  

[active-directory](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)  

بطور مثال می‌توانید از کوئری زیر برای بدست آوردن اطلاعات تمام کاربران استفاده کنید.  

```powershell
Get-ADUser -Filter 'SamAccountName -like "."' -Properties telephoneNumber,mobile,MobilePhone | Format-Table Name,SamAccountName,UserPrincipalName,telephoneNumber,mobile,MobilePhone -A
```

دقت کنید در Active Directory می‌توان مقادیر دلخواه تعریف کرد که بصورت پیش‌فرض اطلاعات آنها در کوئری نشان داده نمی‌شود و اگر مقادیر آنها را هم می‌خواهید باید شبیه کوئری بالا بصورت `-Properties telephoneNumber` آنها را وارد کنید و سپس بعد از `|` تا نشان داده شوند.  

[get-ADUser](https://learn.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps)  