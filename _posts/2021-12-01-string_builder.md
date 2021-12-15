---
title: "استفاده از StringBuilder در .net"
date: 2021-12-01T10:49:00-00:00
categories:
  - Net
tags:
  - net
  - stringBuilder
  - core
---

```c#
using System;
using System.Text;

public class Example
{
    public static void Main()
    {
        var sb = new StringBuilder();
        ShowSbInfo(sb);

        sb.Append("aaaaa");
        ShowSbInfo(sb);

        for (var ctr = 0; ctr <= 10; ctr++)
        {
            sb.Append("bbbbbbbbbbbbbbb");
            ShowSbInfo(sb);
        }

        Console.WriteLine("------------------");
        Console.WriteLine(sb.Capacity);

        sb.EnsureCapacity(240);
        Console.WriteLine(sb.Capacity);

        sb.EnsureCapacity(280);
        Console.WriteLine(sb.Capacity);

        sb.EnsureCapacity(int.MaxValue);
        Console.WriteLine(sb.Capacity);

        Console.WriteLine("------------------");
        Console.WriteLine(sb.MaxCapacity);
        Console.WriteLine(sb.Length);

        Console.WriteLine("------------------");
        var chunk = sb.GetChunks();
        foreach (var item in chunk)
        {
            Console.WriteLine(item);
        }

        Console.WriteLine("------------------");
        var sb2 = new StringBuilder(10, 30);
        sb2.Append("aaaaa");
        sb2.Append("aaaaa");
        sb2.Append("aaaaa");
        sb2.Append("aaaaaaaaaa");
        sb2.Append("aaaaaaaaaa");
        //sb2.Append("aaaaaaaaaa");
        Console.WriteLine(sb2.Capacity);
        Console.WriteLine(sb2.Length);
    }

    private static void ShowSbInfo(StringBuilder sb)
    {
        foreach (var prop in sb.GetType().GetProperties())
        {
            if (prop.GetIndexParameters().Length == 0)
                Console.Write("{0}: {1:N0}    ", prop.Name, prop.GetValue(sb));
        }

        Console.WriteLine();
    }
}
```

اطلاعات بیشتر:  

[stringBuilder](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder?view=net-6.0)  

[getChunks](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder.getchunks?view=net-6.0)  