---
title: "پاس دادن Enum به عنوان ورودی به HTTP GET"
date: 2022-01-23T09:00:00-00:00
categories:
  - Net
tags:
  - net
  - enum
  - httpget
  - required
---



```c#
[HttpGet]
public async Task<ApiResult<List<ResultVm>>> GetPublicWatchItems(MyEnum watch)
```

```c#
public enum MyEnum
{
	[Description("بیشترین تاثیر")]
	MaxIndexAffect = 1,

	[Description("بیشترین حجم")]
	MaxVolumeAmount = 2,
}
```

```c#
api/MyController/GetPublicWatchItems?id=2
api/MyController/GetPublicWatchItems?watch=0
```

```c#
[HttpGet]
public async Task<ApiResult<List<ResultVm>>> GetPublicWatchItems([Required] MyEnum watch)
```

```c#
[HttpGet("{watch}")]
public async Task<ApiResult<List<ResultVm>>> GetPublicWatchItems(MyEnum watch)
```

```c#
api/MyController/GetPublicWatchItems/2
```