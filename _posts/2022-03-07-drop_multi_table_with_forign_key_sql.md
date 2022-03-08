---
title: "پاک کردن تمامی جداول دارای کلید خارجی در SQL Server"
date: 2022-03-07T12:00:00-00:00
categories:
  - SQL
tags:
  - sql
  - forign_key
  - drop
  - table
---

در یکی از بخش‌های جدا سازی و حرکت به سوی ماکروسرویس، نیاز بود تا از صحت عملکرد سیستم اطمینان حاصل کنیم.  
برای این کار سورس‌های سیستم را در یک سرور دیگر اجرا کردیم و برای بخش دیتابیس نیاز داشتیم تا تیبل‌ها و دیگر عناصر اضافی را حذف کنیم و سپس با دیتاهای واقعی سیستم جدا شده را تست کنیم.  
بدین منظور نیاز بود تا بر روی بکاپ دیتابیس اصلی جدول‌های اضافی که دیتای زیادی نیز داشتند را حذف کنیم.  
انجام کار بالا بصورت دستی زمان‌بر بود پس با کمی جستجو و فکر به کوئری زیر رسیدیم که عناصر مورد نیاز ما در آن باقی می‌مانند و دیگر عناصر که شامل جدول‌ها، Function و storedProcedure هست را حذف می‌کند.  
برای کار بالا نیز نیاز به غیر فعال سازی کلیدهای خارجی یا CONSTRAINT است که در کد زیر درنظر گرفته شده است.  


```sql
USE dbMy;

SET NOCOUNT ON;

DECLARE @views TABLE
               (
                   tablename VARCHAR(255)
               );
INSERT INTO @views
SELECT '[' + SCHEMA_NAME(schema_id) + '].[' + name + ']' AS name
FROM sys.views
WHERE SCHEMA_NAME(schema_id) NOT LIKE '%my%';

DECLARE @dropTableSql NVARCHAR(max);
DECLARE @tablename VARCHAR(255);
DECLARE cTable CURSOR FOR
    SELECT tablename
    FROM @views;

OPEN cTable
FETCH NEXT FROM cTable INTO @tableName
WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @dropTableSql = 'DROP VIEW ' + @tablename + '';
        EXEC sp_executesql @dropTableSql;
        FETCH NEXT FROM cTable INTO @tableName
    END
CLOSE cTable
DEALLOCATE cTable

DECLARE @procedures TABLE
                    (
                        tablename VARCHAR(255)
                    );

INSERT INTO @procedures
SELECT '[' + SCHEMA_NAME(schema_id) + '].[' + name + ']' AS name
FROM sys.procedures
WHERE SCHEMA_NAME(schema_id) NOT LIKE '%my%';

DECLARE cTable CURSOR FOR
    SELECT tablename
    FROM @procedures
OPEN cTable
FETCH NEXT FROM cTable INTO @tableName
WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @dropTableSql = 'DROP PROCEDURE ' + @tablename + '';
        EXEC sp_executesql @dropTableSql;
        FETCH NEXT FROM cTable INTO @tableName
    END
CLOSE cTable
DEALLOCATE cTable

DECLARE @tables TABLE
                (
                    tablename VARCHAR(255)
                );
INSERT INTO @tables
SELECT '[' + SCHEMA_NAME(schema_id) + '].[' + name + ']' AS name
FROM sys.tables
WHERE '[' + SCHEMA_NAME(schema_id) + '].[' + name + ']' NOT IN
      (
       '[my.new].[StateArchive]',
       '[my.new].[Request]'
          );

-- Iterate tables, drop FK constraints and tables
DECLARE cTable CURSOR FOR
    SELECT tablename
    FROM @tables;

OPEN cTable
FETCH NEXT FROM cTable INTO @tableName
WHILE @@FETCH_STATUS = 0
    BEGIN
        -- Identify any FK constraints
        DECLARE @fkCount INT;
        SELECT @fkCount = COUNT(*)

        FROM sys.foreign_keys
        WHERE referenced_object_id = OBJECT_ID(@tablename)

        -- Drop any FK constraints from the table
        IF (@fkCount > 0)
            BEGIN
                DECLARE @dropFkSql NVARCHAR(max);
                SET @dropFkSql = '';
                DECLARE @fkName NVARCHAR(max);

                DECLARE cDropFk CURSOR FOR
                    SELECT 'ALTER TABLE [' + OBJECT_SCHEMA_NAME(parent_object_id) + '].[' +
                           OBJECT_NAME(parent_object_id) + '] DROP CONSTRAINT [' + name + ']',
                           name
                    FROM sys.foreign_keys
                    WHERE referenced_object_id = OBJECT_ID(@tablename)
                OPEN cDropFk
                FETCH NEXT FROM cDropFk INTO @dropfksql,@fkName
                WHILE @@FETCH_STATUS = 0
                    BEGIN
                        EXEC sp_executesql @dropFkSql;

                        SELECT 'Dropped FK Constraint: ' + @fkName
                        FETCH NEXT FROM cDropFk INTO @dropfksql,@fkName
                    END
                CLOSE cDropFk
                DEALLOCATE cDropFk
            END

        -- Drop the table
        SET @dropTableSql = 'DROP TABLE ' + @tablename + '';
        EXEC sp_executesql @dropTableSql;

        SELECT 'Dropped table: ' + @tablename

        FETCH NEXT FROM cTable INTO @tableName
    END
CLOSE cTable
DEALLOCATE cTable

ALTER TABLE [test].[User]
    DROP CONSTRAINT IF EXISTS [CK_Test_User(NationalCode)]

DECLARE @functions TABLE
                   (
                       tablename VARCHAR(255)
                   );

INSERT INTO @functions
SELECT '[' + SCHEMA_NAME(schema_id) + '].[' + name + ']' AS name
FROM sys.objects
WHERE type IN ('FN', 'IF', 'TF')
  AND SCHEMA_NAME(schema_id) NOT LIKE '%my%'
  AND name NOT IN
      (
       'FnGetAccessibleUser',
       'FnGetAccessibleUserByOthersParam'
          )

DECLARE cTable CURSOR FOR
    SELECT tablename
    FROM @functions
OPEN cTable
FETCH NEXT FROM cTable INTO @tableName
WHILE @@FETCH_STATUS = 0
    BEGIN
        SET @dropTableSql = 'DROP FUNCTION ' + @tablename + '';
        EXEC sp_executesql @dropTableSql;
        FETCH NEXT FROM cTable INTO @tableName
    END
CLOSE cTable
DEALLOCATE cTable
```

