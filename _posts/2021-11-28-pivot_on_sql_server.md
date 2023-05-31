---
title: "استفاده از pivot در SQL Server"
categories:
  - SQL
tags:
  - sql
  - pivot
  - sql_server
---



```sql
CREATE TABLE Grades
(
    [Student] NVARCHAR(50),
    [Subject] NVARCHAR(50),
    [Marks]   INT
)
GO

INSERT INTO Grades
VALUES ('ali', 'Mathematics', 20),
       ('ali', 'Science', 16),
       ('ali', 'Geography', 18),
       ('mohammad', 'Mathematics', 10),
       ('mohammad', 'Science', 18),
       ('mohammad', 'Geography', 17)
GO
```

```sql
ALTER TABLE Grades
    ADD
        DateOfEvent datetime2 NULL
```

```sql
INSERT INTO Grades (Student, Subject, Marks, DateOfEvent)
VALUES ('ali', 'Mathematics', 14, GETDATE())
```


```sql
CREATE PROCEDURE dbo.DynamicPivotTableInSql @ColumnToPivot NVARCHAR(255),
                                            @ListToPivot   NVARCHAR(255)
AS
BEGIN
    SELECT *
    FROM (SELECT [Student],
                 [Subject],
                 [Marks]
          FROM Grades
         ) StudentResults
             PIVOT (
             AVG([Marks])
             FOR @ColumnToPivot
             IN (@ListToPivot)
             ) AS PivotTable
END
```
