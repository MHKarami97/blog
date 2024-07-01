---
title: "اعتبار سنجی مقادیر Enum"
categories:
  - Net
tags:
  - enum
  - default_value
  - validation
---

یکی از مواردی که در C# باید به آن دقت کرد این است که مقدار پیش‌فرض برای Enum برابر با 0 است و حتی اگر مقدار صفر در Enum نباشد باز مقدار گفته شده را می‌تواند بگیرد. برای جلوگیری از این مورد می‌توانید کلاسی با نام NotNoneAttribute بنویسید که جلوی این مشکل را بگیرد.  

```csharp
namespace Model
{
    public enum MyEnum
    {
        None = 1000,
        Value1 = 2000,
        Value1 = 3000
    }

    public class NotNoneAttribute : ValidationAttribute
    {
        public override bool IsValid(object value)
        {
            if (value is MyEnum enu)
            {
                return Enum.IsDefined(typeof(MyEnum), enu);
            }

            return false;
        }

        public override string FormatErrorMessage(string name)
        {
            return $"The field {name} cannot be set to the default value. Please choose a valid MyEnum.";
        }
    }
}
```
سپس می‌توانید بصورت زیر از Validation اضافه شده استفاده کنید.  

```csharp
[Required]
[NotNone]
public MyEnum MyEnum { get; set; }
```