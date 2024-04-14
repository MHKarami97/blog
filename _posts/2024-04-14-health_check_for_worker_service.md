---
title: "اضافه کردن HealthCheck به worker Service در .Net Core"
categories:
  - Net
tags:
  - net
  - health_check
  - worker_service
  - agent
---

یکی از ابزارهای مفید برای مانیتور سیستم در Production برای اطمینان از صحت انجام کارها، ابزار healthCheck است.  
برای اضافه کردن این ابزار به Worker Service ها که UI ندارند کافی است بصورت زیر عمل کنید تا بتوانید از این ابزار در آنها استفاده کنید:  

```c#
using System.Globalization;
using System.Text.Json;
using System.Text.Json.Serialization;
using Flurl.Http;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;
using Microsoft.Extensions.Options;

namespace Reporter.Configuration;

public static class HealthCheck
{
    public static void ConfigureHealthCheck(this IServiceCollection services, IConfiguration configuration)
    {
        var database = configuration.GetConnectionString("Database");
        var reporterDatabase = configuration.GetConnectionString("ReporterDatabase");
        var bDatabase = configuration.GetConnectionString("BDatabase");
        var redis = configuration.GetConnectionString("RedisCache");

        if (string.IsNullOrEmpty(database) ||
            string.IsNullOrEmpty(reporterDatabase) ||
            string.IsNullOrEmpty(bDatabase) ||
            string.IsNullOrEmpty(redis))
        {
            throw new InvalidOperationException("Could not find a connection string named " +
                                                "'database' or " +
                                                "'reporterDatabase' or " +
                                                "'bDatabase' or " +
                                                "'redis'");
        }

        _ = services.AddHealthChecks()
            .AddSqlServer(database, name: "Database")
            .AddSqlServer(reporterDatabase, name: "reporterDatabase")
            .AddOracle(bDatabase, name: "bDatabase")
            .AddRedis(redis, name: "Redis")
            .AddCheck<EndpointHealthChecker>("Apm", HealthStatus.Degraded, new[] { nameof(EndpointHealthChecker) })
            .AddCheck<AppSettingChecker>("AppSettingChecker", HealthStatus.Degraded, new[] { nameof(AppSettingChecker) })
            .AddCheck<SystemMemoryHealthcheck>("SystemMemoryHealthcheck", HealthStatus.Degraded, new[] { nameof(SystemMemoryHealthcheck) });
    }

    public static void AddHealthCheck(this IEndpointRouteBuilder app)
    {
        _ = app.MapHealthChecks("/health", new HealthCheckOptions
        {
            Predicate = _ => true,
            ResponseWriter = WriteHealthCheckResponseAsync,
        });
    }

    private static Task WriteHealthCheckResponseAsync(HttpContext httpContext, HealthReport healthReport)
    {
        httpContext.Response.ContentType = "application/json";

        var dependencyHealthChecks = healthReport.Entries.Select(entry => new
        {
            Name = entry.Key,
            Status = entry.Value.Status.ToString(),
            DurationInSeconds = entry.Value.Duration.TotalSeconds.ToString("0:0.000", new CultureInfo("en-US")),
            Discription = entry.Value.Description,
            Exception = entry.Value.Exception?.Message,
            Data = entry.Value.Data
        });

        var healthCheckResponse = new
        {
            Status = healthReport.Status.ToString(),
            TotalCheckExecutionTimeInSeconds = healthReport.TotalDuration.TotalSeconds.ToString("0:0.000", new CultureInfo("en-US")),
            DependencyHealthChecks = dependencyHealthChecks,
        };

        var responseString = JsonSerializer.Serialize(healthCheckResponse, new JsonSerializerOptions
        {
            WriteIndented = true,
            DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
        });

        return httpContext.Response.WriteAsync(responseString);
    }
}

public class EndpointHealthChecker : IHealthCheck
{
    private readonly TracingSettings _appSettings;

    public EndpointHealthChecker(IOptionsMonitor<TracingSettings> appSettings)
    {
        _appSettings = appSettings.CurrentValue;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        var baseUrl = _appSettings.OtlpEndpoint;

        try
        {
            var result = await baseUrl.AllowAnyHttpStatus().GetAsync(cancellationToken: cancellationToken);

            return result is { StatusCode: 200 }
                ? HealthCheckResult.Healthy($"Apm available ({baseUrl})")
                : HealthCheckResult.Unhealthy($"Apm not available ({baseUrl})");
        }
        catch (Exception)
        {
            return HealthCheckResult.Degraded($"Exception on call ({baseUrl})");
        }
    }
}

public class AppSettingChecker : IHealthCheck
{
    private readonly AppSettings _appSettings;

    public AppSettingChecker(IOptionsMonitor<AppSettings> appSetting)
    {
        _appSettings = appSetting.CurrentValue;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        try
        {
            return HealthCheckResult.Healthy(null, _appSettings.ToDictionary());
        }
        catch (Exception)
        {
            return HealthCheckResult.Unhealthy("Exception on appSetting");
        }
    }
}

public class SystemMemoryHealthcheck : IHealthCheck
{
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        var client = new MemoryMetricsClient();
        var metrics = client.GetMetrics();
        var percentUsed = 100 * metrics.Used / metrics.Total;
        var status = HealthStatus.Healthy;

        if (percentUsed > 80)
        {
            status = HealthStatus.Degraded;
        }

        if (percentUsed > 90)
        {
            status = HealthStatus.Unhealthy;
        }

        var data = new Dictionary<string, object>(StringComparer.OrdinalIgnoreCase)
        {
            { "Total", metrics.Total },
            { "Used", metrics.Used },
            { "Free", metrics.Free },
        };

        var result = new HealthCheckResult(status, description: null, exception: null, data);

        return await Task.FromResult(result);
    }
}
```


