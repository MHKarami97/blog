---
title: "مجموعه ها و ساختارهای داده در .net"
date: 2021-05-11T23:00:00-00:00
categories:
  - net
tags:
  - collection
  - dataStructures
  - dotnet
---

داده های هم شکل و مشابه، در زمان ویرایش یا ذخیره سازی، با کارایی بیشتری قالب مدیریت هستند.  
در زبان .net و دیگر زبان های برنامه نویسی، کالکشن یا مجموعه های مختلفی برای مدیریت داده های هم شکل وجود دارد.  
ساختار های داده به دو صورت کلی Generic و Non-Generic وجود دارند. مورد اول بر خلاف مورد دوم Type-Safe می‌باشد، پس عملکرد بهتری دارد.  


# نوشته شود


|دلیل|نوع Generic|نوع Non-Generic|نوع Thread-safe|
|-|-|-|-|
|ذخیره بصورت key/value با قابلیت جستجو سریع بر اساس کلید|Dictionary|Hashtable|ConcurrentDictionary<br /><br /> ReadOnlyDictionary<br /><br /> ImmutableDictionary|
|دسترسی بر اساس index|List|Array<br /><br /> ArrayList|ImmutableList<br /><br /> ImmutableArray|
|استفاده بصورت FIFO|Queue|Queue|ConcurrentQueue<br /><br /> ImmutableQueue|
|استفاده بصورت LIFO|Stack|Stack|ConcurrentStack<br /><br /> ImmutableStack|
|دسترسی به ترتیب به آیتم ها|LinkedList|پیشنهاد نمی شود|پیشنهاد نمی شود|
|دریافت نوتیفیکیشن در زمان افزودن و حذف|ObservableCollection|پیشنهاد نمی شود|پیشنهاد نمی شود|
|داده های مرتب شده|SortedList|SortedList|ImmutableSortedDictionary<br /><br /> ImmutableSortedSet|
|مجموعه ای بر اساس توابع ریاضی|HashSet<br /><br /> SortedSet|پیشنهاد نمی شود|ImmutableHashSet<br /><br /> ImmutableSortedSet|


# پیچیدگی زمانی

| متغیر                   | هزینه استهلاک  | بدترین زمان                | غیرقابل تغییر                          | پیچیدگی زمانی |
|---------------------------|------------|---------------------------|------------------------------------|------------|
| `Stack<T>.Push`           | O(1)       | O(`n`)                    | `ImmutableStack<T>.Push`           | O(1)       |
| `Queue<T>.Enqueue`        | O(1)       | O(`n`)                    | `ImmutableQueue<T>.Enqueue`        | O(1)       |
| `List<T>.Add`             | O(1)       | O(`n`)                    | `ImmutableList<T>.Add`             | O(log `n`) |
| `List<T>.Item[Int32]`     | O(1)       | O(1)                      | `ImmutableList<T>.Item[Int32]`     | O(log `n`) |
| `List<T>.Enumerator`      | O(`n`)     | O(`n`)                    | `ImmutableList<T>.Enumerator`      | O(`n`)     |
| `HashSet<T>.Add`, lookup  | O(1)       | O(`n`)                    | `ImmutableHashSet<T>.Add`          | O(log `n`) |
| `SortedSet<T>.Add`        | O(log `n`) | O(`n`)                    | `ImmutableSortedSet<T>.Add`        | O(log `n`) |
| `Dictionary<T>.Add`       | O(1)       | O(`n`)                    | `ImmutableDictionary<T>.Add`       | O(log `n`) |
| `Dictionary<T>` lookup    | O(1)       | O(1) – or strictly O(`n`) | `ImmutableDictionary<T>` lookup    | O(log `n`) |
| `SortedDictionary<T>.Add` | O(log `n`) | O(`n` log `n`)            | `ImmutableSortedDictionary<T>.Add` | O(log `n`) |

مبنع:  
[https://docs.microsoft.com/en-us/dotnet/standard/collections/](https://docs.microsoft.com/en-us/dotnet/standard/collections/)  
