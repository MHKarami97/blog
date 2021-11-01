---
title: "مپ کردن List در AutoMapper"
date: 2021-11-01T09:36:00-00:00
categories:
  - Net
tags:
  - net
  - automapper
  - list
---

یکی از کتابخانه هایی که کار تبدیل مدل ها در برنامه نویسی را راحت می‌کند، کتابخانه Automapper است که با استفاده از آن دیگر لازم نیست Atribute های یک مدل را بصورت تک تک به یک مدل دیگر پاس بدهیم.  
یکی از کاربردهای این کتابخانه در زمان هایی هست که شما می‌خواهید عملیات تبدیل را بر روی یک لیست انجام بدهید.  
در صورتی که بخواهید این عملیات را خودتان انجام بدهید، نیاز است که از حلقه ها در برنامه استفاده کنید اما این کتابخانه این کار را راحت می‌کند.  

ابتدا نیاز است که مدل های خود را به کتابخانه معرفی کنید.  
دقت کنید که لازم نیست در معرفی مدل ها از `List` استفاده کنید و فقط همان خود مدل کفایت می‌کند.  

```c#
CreateMap<SearchVm, SearchDto>();
CreateMap<SearchResultDto, SearchResultVm>();
```

اکنون در محل هایی که نیاز به مپ کردن دیتا دارید کافی است بصورت زیر عمل کنید.  
دقت کنید که در مپ دوم یک `List` را به یک `List` مپ کرده ایم که اینکار توسط خود کتابخانه انجام می‌شود.  

```c#
public class MyController : Controller
{
    private readonly IMapper _mapper;
    private readonly AppSettings AppSettings;

    public BaseController(IMapper mapper,
        IOptions<AppSettings> appSettings)
    {
        _mapper = mapper;
        AppSettings = appSettings.Value;
    }

    [HttpGet]
    public async Task<IActionResult> Search(SearchVm input)
    {
        var data = _mapper.Map<SearchDto>(input);

        var resultData = await _myService.Search(data);

        var result = _mapper.Map<List<SearchResultVm>>(resultData);

        return Ok(result);
    }
}
```

اطلاعات بیشتر:  

[automapper](https://docs.automapper.org/en/stable/Lists-and-arrays.html)  