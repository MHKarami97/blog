---
title: "پاک کردن تمام فایل‌های Bin و Obj پروژه"
date: 2022-04-21T12:00:00-00:00
categories:
  - Net
tags:
  - net
  - bin
  - obj
  - bat
---

گاهی مواقع که نیاز به انتقال فایل‌های پروژه و یا فشرده سازی آنها است، برای کاهش حجم پاک کردن فولدرهای `bin` و `obj` از حجم نهایی می‌کاهد.  
توسط کد زیر می‌توانید این فولدرهای را پروژه خود پاک کنید.  


```bat
@ECHO off
cls

ECHO Deleting all BIN and OBJ folders...
ECHO.

FOR /d /r . %%d in (bin,obj) DO (
	IF EXIST "%%d" (		 	 
		ECHO %%d | FIND /I "\node_modules\" > Nul && ( 
			ECHO.Skipping: %%d
		) || (
			ECHO.Deleting: %%d
			rd /s/q "%%d"
		)
	)
)

ECHO.
ECHO.BIN and OBJ folders have been successfully deleted. Press any key to exit.
pause > nul
```

کافی است فایل بالا را بصورت `.bat` ذخیره کنید و آن را در Root پروژه قرار دهید. با اجرا این فایل، فولدرهای گفته شده پاک می‌شوند.  