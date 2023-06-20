---
title: "Oracle使用impdp/expdp來備份"
description:
date: 2023-06-20T15:47:46+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Oracle
tags: [
    "Oracle"
]
---

## 前言

使用Oracle Data Pump來匯入匯出資料庫  
impdp/expdp只能在Server端執行，Client端不可用

## 匯出Impdp

`登入sqlplus建立目錄並授權`
```bash
SQL: sqlplus /nolog
: connect sys/password@localhost/xxx as sysdba
: create directory 資料夾定義名稱 as '資料夾路徑'
>>>建立目錄
>>>查詢目錄
: select * from dba_directories
>>>授權
: GRANT READ,WRITE ON DIRECTORY 資料夾定義名稱 TO 使用者;
>>>匯出
: expdp sys/password SCHEMAS=使用者 DIRECTORY=資料夾定義名稱 DUMPFILE=oooooo.dmp : : : LOGFILE=oooo.log
```

## 匯入impdp

```bash 
SQL: impdp 帳號/密碼 DIRECTORY=資料夾定義名稱 DUMPFILE=dmp檔案
: content=data_only只匯資料 
: remap_schema=A:B 從A schema改到B
: table_exists_action=table存在的處置
```

## 小結

實際在操作中會有不同情況需要調整指令  
本次只嘗試data_only的方式，把資料匯進不同db

## 參考連結

>* [url1](https://www.cnblogs.com/promise-x/p/7477360.html)