```c#
using System.Diagnostics;
using System.Runtime.InteropServices;

namespace Utility;

public class MemoryMetrics
{
    public double Total;
    public double Used;
    public double Free;
}

public class MemoryMetricsClient
{
    public MemoryMetrics GetMetrics()
    {
        MemoryMetrics metrics;
        if (IsUnix())
        {
            metrics = GetUnixMetrics();
        }
        else
        {
            metrics = GetWindowsMetrics();
        }

        return metrics;
    }

    private bool IsUnix()
    {
        var isUnix = RuntimeInformation.IsOSPlatform(OSPlatform.OSX) ||
                     RuntimeInformation.IsOSPlatform(OSPlatform.Linux);

        return isUnix;
    }

    private MemoryMetrics GetWindowsMetrics()
    {
        var output = "";
        var info = new ProcessStartInfo
        {
            FileName = "wmic",
            Arguments = "OS get FreePhysicalMemory,TotalVisibleMemorySize /Value",
            RedirectStandardOutput = true
        };

        using (var process = Process.Start(info))
        {
            output = process.StandardOutput.ReadToEnd();
        }

        var lines = output.Trim().Split("\n");
        var freeMemoryParts = lines[0].Split("=", StringSplitOptions.RemoveEmptyEntries);
        var totalMemoryParts = lines[1].Split("=", StringSplitOptions.RemoveEmptyEntries);
        var metrics = new MemoryMetrics
        {
            Total = Math.Round(double.Parse(totalMemoryParts[1]) / 1024, 0),
            Free = Math.Round(double.Parse(freeMemoryParts[1]) / 1024, 0)
        };

        metrics.Used = metrics.Total - metrics.Free;

        return metrics;
    }

    private MemoryMetrics GetUnixMetrics()
    {
        var output = "";
        var info = new ProcessStartInfo("free -m")
        {
            FileName = "/bin/bash",
            Arguments = "-c \"free -m\"",
            RedirectStandardOutput = true
        };

        using (var process = Process.Start(info))
        {
            output = process.StandardOutput.ReadToEnd();
        }

        var lines = output.Split("\n");
        var memory = lines[1].Split(" ", StringSplitOptions.RemoveEmptyEntries);

        var metrics = new MemoryMetrics
        {
            Total = double.Parse(memory[1]),
            Used = double.Parse(memory[2]),
            Free = double.Parse(memory[3])
        };

        return metrics;
    }
}
```

ابتدا در scproj باید `Microsoft.NET.Sdk.Worker` را به مقدار کد دوم تغییر دهید.  

```c#
<Project Sdk="Microsoft.NET.Sdk.Worker">
    <PropertyGroup>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
        <UserSecretsId>Reporter-43A646F5-FCF2-4B69-894F-4B2E702DF0D8</UserSecretsId>
        <DockerDefaultTargetOS>Linux</DockerDefaultTargetOS>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="AspNetCore.HealthChecks.Oracle" Version="8.0.1" />
        <PackageReference Include="AspNetCore.HealthChecks.Redis" Version="8.0.1" />
        <PackageReference Include="AspNetCore.HealthChecks.SqlServer" Version="8.0.0" />
    </ItemGroup>
</Project>
```

