---
title: "Oracle function select效能"
description: "oracle function子查詢效能差異"
date: 2022-12-18T17:50:19+08:00
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

使用Oracle function的時候，使用的方式的效能差異，先說結論是`子查詢`會快上許多。

#### 直接call function
舉例有一個function `funcA`傳入`tableA`表的`col1,col2`欄位比大小，左邊大回傳1否則2，找`col2>col1`的結果集
```SQL
select * from tableA where funcA(col1,col2)=2 
```

#### 子查詢  
```SQL
select * from tableA where (select funcA(col1,col2) from dual)=2 
```

#### 效能
根據參考文章，可以加快效能的原因是，Oracle 會為 SELECT 出現的 Scalar Subquery (只傳回單一值的子查詢) 在記憶體建立一個 Hashtable，整理不同參數與查詢結果的對應表。若參數欄位值先前出現過，即可直接由 Hashtable 取值不用重新計算。


## 參考連結

>* [url1](https://blog.darkthread.net/blog/oracle-scalar-subquery-caching/)