در کوئری بالا موارد `NOT IN` و `NOT LIKE` مواردی است که می‌خواهیم آنها را در سیستم حفظ کنیم.  

کوئری‌های زیر نیز از سطح اینترنت جمع‌آوری شده است:  

```sql
To get all foreign key relationships referencing your table, you could use this SQL (if you're on SQL Server 2005 and up):

SELECT * 
FROM sys.foreign_keys
WHERE referenced_object_id = object_id('Student')
and if there are any, with this statement here, you could create SQL statements to actually drop those FK relations:

SELECT 
    'ALTER TABLE [' +  OBJECT_SCHEMA_NAME(parent_object_id) +
    '].[' + OBJECT_NAME(parent_object_id) + 
    '] DROP CONSTRAINT [' + name + ']'
FROM sys.foreign_keys
WHERE referenced_object_id = object_id('Student')
```


```sql
Here is another way to drop all tables correctly, using sp_MSdropconstraints procedure. The shortest code I could think of:

exec sp_MSforeachtable "declare @name nvarchar(max); set @name = parsename('?', 1); exec sp_MSdropconstraints @name";
exec sp_MSforeachtable "drop table ?";
```


```sql
execute the below code to get the foreign key constraint name which blocks your drop. For example, I take the roles table.

      SELECT *
      FROM sys.foreign_keys
      WHERE referenced_object_id = object_id('roles');

      SELECT name AS 'Foreign Key Constraint Name',
      OBJECT_SCHEMA_NAME(parent_object_id) + '.' + OBJECT_NAME(parent_object_id)
      AS 'Child Table' FROM sys.foreign_keys
      WHERE OBJECT_SCHEMA_NAME(referenced_object_id) = 'dbo'
      AND OBJECT_NAME(referenced_object_id) = 'dbo.roles'
you will get the FK name something as below : FK__Table1__roleId__1X1H55C1

now run the below code to remove the FK reference got from above.

ALTER TABLE dbo.users drop CONSTRAINT FK__Table1__roleId__1X1H55C1;
```


```sql
In SQL Server Management Studio 2008 (R2) and newer, you can Right Click on the

DB -> Tasks -> Generate Scripts

Select the tables you want to DROP.

Select "Save to new query window".

Click on the Advanced button.

Set Script DROP and CREATE to Script DROP.

Set Script Foreign Keys to True.

Click OK.

Click Next -> Next -> Finish.

View the script and then Execute.
```

البته کوئری‌های بالا تست نشده‌اند و فقط کوئری اول که توضیح داده شد تست شده است.  