---
title: "ایجاد http call در net core با flurl و polly"
date: 2020-12-16T21:41:00-00:00
categories:
- core
  tags:
- core
- polly
- httpclient
- flurl
---

<div dir="rtl">

یکی از مشکلاتی که اافراد در زمان کار با http client در زبان برنامه نویسی .net مواجه می شوند، سخت بودن نسبی مدیریت آن و همچنین نیاز به نوشتن کدهای زیاد برای کال کردن یک api خارجی می باشد.
<br />
یکی از کتابخانه های خوب برای این کار که تقریبا تمام مشکلاتی که در زمان کار با http client ذکر شد را حل می کند، کتابخوانه FlUrl می باشد.
<br />
از کتابخوانه های خوب دیگر برای مدیریت خطاها و تکرار کردن اتومات api call در صورت مواجه شدن با خطا، وجود دارد، کتابخوانه polly می باشد
<br />
<br />
لینک های موارد معرفی شده:
<br />
[polly github](https://github.com/App-vNext/Polly)  
[flurl github](https://github.com/tmenier/Flurl)  
<br />
روش کار کردن بخصوص با flurl بسیار آسان می باشد و در هر دو حالت نیاز به هیچ کانفیگ خاصی نیست
<br />
بطور مثال برای فراخوانی یک http کافی است بصورت زیر عمل کنید:
<br />
<div dir="ltr">

```c#
var result = await "https://api.com"
    .SetQueryParams(new { api_key = "abc" })
    .WithOAuthBearerToken("my_token")
    .PostJsonAsync(new { first_name = firstName, last_name = lastName })
    .ReceiveJson<T>();
```

</div>
همانطور که مشاهده میکنید روش کار بسیار آسان می باشد و نیاز به هیچ گونه تزریق وابستگی HttpClient نیز نیست.
<br />
کافی است آدرس پایه api خود را بصورت string وارد کنید و توسط موارد پیاده سازی شده در کتابخوانه به نتیجه دلخواه خود برسید.
<br />
مواردی که در کد بالا آمده اند برای عناوین زیر استفاده می شوند:
<br />

- SetQueryParams : برای اضافه کردن پارامتر دلخواه به آدرس
- WithOAuthBearerToken : اضافه کردن توکن به هدر
- PostJsonAsync : مواردی که بصورت post ارسال می شوند
- ReceiveJson : نتیجه بصورت json

البته موارد بسیار بیشتری نیز وجود دارد که می توانید از [این لینک](https://flurl.dev/docs/fluent-http) آن ها را مشاهده کنید
<br />
بطور مثال می توانید نوع خروجی را تعیین نکنید تا بصورت dynamic مقدار دهی شود.
یا هدر های دلخواه را به ریکوئست خود اضافه کنید
<br />
و تمام موارد دیگری که برای کار کردن با api ها لازم می شود
<br />
<br />
از موارد بسیار خود این کتابخوانه قسمت exception handing آن می باشد
<br />
می توانید status code ها را بصورت زیر تعریف کنید تا خطا صادر نشود و یا بصورت پیش فرض قرار دهید تا در صورتی که برنامه با کد 200 مواجه نشد خطا صدر شود
<div dir="ltr">

```c#
url.AllowHttpStatus("400-404,6xx").GetAsync();
url.AllowAnyHttpStatus().GetAsync();
```

</div>

در صورت به خطا خودرن دو exception زیر صادر می شوند:

- FlurlHttpTimeoutException
- FlurlHttpException

در حالت خطا نیز می توانید خروجی را بخوانید. بطور مثال در حالت هایی که api در حالت خطا نیز توضیحاتی ارسال می کند
<div dir="ltr">

```c#
return await ex.GetResponseJsonAsync() ??
             ex.GetBaseException().Message;
```

</div>

از موارد مهمی که در http client باید رعایت شود، ایجاد نکردن آن به ازای هر ریکوست می باشد که در [این کتابخوانه](https://flurl.dev/docs/client-lifetime) به درستی پیاده سازی شده است
<br />
این کتابخوانه قابلیت تست و ایجاد حالت fake نیز دارد که در نوشتن تست های مختلف برای برنامه کاربرد فراوان دارد که از [این لینک](https://flurl.dev/docs/testable-http/) توضیحات کامل آن را می توانید مشاهده کنید:
<div dir="ltr">

```c#
[Test]
public void Test_Some_Http_Calling_Method() {
    using (var httpTest = new HttpTest()) {
        sut.CallThingThatUsesFlurlHttp();
    }
}
```

</div>
<br />
یکی از کتابخوانه های خوب دیگر Polly می باشد که بطور مثال در مواقعی استفاده می شود که شما نیاز دارید در صورت به خطا خوردن یک api call به تعداد دلخواه نیز آن عمل تکرار شود
<br />
برای استفاده از این کتابخوانه بصورت زیر عمل می کنیم.
<br />

<div dir="ltr">

```c#
private readonly AsyncRetryPolicy _polly;

public MyConstructor()
{
    _polly = Policy
        .Handle<FlurlHttpTimeoutException>()
        .WaitAndRetryAsync(new[]
        {
            TimeSpan.FromSeconds(1),
            TimeSpan.FromSeconds(2)
        });
}
```

</div>

در کد بالا در قسمت Handle می توانیم نوع خطایی که اگر برنامه با آن مواجه شد، دوباره آن عمل را انجام دهد را مشخص کنیم. که در این کد ما خطایی time out کتابخوانه flurl را مشخص کرده ایم.
<br />
در قسمت بعد نیز ما تعداد تکرار ها و مدت زمان صبر بین هر تکرار را مشخص کرده ایم.
<br />
بطور مثال در کد گفته ایم که دو بار عمل را تکرار کن و در دفعه اول 1 ثانیه و در دفعه دوم 2 ثانیه قبل از تکرار صبر کن
<br />
اکنون کافی است بصورت زیر در هر قسمت از کد که نیاز باشد، عمل کنیم:
<div dir="ltr">

```c#
var result = await _polly
    .ExecuteAsync(async ct =>
            await MyMethod(),
        cancellationToken);
```

</div>

بطور مثال اگر از flurl بخواهیم استفاده کنیم بصورت زیر می شود:
<div dir="ltr">

```c#
var result = await _polly
    .ExecuteAsync(async ct =>
            await BaseInfo.WebApiAddress
                .AppendPathSegment("auth")
                .WithHeader(abc,
                    "in header")
                .PostJsonAsync(new
                {
                    Username = userUserName,
                    Password = userPassword
                }, ct).ReceiveJson<ResultInfo>(),
        cancellationToken);
```

</div>

</div>