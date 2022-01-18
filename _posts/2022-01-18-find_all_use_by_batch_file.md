---
title: "پیدا کردن تمام خطوط استفاده شده در فایل‌ها توسط Bath script"
date: 2022-01-15T09:00:00-00:00
categories:
  - Trick
tags:
  - bat
  - windows
  - batch
  - script
---



```batch
@ECHO OFF

for /f "delims=" %%x in (%1) do (
	echo:
	echo SERVICE NAME:  %%x 
	echo:
	d:
	cd d:\Workspaces\folder\other
	findstr /S /M %%x *.cs 2>nul | findstr /V myFile.cs | findstr /V myFolder
)
c:
cd c:\Users\user\Desktop
```

[microsoft](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)  