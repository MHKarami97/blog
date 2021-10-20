---
title: "پیاده سازی Two Phase Commit یا TPC در .Net"
date: 2021-09-28T11:03:00-00:00
categories:
  - Net
tags:
  - tpc
  - two_phase_commit
  - IEnlistmentNotification
  - transaction
  - distributed
---

یکی از موارد مهم در زبان های برنامه نویسی بحث Transaction ها می‌باشد. در بعضی موارد تمام مواردی که می‌خواهید بصورت یک ترنزکشن انجام شوند، بر روی یک سیستم نیستند. بخصوص در مواردی که از MicroService ها استفاده می‌کنید.  
بطور مثال می‌خواهید فراخوانی یک API از سیستم دیگر و ثبت در دیتابیس خودتان بصورت ترنزکشن انجام شود.  
در این موارد بحث Distributed Transaction آغاز می‌شود.  

برای پیاده سازی این نوع ترنزکشن ها دو روش کلی TPC و SAGA وجود دارد که در این مطلب ما روش TPC را بررسی می‌کنیم.  
این مطلب بیشتر درباره روش پیاده سازی این روش می‌باشد، اگر مایل هستید درباره این روش بیشتر بدانید می‌توانید لینک زیر را مشاهده کنید:  

[wikipedia](https://en.wikipedia.org/wiki/Two-phase_commit_protocol)  

<p align="center" >
  <img src="/assets/img/tpc.png" alt="mhkarami97" width="600" />
</p>

برای پیاده سازی راحت تر این روش یک Interface با نام `IEnlistmentNotification` در زبان .net ایجاد شده است که موارد مورد نیاز برای این کار را پیاده سازی می‌کند.  

درباره این اینترفیس می‌توانید اطلاعات بیشتری در لینک زیر پیدا کنید:  

[microsoft docs](https://docs.microsoft.com/en-us/dotnet/api/system.transactions.ienlistmentnotification)  

این اینترفیس از 4 متود زیر تشکیل شده است:  

  - `Commit` : به یک شیء ثبت شده اطلاع می دهد که یک تراکنش در حال انجام است.
  - `InDoubt` : به یک شیء ثبت شده اطلاع می دهد که وضعیت یک تراکنش مشکوک است.
  - `Prepare` : به یک شیء ثبت شده اطلاع می دهد که یک ترنزکشن برای کامیت آماده می شود.
  - `Rollback` : به یک شیء ثبت شده اطلاع می دهد که یک معامله لغو می شود

کد کلی این پیاده سازی بصورت زیر است:  

```c#
namespace Blocks.Concretes
{
    public class SendTradeToTraderCommander : IEnlistmentNotification
    {
        private readonly BlockDto _input;
        private readonly IServicesProvider _serviceProvider;

        public SendTradeToGroupTraderCommander(BlockDto input,
            IServicesProvider serviceProvider)
        {
            _input = input;
            _serviceProvider = serviceProvider;
        }

        public void Commit(Enlistment enlistment)
        {
            //Declare done on the enlistment
            enlistment.Done();
        }

        public void InDoubt(Enlistment enlistment)
        {
            //Declare done on the enlistment
            enlistment.Done();
        }

        public void Prepare(PreparingEnlistment preparingEnlistment)
        {
            Enqueue();

            //If work finished correctly, reply prepared
            preparingEnlistment.Prepared();
        }

        public void Rollback(Enlistment enlistment)
        {
            try
            {
                _externalServiceProvider.SendTradeErrorForToTraderProcess(_input);

                enlistment.Done();
            }
            catch (Exception e)
            {
                Logger.Instance.Log4Repo.Critical(
                    "Exception in rollback", e);
            }
        }

        public void Execute()
        {
            if (Transaction.Current != null)
            {
                //Enlist on the current transaction with the enlistment object
                Transaction.Current.EnlistVolatile(this, EnlistmentOptions.None);
            }
            else
            {
                Enqueue();
            }
        }

        private void Enqueue()
        {
            _serviceProvider.SendTradeToTraderProcess(_input);
        }
    }
}
```

برای این کار یک کلاس با نام `SendTradeToTraderCommander` ایجاد شده است که اینترفیس گفته شده را پیاده سازی کرده است کرده است.  
در کدهای بالا بجز متود `Execute` دیگر متودها در اینترفیس گفته شده فراهم شده اند و در این کلاس پیاده سازی شده اند.  

برای فراخوانی کلاس بالا بصورت زیر عمل می‌شود:  

```c#
public void ReleaseFromBlock(BlockDto input)
{
    var commander = new SendTradeToTraderCommander(input, new _ServiceProviderWithNoCache());

    commander.Execute();
}
```

با فراخونی متد `Execute` عملیات شما که در این مثال فراخونی یک API می‌باشد شروع می‌شود و در صورت به خطا خوردن بصورت اتوماتیک متود `Rollback` فراخونی می‌شود که در آن یک API دیگر برای برگشت تراکنش فراخوانی می‌شود و در صورت به خطا خوردن این عملیات نیز یک لاگ ثبت می‌شود.  