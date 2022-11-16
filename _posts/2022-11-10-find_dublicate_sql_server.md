---
title: "پیدا کردن موارد تکراری در دیتابیس SQL Server"
date: 2022-11-10T00:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - having
  - grouped
---

توسط کوئری زیر می‌توانید مواردی که در یک جدول بیشتر از یکبار تکرار شده‌اند را پیدا کنید.  
ابتدا توسط `Group By` مواردی که می‌خواهیم تکراری بودن را بر روی آنها تست کنیم بدست می‌آوریم و سپس توسط `Having` شرط بیشتر از یک بار را بر روی آنها اعمال می‌کنیم

```sql
SELECT rr.MessageId,
       rr.GapId,
       rr.MessageType,
       COUNT(*) AS CountData
FROM dbo.RawMessage rr
GROUP BY rr.MessageId, rr.GapId, rr.MessageType
HAVING COUNT(*) > 1
```

حال اگر نیاز داشتید تا اطلاعات بیشتر این ستون‌ها را هم بدست آورید می‌توانید بصورت زیر عمل کنید:  

```sql
SELECT r.Id,
       r.MessageId,
       r.GapId,
       r.EngineId,
       r.MessageType,
       CAST(r.DbEntryDateTime AS TIME) AS DbEntryTime
FROM (SELECT rr.MessageId,
             rr.GapId,
             rr.MessageType,
             COUNT(*) AS CountData
      FROM dbo.RawMessage rr
      GROUP BY rr.MessageId, rr.GapId, rr.MessageType
      HAVING COUNT(*) > 1) ii
         INNER JOIN dbo.RawMessage r
                    ON r.MessageId = ii.MessageId AND r.MessageType = ii.MessageType AND r.GapId = ii.GapId
```