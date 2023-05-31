---
title: "استفاده از serilog برای لاگ کردن در دیتابیس Sql Server"
categories:
  - Log
tags:
  - log
  - sql_server
  - serilog
---

یکی از کتابخونه های خیلی خوب برای لاگ کردن، کتابخونه ای به اسم serilog هستش.  
توسط این کتابخونه تقریبا میتونید تمام کارهایی که مربوط به لاگ کردن هستش رو انجام بدید.  
بطور مثال لاگ های خودتون رو بدون نیاز به انجام عملیات پیچیده، با راحت ترین روش در دیتابیس ذخیره کنید.  

برای شروع نیاز به نصب کتابخونه های زیر هست:  

```
Serilog
serilog.sinks.mssqlserver
```
بعد از نصب دو پکیج بالا نیاز به کانفیگ اولیه عملیات لاگ هستش:  
اول از همه داکیومنت  این کتابخونه رو میتونید در لینک زیر مشاهده کنید:  

[serilog](https://github.com/serilog/serilog/wiki/Getting-Started)  

داکیومنت پکیج دوم هم در لینک زیر در دسترس هست:  

[serilog_sinks_mssqlserver](https://github.com/serilog/serilog-sinks-mssqlserver)  

کدهای کلی این آموزش بصورت زیر هستش:  

```c#
namespace ConsoleTest
{
    class Program
    {
        private const int MaxWcfCall = 500;
        private const string LogTableName = "SerilogWcfConsoleTest";

        private const string LogConnectionString =
            "Server=.;Database=LogDb;Persist Security Info=True;User ID=sa;Password=1qaz@WSX;MultipleActiveResultSets=True;";

        private static Serilog.Core.Logger _logger;
        private static Random _random;

        static void Main(string[] args)
        {
            try
            {
                ConfigLogger();

                Console.WriteLine($"start time : {DateTime.Now:O}");

                TestMethod(false, true);

                Console.WriteLine($"end time : {DateTime.Now:O}");
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
                _logger.Error(e.Message);
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }

        private static void ConfigLogger()
        {
            _random = new Random();

            var columnOpts = new ColumnOptions
            {
                AdditionalColumns = new Collection<SqlColumn>
                {
                	     new SqlColumn
	                {
	                    ColumnName = "ThreadId", PropertyName = "ThreadId", DataType = SqlDbType.Int
	                },
	                new SqlColumn
	                {
	                    ColumnName = "ManagedThreadId", PropertyName = "ManagedThreadId", DataType = SqlDbType.Int
	                },
                     new SqlColumn
                     {
                        ColumnName = "CallId", PropertyName = "CallId", DataType = SqlDbType.NVarChar
                     }
                }
            };

            columnOpts.Store.Remove(StandardColumn.Properties);
            columnOpts.Store.Remove(StandardColumn.MessageTemplate);
            columnOpts.Store.Remove(StandardColumn.Level);
            columnOpts.Store.Remove(StandardColumn.Exception);

            _logger = new LoggerConfiguration()
                .MinimumLevel.Information()
                .WriteTo
                .MSSqlServer(
                    connectionString: LogConnectionString,
                    columnOptions: columnOpts,
                    sinkOptions: new MSSqlServerSinkOptions
                    {
                        TableName = LogTableName,
                        AutoCreateSqlTable = true
                    }
                ).CreateLogger();
        }

        private static void TestMethod(int callId)
        {
            for (var i = 0; i < MaxCall; i++)
            {
            	 var caller = callId.ToString("N");
                 var threadId = System.Diagnostics.Process.GetCurrentProcess().Threads[0].Id;
                 var managedThreadId = Thread.CurrentThread.ManagedThreadId;
 
                 _logger.Information(
                        "entry ,thread number: {ThreadId}, managed thread id: {ManagedThreadId}, call id: {CallId}",
                        threadId, managedThreadId, caller);

               Caller.Instance().MyMethod(callId);

                _logger.Information("exit");
            }
        }
}
```

در متود `ConfigLogger` ما کانفیگ های مربوط به لاگر رو انجام دادیم:  
قسمت کلی کانفیگ ها بخش `LoggerConfiguration` هستش  
که حالا چون ما نیاز به لاگ کردن در دیتابیس داریم اون رو با `MSSqlServer` کانفیگ کردیم.  
کتابخونه مورد نظر در ساده ترین حالت فقط نیاز به `connectionString` داره  

اگه نیاز به کانفیگ خاصی داشتید هم دست شما کاملا باز هست.  
بطور مثال در کد بالا ما چندتا از ستون های پیش فرض رو پاک کردیم و چند ستون جدید اضافه کردیم.  
همچنین ما متغیر `AutoCreateSqlTable` عملیات ساخت تیبل لاگ رو خودکار کردیم.  

در نهایت در محلی که نیاز به عملیات لاگ هست، با استفاده از کد زیر، لاگ خودمون رو ثبت کردیم:  

```c#
 _logger.Information(
                        "entry ,thread number: {ThreadId}, managed thread id: {ManagedThreadId}, call id: {CallId}",
                        threadId, managedThreadId, caller);
```
برای مقدار دادن به ستون های جدید که اضافه کردیم، نیاز به آوردن نام اونها در متن لاگ هست.  
بطور مثال در کد بالا `{ThreadId}` قرار داده شده.  
نام قرار داده شده هم به حروف بزرگ و کوچک حساس هست.  
اگر هم داخل متن نام ستون اضافه شده نباشه مقدار null براش قرار داده میشه.  


