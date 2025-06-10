---
title: "EF Core Tool反向工程"
description:
date: 2025-04-21T09:15:27+08:00
image:
draft: false
license:
hidden: false
comments: true
categories:
  - EF
tags: ["EF"]
---

## 前言

## 主要內容

建立一個`NET8 Console`專案，安裝這兩個套件

```
Microsoft.EntityFrameworkCore.Design
Oracle.EntityFrameworkCore 5.0.21
```

oracle 版本太舊的話，裝的版本要低一點  
oracle 11.2c => `Microsoft.EntityFrameworkCore.Design 5.0.17` + `Oracle.EntityFrameworkCore 5.0.21`

```command
dotnet ef dbcontext scaffold "DATA SOURCE=192.168.0.220:1521/xe;PERSIST SECURITY INFO=True;USER ID=XXX; password=XXX;" Oracle.EntityFrameworkCore --output-dir Models --table "CUBADM.PU_WORKER"  -f --data-annotations --use-database-names
```

--output-dir 輸出資料夾  
--table 指定 table  
--schema 指定 schema  
--use-database-names 使用 DB 的命名，不改成 C#的預設命名
--data-annotations 類別屬性會註記長度必填這種，不用則是定義在 Context
-f 資料夾裡面有東西 就要加-f 才蓋的過去

## 小結

## 參考連結

> - [url1](https://)
> - [url2]()
