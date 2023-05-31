---
title: "دیزاین پترن State در عمل"
categories:
  - DesignPattern
tags:
  - pattern
  - csharp
  - state_pattern
---

در قسمتی از پروژه ای که در حال نوشتن بودم و میخواستیم Fail Over رو بر روی قسمتی از سیستم پیاده سازی کنیم تا سیستم بصورت اتومات بتونه بعد از به خطا خوردن و یا مشکل دیگه ای، در سریعترین زمان ممکن یه نسخه دیگه کارش رو انجام بده، نیاز بود تا یه نتیجه گیری برای دونستن وضعیت بگیریم
<br />
البته خود این fail over یه بحث جدا هست و توضیحات بیشتری میخواد که در آینده دربارش مینویسم
<br />
بصورت خلاصه این قسمت از برنامه در حال دریافت پیام های بورس هست و اون پیام ها رو برای بقیه سیستم در اختیار میزاره، اگه این قسمت قطع بشه میشه گفت کل سیستم میخوابه
<br />
پس اون fail over تو اینجا بدرد میخوره
<br />
حالا برای این کار تو قسمتی از کد نیاز بود که بدونیم از چه پیامی باید ادامه بدیم
<br />
فرض کنید ما دوتا نسخه داریم با اسم های primary و secondary که اولی پیام ها رو به سیستم ارسال میکنه ولی دومی فقط پیام ها رو در دیتابیس ذخیره میکنه و هرموقع اون اولی به مشکل خورد، بجاش ادامه میده
<br />
برای اینکه secondary بتونه بجای سیستم اول ادامه بده، سه تا حالت مختلف پیش میاد
<br />

- درست تا جایی که سیستم اول پیام ها رو پردازش کرده بود، دریافت کرده باشه
- از سیستم اول عقب تر باشیم
- از سیستم دوم جلوتر باشیم

حالا تو هر کدوم از این حالت ها باید تصمیم متفاوتی بگیریم
<br />
اگه عقب تر باشیم باید پیام ها رو دریافت کنیم و داخل دیتابیس ذخیره کنیم و زمانی که درست به حالت سیستم اول رسیدیم، پیام ها رو ارسال هم کنیم
<br />
اگه جلوتر باشیم باید از دیتابیس بخونیم و بعدش شبیه حالت بالا
<br />
<br />

حالا برای این کار لازم بود که بدونیم دقیقا در چه حالتی هستیم که تو سناریوهای مختلف 3 متغیر زیر به برنامه اضافه شدن

```c#
public static bool IsPrimary { get; private set; }
public static bool IsBefotrPrimary { get; private set; }
public static bool IsFirstTime { get; private set; }
```

تو قسمت نتیجه گیری هم با توجه به اینها تصمیم میگرفتیم که چیکار کنیم، شبیه حالت زیر

```c#
private static int GetLastDispatchedMessageIdTypeCode()
{
    int result;

    switch (FailOverService.IsBeforePrimary)
    {
        case true when FailOverService.IsPrimary:
            result = PrimaryTypeCode;
            break;
        
        case true when !FailOverService.IsPrimary:
            result = SecondaryTypeCode;
            break;
        
        case false when !FailOverService.IsPrimary:
            result = SecondaryTypeCode;
            break;
        
        case false when FailOverService.IsPrimary && FailOverService.IsFirstTime:
            result = PrimaryTypeCode;
            break;
        
        case false when FailOverService.IsPrimary && !FailOverService.IsFirstTime:
            result = SecondaryTypeCode;
            break;
        
        default:
            throw new Exception("not valid input");
    }

    return result;
}
```

اگه با clean code آشنایی داشته باشید، میبیند که کد بالا خیلی بد هست و خوانایی خیلی کمی هم داره
<br />
روشی که برای درست کردن کد بالا انجام دادم، استفاده از دیزاین پترن State بود

[https://sourcemaking.com/design_patterns/state](https://sourcemaking.com/design_patterns/state)  

خلاصه کار به این صورت بود:
<br />
یه کلاس با اسم Context به این صورت:

```c#
public class Context
{
    private State _state;

    public Context()
    {
        _state = new InitialState();
    }

    public void TransitionTo(State state)
    {
        Logger.WriteInformationLog($"Context: Transition to {state.GetType().Name}");

        _state = state;
        _state.SetContext(this);
    }

    public FailoverReadingTypeEnum GetState()
    {
        return _state.Handle();
    }
}
```

کلاس State به صورت زیر:

```c#
public abstract class State
{
    protected Context Context;

    public void SetContext(Context context)
    {
        Context = context;
    }

    public abstract FailoverReadingTypeEnum Handle();
}
```

این حالت فقط برای اجرا در حالت اول است و عملیاتی در آن انجام نمی‌شود:  

```c#
public class InitialState : State
{
    public override FailoverReadingTypeEnum Handle()
    {
        Logger.WriteInformationLog($"FailoverReadingTypeEnum change to : {FailoverReadingTypeEnum.None.ToString()}");
        
        return FailoverReadingTypeEnum.None;
    }
}
```

یه Enum با مقادیر زیر

```c#
public enum FailoverReadingTypeEnum
{
    None = 0,
    Primary = 1,
    Secondary = 2
}
```

روش کار به این صورت هست که ما برای هر کدوم از حالت هایی که نیاز داریم یه کلاس جدید درست میکنیم که از State ارث بری کرده باشه
<br />
به این صورت:

```c#
public class PrimaryState : State
{
    public override FailoverReadingTypeEnum Handle()
    {
        Logger.WriteInformationLog($"FailoverReadingTypeEnum change to : {FailoverReadingTypeEnum.Primary.ToString()}");
        
        return FailoverReadingTypeEnum.Primary;
    }
}
```

برای استفاده هم کافی هست کلاس Context رو تو قسمتی که میخوایم New کنیم و در زمان هایی که لازم داریم State اون رو تغییر بدیم تا State فعلی رو بگیریم
<br />

```c#
public FailOverService(ProcessorType processorType)
{
    _context = new Context();
}
```
به این صورت میتونیم حالت رو تغییر بدیم:

```c#
_context.TransitionTo(new PrimaryState());
```

که ورودی همون کلاس هایی هستن که از State ارث بری کردن
<br />
برای دریافت حالت هم کافیه بصورت زیر عمل کنیم
<br />

```c#
public static FailoverReadingTypeEnum GetReadingType()
{
    return _context.GetState();
}
```

با این کار در درجه اول یکی از اون boolean ها برای دونستن حالت حذف شد و فقط دو مقدار IsPrimary و IsFirstTime باقی موندن
<br />
اون متد که توش از switch , case استفاده شده بود هم بصورت زیر در اومد
<br />

```c#
private static int GetLastDispatchedMessageIdTypeCode()
{
    return (int) FailOverService.GetReadingType();
}
```
