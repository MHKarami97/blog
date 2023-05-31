---
title: "الگوی Chain Of Responsibility"
categories:
  - DesignPattern
tags:
  - design_pattern
  - chain_of_responsibility
---

یکی از الگوهای طراحی که در برنامه نویسی کاربرد داره، الگویی با اسم Chain Of Responsibility هستش.  
این الگو در زمان هایی  کاربرد داره که یک سری عملیات رو بصورت زنجیر وار برای یک درخواست میخوایم انجام بدیم.  
بطور مثال برای قسمت ثبت سفارش خرید در یک سایت نیاز هست که ابتدا بررسی کنیم مشتری موجودی داره، کالا موجود هست و مشتری در محدوده مورد تایید واقع شده.  
در این حالت میشه از الگوی گفته شده استفاده کرد.  

بصورت پیش فرض در صورت به خطا خوردن در هر بخش، بقیه بخش ها بررسی نمیشن و خطا داده میشه، اما فرض کنید در مثال بالا میخوایم اگه مشتری موحودی نداشت هم بقیه شرطها بررسی بشن تا مشتری بتونه در محل هم پرداخت کنه.  
پس الگوی گفته شده رو یه مقدار تغییر میدیم.

قسمت اصلی کار بخش زیر هست که یک کلاس از نوع abstract تعریف کردیم.  
شرطهای ما که همون زنجیرها میشن از این کلاس ارث بری میکنن.  
در این کلاس متغیری با اسم `shouldContinueOnError` هست که مشخص میکنه اگه یک شرط به خطا خورد ادامه داده بشه و شرط های بعدی بررسی بشن یا نه.  

روش کلی به این صورت هست که شرط هایی که از این کلاس ارث بری کردن متود `CheckMethod` رو بازنویسی میکنن و شرط دلخواهشون رو داخل اون قرار میدن.  
سپس توسط `SetNext` شرط بعدی فراخونی میشه و بعد از تموم شدن زنجیرها، نتایج برگردونده میشن.  

در این کد از `OperationResult` استفاده شده که یک کلاس دلخواه از نوع جنریک هست تا نتیجه و خطا رو بتونیم داخل اون قرار بدیم.  

```c#
namespace My
{
    public abstract class RequestBaseValidator
    {
        private RequestBaseValidator _nextHandler;
        protected readonly Request Request;
        private readonly bool _shouldContinueOnError;

        protected RequestBaseValidator(Request request, bool shouldContinueOnError)
        {
            Request = request;
            _shouldContinueOnError = shouldContinueOnError;
        }

        public void SetNext(RequestBaseValidator requestBaseValidator)
        {
            if (_nextHandler == null)
            {
                _nextHandler = requestBaseValidator;
            }
            else
            {
                _nextHandler.SetNext(requestBaseValidator);
            }
        }

        public OperationResult Validate(bool hasError = false)
        {
            var result = CheckMethod();
            var nextResult = new OperationResult();

            hasError = hasError || !result.IsSuccess;
            
            if (hasError && !_shouldContinueOnError)
            {
                return result;
            }

            if (_nextHandler != null)
            {
                nextResult = _nextHandler.Validate(hasError);
            }

            result.AddErrors(nextResult.GetErrors());

            return result;
        }

        protected abstract OperationResult CheckMethod();
    }
}
```

برای اضافه کردن شرط یا همون زنجیر جدید میتونید بصورت زیر عمل کنید:  

```c#
namespace My
{
    public class Custom1Validator : RequestBaseValidator
    {
        public Custom1Validator(Request request, bool shouldContinueOnError)
            : base(request, shouldContinueOnError)
        {
        }

        protected override OperationResult CheckMethod()
        {
            var result = new OperationResult();

            if (Request.Side != Side.Buy)
            {
                result.AddError(new Error
                {
                    Description = MessageText.NotValidSide,
                    ErrorCode = ErrorCode.NotValidSide
                });
            }

            return result;
        }
    }
}
```

در انتها برای وصل کردن زنجیر ها به هم از کد زیر استفاده میکنیم:  

```c#
namespace My
{
    public class RequestValidatorFactory
    {
        public RequestnBaseValidator CreateChain(Request request)
        {
            var custom1Validator = new Custom1Validator(request, false);

            var Custom2Validator = new Custom2Validator(request, true);

            custom1Validator.SetNext(Custom2Validator);
            Custom2Validator.SetNext(null);

            return custom1Validator;
        }
    }
}
```

در صورتی که متغیر bool پاس داده شده برابر با `bool` باشه زنجیرهای در صورت به خطا خودرن در اون مرحله ادامه داده نمیشن.

روش فراخونی هم بصورت زیر هست:  

```c#
private bool IsValidate(out List<Error> errors)
{
    var requestValidatorFactory = new RequestValidatorFactory();
    var validator =
        requestValidatorFactory.CreateChain(this);

    errors = validator.Validate().GetErrors().ToList();
    
    return !errors.Any();
}
```

در لینک زیر هم میتونید درباره این الگو بیشتر مطالعه کنید:  

[refactoring](https://refactoring.guru/design-patterns/chain-of-responsibility/csharp/example)  

