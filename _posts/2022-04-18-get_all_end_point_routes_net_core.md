---
title: "دریافت تمام EndPoint‌ها در برنامه .Net Core"
categories:
  - Net
tags:
  - net_core
  - routes
  - endpoint
  - net
---

در .Net core با معرفی Minimal API, Controllers, Razor Pages, gRPC, Health checks و دیگر موارد، Route های سیستم افزایش پیدا کرده‌اند و با بزرگ شدن پروژه نیاز به دانستن تمام آن‌ها احساس می‌شود.  
توسط کدهای زیر می‌توانید تمام آنها را در آدرس مشخص شده مشاهده کنید.  
بطور مثال کد اول فقط آدرس‌ها را در `http://localhost/debug/routes` نشان می‌دهد.  
کد دوم نیز آدرس‌ها را به همراه جزئیات بیشتر نشان می‌دهد.  
تکه کد سوم نیز در آدرس `http://localhost/-/info/endpoints` موارد گفته شده را بصورت Json نشان می‌دهد.  


```c#
if (app.Environment.IsDevelopment())
{
    app.MapGet("/debug/routes", (IEnumerable<EndpointDataSource> endpointSources) =>
        string.Join("\n", endpointSources.SelectMany(source => source.Endpoints)));
}
```

کد دوم:  

```c#
if (app.Environment.IsDevelopment())
{
    _ = app.MapGet("/debug/routes", (IEnumerable<EndpointDataSource> endpointSources) =>
    {
        var sb = new StringBuilder();
        var endpoints = endpointSources.SelectMany(es => es.Endpoints);
        foreach (var endpoint in endpoints)
        {
            if (endpoint is RouteEndpoint routeEndpoint)
            {
                _ = routeEndpoint.RoutePattern.RawText;
                _ = routeEndpoint.RoutePattern.PathSegments;
                _ = routeEndpoint.RoutePattern.Parameters;
                _ = routeEndpoint.RoutePattern.InboundPrecedence;
                _ = routeEndpoint.RoutePattern.OutboundPrecedence;
            }

            var routeNameMetadata = endpoint.Metadata.OfType<RouteNameMetadata>()
                .FirstOrDefault();
            _ = routeNameMetadata?.RouteName;

            var httpMethodsMetadata = endpoint.Metadata.OfType<HttpMethodMetadata>().FirstOrDefault();
            _ = httpMethodsMetadata?.HttpMethods;

            _ = sb.Append(string.Join("\n", endpoint));
        }

        return sb.ToString();
    });
}
```

کد سوم:  

```c#
[Route("/-/{controller}")]
public class InfoController : Controller
{
    private readonly IEnumerable<EndpointDataSource> _endpointSources;

    public InfoController(
        IEnumerable<EndpointDataSource> endpointSources
    )
    {
        _endpointSources = endpointSources;
    }

    [HttpGet("endpoints")]
    public async Task<ActionResult> ListAllEndpoints()
    {
        var endpoints = _endpointSources
            .SelectMany(es => es.Endpoints)
            .OfType<RouteEndpoint>();
        var output = endpoints.Select(
            e =>
            {
                var controller = e.Metadata
                    .OfType<ControllerActionDescriptor>()
                    .FirstOrDefault();
                var action = controller != null
                    ? $"{controller.ControllerName}.{controller.ActionName}"
                    : null;
                var controllerMethod = controller != null
                    ? $"{controller.ControllerTypeInfo.FullName}:{controller.MethodInfo.Name}"
                    : null;
                return new
                {
                    Method = e.Metadata.OfType<HttpMethodMetadata>().FirstOrDefault()?.HttpMethods?[0],
                    Route = $"/{e.RoutePattern.RawText.TrimStart('/')}",
                    Action = action,
                    ControllerMethod = controllerMethod
                };
            }
        );
        
        return Json(output);
    }
}
```