مقدار جدید : `Microsoft.NET.Sdk.Web`  

```c#
<Project Sdk="Microsoft.NET.Sdk.Web">
    <PropertyGroup>
        <TargetFramework>net6.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
        <UserSecretsId>Reporter-43A646F5-FCF2-4B69-894F-4B2E702DF0D8</UserSecretsId>
        <DockerDefaultTargetOS>Linux</DockerDefaultTargetOS>
    </PropertyGroup>
    <ItemGroup>
        <PackageReference Include="AspNetCore.HealthChecks.Oracle" Version="8.0.1" />
        <PackageReference Include="AspNetCore.HealthChecks.Redis" Version="8.0.1" />
        <PackageReference Include="AspNetCore.HealthChecks.SqlServer" Version="8.0.0" />
    </ItemGroup>
</Project>
```

همچنین در Progam.cs نیز باید `Host.CreateApplicationBuilder` به کد دوم تغییر کند:  

```c#
using System.Globalization;
using Business;
using DataAccess;
using Serilog;

var builder = Host.CreateApplicationBuilder(args);

Log.Logger = new LoggerConfiguration()
    .WriteTo.Console(formatProvider: new CultureInfo("fa-IR"))
    .CreateBootstrapLogger();

Log.Information("Starting up...");

try
{
    var builderConfiguration = builder.Configuration.AddJsonFile("appsettings.json", reloadOnChange: true, optional: false)
        .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")}.json", reloadOnChange: true, optional: true)
        .AddEnvironmentVariables();

    var configuration = builderConfiguration.Build();
    var services = builder.Services;

    services.ConfigureHealthCheck(configuration);

    var host = builder.Build();

    await host.RunAsync();
}
catch (Exception e)
{
    Log.Fatal(e, "An unhandled exception occured during bootstrapping");
    throw;
}
finally
{
    Log.Information("Stopped...");
    
    await Log.CloseAndFlushAsync();
}
```

نمونه تغییر یافته:  
به `WebApplication.CreateBuilder(args)` تغییر پیدا کرده است.  
اکنون در بخش آخر کد می‌توانید به `App` دسترسی داشته باشید.  

```c#
using System.Globalization;
using Business;
using DataAccess;
using Serilog;

var builder = WebApplication.CreateBuilder(args);

Log.Logger = new LoggerConfiguration()
    .WriteTo.Console(formatProvider: new CultureInfo("fa-IR"))
    .CreateBootstrapLogger();

Log.Information("Starting up...");

try
{
    var builderConfiguration = builder.Configuration.AddJsonFile("appsettings.json", reloadOnChange: true, optional: false)
        .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT")}.json", reloadOnChange: true, optional: true)
        .AddEnvironmentVariables();

    var configuration = builderConfiguration.Build();
    var services = builder.Services;

    services.ConfigureHealthCheck(configuration);

    var app = builder.Build();

    app.AddHealthCheck();

    await app.RunAsync();
}
catch (Exception e)
{
    Log.Fatal(e, "An unhandled exception occured during bootstrapping");
    throw;
}
finally
{
    Log.Information("Stopped...");
    
    await Log.CloseAndFlushAsync();
}
```

همچنین اگر در Docker می‌خواهید پابلیش بدهید مقدار زیر را به AppSetting اضافه کنید:  

```csharp
{
  "HostPort": "19999",
}
```

همچنین برای دیباگ بر روی سیستم خود نیز launchSettings.json را نیز با کد زیر جایگزین بکنید:  

```csharp
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "iisSettings": {
    "windowsAuthentication": false,
    "anonymousAuthentication": true,
    "iisExpress": {
      "applicationUrl": "http://localhost:19999;https://localhost:37887"
    }
  },
  "profiles": {
    "CustomerOrderGateway": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "health",
      "environmentVariables": {
        "DOTNET_ENVIRONMENT": "Development",
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:19999;https://localhost:7186"
    },
    "IIS Express": {
      "commandName": "IISExpress",
      "launchBrowser": true,
      "launchUrl": "health",
      "environmentVariables": {
        "DOTNET_ENVIRONMENT": "Development",
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    },
    "Docker": {
      "commandName": "Docker"
    }
  }
}
```

[health check](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-6.0)  