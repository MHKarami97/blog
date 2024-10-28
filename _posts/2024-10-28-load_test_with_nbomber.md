---
title: "لود تست با استفاده از ابزار NBomber"
categories:
  - Net
tags:
  - net
  - load_test
  - nbomber
---

یکی از ابزارهایی که برای Load Test در .Net وجود دارد، ابزار NBomber است.  
[docs](https://nbomber.com/docs/getting-started/overview)
[NBomber](https://github.com/PragmaticFlow/NBomber)

از این ابزار می‌توانید برای تست برنامه خود با لود بالا استفاده کنید.  
بطور مثال جوابگویی Rest Api در IIS یا Nginx را مقایسه کنید.  
بدین منظور در بخش اول برنامه خود کد زیر را می‌نویسیم تا تنظیمات اولیه برنامه را به آن پاس بدهیم. تنظیماتی مانند آدرس Api یا روش انجام تست.  

```csharp
using LoadTest.Creator;
using LoadTest.Model;
using LoadTest.Test;

Console.WriteLine("Start");
Console.WriteLine("---------------");

Console.WriteLine("Insert type:\n" +
                  "1 : v2/Api/Create"
                  );

var type = int.Parse(Console.ReadLine() ?? string.Empty);

Console.Clear();

BaseTest test;

switch (type)
{
    case 1:
        test = new ApiTest(new ApiModel
            {
                Method = "POST",
                Address = "http://localhost:50291/api/v2/Api/Create",
            },
            new LoadTestModel
            {
                Name = "Create",
                Rate = 5,
                MaxFailCount = 1000,
                WarmUp = TimeSpan.FromSeconds(5),
                TimeOut = TimeSpan.FromSeconds(20),
                During = TimeSpan.FromSeconds(10),
                Interval = TimeSpan.FromMilliseconds(5),
            },
            new CreatorRule
            {
                MinPrice = 8440,
                MaxPrice = 8440,
                MinQuantity = 1000,
                MaxQuantity = 1000,
                Stores = [1,2,3],
                Customers =
                [
                    new Customer
                    {
                        Id = 165158131,
                        Type = 1,
                    },
                    new Customer
                    {
                        Id = 265155131,
                        Type = 2,
                    }
                ]
            });
        test.Do();
        break;

    default:
        Console.WriteLine("Not supported type");
        break;
}

Console.WriteLine("End");
```

تمام مدل‌های استفاده شده در برنامه را در بخش زیر مشاهده می‌کنید.  

```csharp
public class ApiModel
{
    public string Method { get; init; }
    public string Address { get; init; }
}

public class LoadTestModel
{
    public string Name { get; init; }
    public int Rate { get; init; }
    public TimeSpan Interval { get; init; }
    public TimeSpan During { get; init; }
    public TimeSpan WarmUp { get; init; }
    public TimeSpan TimeOut { get; init; }
    public int MaxFailCount { get; init; }
}

public class CreatorRule
{
    public long MinPrice { get; init; }
    public long MaxPrice { get; init; }
    public int MinQuantity { get; init; }
    public int MaxQuantity { get; init; }
    public List<Customer> Customers { get; init; }
    public List<int> Stores { get; init; }
}

public class Customer
{
    public int Id { get; init; }
    public int Type { get; init; }
}

public class OutPutModel
{
    public bool IsSuccess { get; set; }
}
```

برای ایجاد دیتا تست برای تست برنامه می‌توانید از کد زیر و کتابخانه Bogus استفاده کنید.  
با این کتابخانه برای مدل خود یک سری رول تعریف می‌کنید تا طبق آن مدل ساخته شود.  

```csharp
using Bogus;
using LoadTest.Model.Input;

namespace LoadTest.Creator;

public class InputModelCreator
{
    private readonly List<Customer> _customers;
    private Faker<InputModel> Faker { get; }

    public InputModelCreator(CreatorRule r)
    {
        _customers = r.Customers;

        var selectedCustomer = new Faker().PickRandom(_customers);

        Faker = new Faker<InputModel>()
            .RuleFor(m => m.Price, f => f.Random.Long(r.MinPrice, r.MaxPrice))
            .RuleFor(m => m.Quantity, f => f.Random.Int(r.MinQuantity, r.MaxQuantity))
            .RuleFor(m => m.InstrumentCode, f => f.PickRandom(r.Stores))
            .RuleFor(m => m.CustomerCode, f => selectedCustomer.Id)
            .RuleFor(m => m.Type, f => selectedCustomer.Type)
            .RuleFor(m => m.ExpirationDate, f => null)
            .RuleFor(m => m.Side, f => f.PickRandom<OrderSide>());
    }

    public IReadOnlyCollection<InputModel> Build(int count)
    {
        var selectedCustomer = new Faker().PickRandom(_customers);

        Faker
            .RuleFor(m => m.CustomerCode, f => selectedCustomer.Id)
            .RuleFor(m => m.Type, f => selectedCustomer.Type);

        return Faker.Generate(count);
    }

    public InputModel Build()
    {
        var selectedCustomer = new Faker().PickRandom(_customers);

        Faker
            .RuleFor(m => m.CustomerCode, f => selectedCustomer.Id)
            .RuleFor(m => m.Type, f => selectedCustomer.Type);


        return Faker.Generate(1).First();
    }
}
```

بخش Base کلاس انجام دهنده تست بصورت زیر می‌باشد که کلاس‌های بعدی از آن ارث‌بری می‌کنند.  

```csharp
using System.Diagnostics;
using NBomber.Contracts;

namespace LoadTest.Test;

public abstract class BaseTest
{
    public void Do()
    {
        var props = CreateStep();
        RunStep(props);
    }
    protected abstract ScenarioProps CreateStep();
    protected abstract void RunStep(ScenarioProps props);

    protected void ShowResult(string name)
    {
        try
        {
            var reportFinalPath = Path.Combine(Directory.GetCurrentDirectory(), $@"reports\{name}\{name}.html");

            if (File.Exists(reportFinalPath))
            {
                Process.Start(new ProcessStartInfo
                {
                    FileName = reportFinalPath,
                    UseShellExecute = true
                });
            }
            else
            {
                Console.WriteLine("Report not found.");
            }
        }
        catch (Exception)
        {
            Console.WriteLine("Report can not open.");
        }
    }
}
```

بخش اصلی برنامه در این بخش آمده است که وظیفه انجام تست را بر عهده دارد.  
در انتها نیز بعد از انجام تست بصورت اتومات نتیجه تست که فایل html است در مرورگر باز می‌شود.  

```csharp
using System.Net;
using LoadTest.Creator;
using LoadTest.Model;
using LoadTest.Model.OutPut;
using NBomber.Contracts;
using NBomber.CSharp;
using NBomber.Http;
using NBomber.Http.CSharp;
using NBomber.Plugins.Network.Ping;
using HttpVersion = NBomber.Http.HttpVersion;

namespace LoadTest.Test;

public class ApiTest(ApiModel apiModel, LoadTestModel loadConfig, CreatorRule creatorRule) : BaseTest
{
    private readonly HttpClient _httpClient = new();
    private readonly DecisionInputModelCreator _creator = new(creatorRule);

    protected override ScenarioProps CreateStep()
    {
        return Scenario.Create(loadConfig.Name, async _ =>
            {
                try
                {
                    var model = _creator.Build();
                    
                    var request = Http.CreateRequest(apiModel.Method, apiModel.Address)
                        .WithHeader("Content-Type", "application/json")
                        .WithJsonBody(model);

                    var response = await Http.Send<OutPutModel>(_httpClient, request);

                    if (response.StatusCode == HttpStatusCode.OK.ToString())
                    {
                        if (response.Payload != null)
                        {
                            if (response.Payload.Value.Data.IsSuccess)
                            {
                                return Response.Ok(statusCode: response.StatusCode, message: response.Message, sizeBytes: response.SizeBytes);
                            }

                            return Response.Fail(statusCode: HttpStatusCode.BadRequest.ToString(), message: $"Failed with status: {response.StatusCode}", sizeBytes: response.SizeBytes);
                        }
                    }

                    return Response.Fail(statusCode: response.StatusCode, message: $"Failed with status: {response.StatusCode}", sizeBytes: response.SizeBytes);
                }
                catch (Exception ex)
                {
                    return Response.Fail(message: ex.Message);
                }
            })
            .WithClean(_ =>
            {
                try
                {
                    _httpClient.Dispose();

                    Console.WriteLine("httpClient disposed after test");
                }
                catch (Exception e)
                {
                    Console.WriteLine("Error on stop" + e.Message);
                }

                return Task.CompletedTask;
            })
            .WithWarmUpDuration(loadConfig.WarmUp)
            .WithMaxFailCount(loadConfig.MaxFailCount)
            .WithLoadSimulations(
                Simulation.Inject(
                    rate: loadConfig.Rate,
                    interval: loadConfig.Interval,
                    during: loadConfig.During));
    }

    protected override void RunStep(ScenarioProps props)
    {
        NBomberRunner
            .RegisterScenarios(props)
            .WithReportFileName(loadConfig.Name)
            .WithReportFolder("reports\\" + loadConfig.Name)
            .WithClusterId(loadConfig.Name)
            .WithSessionId(loadConfig.Name)
            .WithTestName(loadConfig.Name)
            .WithScenarioCompletionTimeout(loadConfig.TimeOut)
            .WithTestSuite(loadConfig.Name)
            .WithWorkerPlugins(
                new HttpMetricsPlugin([HttpVersion.Version1]),
                new PingPlugin(PingPluginConfig.CreateDefault(apiModel.Address))
            )
            .Run();

        ShowResult(loadConfig.Name);
    }
}
```