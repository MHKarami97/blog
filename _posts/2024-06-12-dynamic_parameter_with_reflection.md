---
title: "ساخت DynamicParameter توسط Reflection"
categories:
  - Net
tags:
  - dapper
  - sql
  - parameter
---

در زمان‌های که می‌خواهید یک SP را در دیتابیس فراخوانی کنید در بیشتر مواقع نیاز است پارامتر‌های خود را به آن پاس بدهید.  در Dapper برای این کار DynamicParameters وجود دارد.  
پر کردن دستی این مورد هم زمانبر و هم احتمال خطا دارد. توسط کد زیر می‌توانید به راحتی پارامترهای کلاس خود را به آبجکت مورد نظر تبدیل کنید.  

```c#
public DynamicParameters ConvertToDynamicParameters()
{
    var parameters = new DynamicParameters();
    var properties = GetType().GetProperties(BindingFlags.Public | BindingFlags.Instance);

    foreach (var property in properties)
    {
        var name = property.Name;
        var value = property.GetValue(this);

        if (property.PropertyType == typeof(Enum) && value is not null)
        {
            value = value.GetHashCode();
        }

        if (property.PropertyType.IsGenericType &&
            property.PropertyType.GetGenericTypeDefinition() == typeof(IReadOnlyCollection<>)
            && value is not null)
        {
            var collection = (IReadOnlyCollection<object>)value;

            var str = new StringBuilder();

            foreach (var item in collection)
            {
                str.Append(item)
                    .Append(',');
            }

            value = str.ToString().TrimEnd(',');
        }

        parameters.Add(name, value);
    }

    return parameters;
}
```
اکنون از کد بالا می‌توانید به صورت زیر استفاده کنید.  

```csharp
public async Task<long> GetDataAsync(CustomerModel model)
{
    using (Tracing.ActivitySource.StartActivity())
    {
        using var connection = DbContext.CreateConnection();

        var parameters = model.ConvertToDynamicParameters();

        var result = await connection.QuerySingleAsync<long>($"{OmsSchema}.{DbItem.GeyQuery}", parameters, commandType: CommandType.StoredProcedure);

        return result;
    }
}
```

