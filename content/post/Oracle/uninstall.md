---
title: "完整刪除Oracle"
description:
date: 2021-11-23T15:07:03+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Oracle
tags: [
    "Oracle",
]
---

## 前言

包含環境參數，Regedit內機碼，一併刪除。  
版本為19c

#### 關閉服務
`Services 和Oracle相關服務都關閉`  
`Universal Installer解除安裝或是在19cs內deinstall `  
#### egedit刪除Oracle開頭的項目  
`HKEY_LOCAL_MACHINE\SOFTWARE\`   `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services `  
`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\`
`HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\`  
`HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Oracle*`  

#### 刪除環境變數  
`ORACLE_HOME，TNS_ADMIN，NLS_LANG`  
#### 刪除C槽Oracle相關目錄
`Program Files\，Program Files(x86)\，ProgramData\Microsoft\Windows\Start Menu\Programs\ `   
#### 重新開機
重新開機  

## 參考連結

>* [url1](https://www.pianshen.com/article/69701376707/)
>* [url2](https://blog.csdn.net/Amelia__Liu/article/details/84314782)