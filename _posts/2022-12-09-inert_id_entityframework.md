---
title: "Insert کردن Id در EntityFrameWork"
categories:
  - Net
tags:
  - net
  - identity
  - entity_frame_work
---

اگر در دیتابیس خود جدولی دارید که ستون آن `Identity` نیست و باید بصورت دستی وارد شود، برای فعال کردن این قابلیت در EntityFrameWork کافی است موارد زیر را به کانفیگ آن جدول اضافه کنید:  

```csharp
public MyEntityConfig()
{
    ToTable("MyEntity", "dbo");
    HasKey(x => x.Id);
    Property(x => x.Id).IsRequired();
    Property(x => x.Id).HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);
}
```

و یا:  

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

public class MyEntity
{
    [Key, Required]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public int Id { get; set; }

    public DateTime StartDate { get; set; }

    p
```