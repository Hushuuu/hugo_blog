---
title: "Table跨行sticky"
description: "處理table header rowspan sticky問題"
date: 2023-09-11T08:51:28+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - css
tags: [
    "css",
    "javascript",
]
---

## 前言

一般table標頭固定上方使用sticky方式  
但遇到有跨行合併儲存格的情況  
就會有跑版的情形  

## 主要內容

使用`ele.getBoundingClientRect()`  
來取得元素「相對於視窗」的座標  
重新計算`th`所需的`top`值  
```javascript
//
let tableHeaderTop = document.querySelector('table thead').getBoundingClientRect().top;
let ths = document.querySelectorAll('table thead th')

for (var i = 0; i < ths.length; i++) {
    var th = ths[i];
    th.style.top = th.getBoundingClientRect().top - tableHeaderTop + "px";
}
```

## 小結

來源應該是stackoverflow