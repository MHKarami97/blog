---
title: "ساخت فایل proto برای GRPC Service در کتابخانه protobuf-net.Grpc"
date: 2021-07-04T10:03:00-00:00
categories:
  - GRPC
tags:
  - grpc
  - proto
  - protobuf_net_Grpc
  - bloomrpc
---

یکی از کتابخانه های خوب برای نوشتن سرویس GRPC کتابخونه ای با اسم protobuf-net.Grpc هست که قابلیت Code First رو به شما میده و لازم نیست اول فایل proto رو بسازید که در نتیجه نوشتن سرویس رو خیلی راحتر میکنه.  
اما مشکلی که احتمالا باهاش روبرو میشید همین نبودن فایل proto هست، به طور مثال من نیاز داشتم توسط ابزار Jmeter سرویس نوشته شده رو تست کنم و یا از درست کار کردن سرویس مطمئن بشم اما برای این کار در ابزارهایی که بود نیاز به داشتن فایل proto بود. 

بعد از جستجوهایی که کردم متوجه یه کتابخونه ثانویه برای کتابخونه گفته شده در بالا شدم که برای سرویس شما این فایل رو میسازه.  
اول از همه کتابخونه اصلی از لینک زیر در دسترس هست:  
[protobuf-net.Grpc](https://github.com/protobuf-net/protobuf-net.Grpc)  

و کتابخونه ثانویه هم از لینک زیر:  
[protobuf-net.Reflection](https://www.nuget.org/packages/protobuf-net.Reflection)  

برای ساخته شدن فایل proto کافی هست کد زیر رو به Service خود اضافه کنید:  

```c#
var generator = new SchemaGenerator
{
    ProtoSyntax = ProtoSyntax.Proto3
};

var schema = generator.GetSchema<ICalculator>(); // there is also a non-generic overload that takes Type

using (var writer = new System.IO.StreamWriter("services.proto"))
{
    await writer.WriteAsync(schema);
}
```

کد کامل قسمت Service هم بصورت زیر میشه:  

```c#
static async Task Main(string[] args)
{
    try
    {
        const int port = 10042;

        var server = new Server
        {
            Ports =
            {
                new ServerPort("localhost", port, ServerCredentials.Insecure)
            }
        };

        server.Services.AddCodeFirst(new MyCalculator());

        server.Start();

        var generator = new SchemaGenerator
        {
            ProtoSyntax = ProtoSyntax.Proto3
        };

        var schema = generator.GetSchema<ICalculator>(); // there is also a non-generic overload that takes Type

        using (var writer = new System.IO.StreamWriter("tet.proto"))
        {
            await writer.WriteAsync(schema);
        }

        Console.WriteLine("server listening on port " + port);

        Console.ReadKey();

        await server.ShutdownAsync();
    }
    catch (Exception e)
    {
        Console.WriteLine(e);

        throw;
    }
}
```

کدهای زیر هم برای قسمت Client برای فراخونی سرویس های نوشته شده هست:  

```c#
static async Task Main(string[] args)
{
    var channel = new Channel("localhost", 10042, ChannelCredentials.Insecure);
    try
    {
        var calculator = channel.CreateGrpcService<ICalculator>();

        var response = await calculator.MultiplyAsync(new MultiplyRequest
        {
            X = 2,
            Y = 4
        });

        if (response.Result != 8)
        {
            throw new InvalidOperationException();
        }

        Console.WriteLine(response.Result);
    }
    finally
    {
        await channel.ShutdownAsync();
    }
}
```

قسمت مشترک کدهای بالا یا همون تابع نوشته شده:  

```c#
namespace GRPC
{
    [ServiceContract(Name = "Hyper.Calculator")]
    public interface ICalculator
    {
        ValueTask<MultiplyResult> MultiplyAsync(MultiplyRequest request);
    }
    
    [DataContract]
    public class MultiplyRequest
    {
        [DataMember(Order = 1)]
        public int X { get; set; }

        [DataMember(Order = 2)]
        public int Y { get; set; }
    }

    [DataContract]
    public class MultiplyResult
    {
        [DataMember(Order = 1)]
        public int Result { get; set; }
    }
}
```

توسط این کتابخونه هم میتونید سرویس ساخته شده رو فراخونی کنید:  
[bloomrpc](https://github.com/uw-labs/bloomrpc)

<p align="center" >
  <img src="https://github.com/uw-labs/bloomrpc/raw/master/resources/editor-preview.gif" alt="mhkarami97" width="400" />
</p>
