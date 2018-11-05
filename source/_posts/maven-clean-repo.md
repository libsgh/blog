---
title: 脚本清理本地仓库未下载完成的ja包
categories: java
date: 2018-9-27 19:37:56
tags: 
     - java
     - maven
comments: true
---

> 新建文本文档另存为<code>.bat</code>文件

```bash

@echo off
  
rem 这里写你的仓库路径
set REPOSITORY_PATH=D:\repository
rem 正在搜索...
for /f "delims=" %%i in ('dir /b /s "%REPOSITORY_PATH%\*lastUpdated*"') do (
    del /s /q %%i
)
rem 搜索完毕
pause
```