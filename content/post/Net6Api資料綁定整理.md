---
title: "Net6Api資料綁定整理"
description:
date: 2023-08-07T15:27:57+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Net6
tags: [
    "Net6",
    "WebApi",
    "Json"
]
---

## 前言

Net6在開發web api時，有時候會遇到後端沒接到值的情況  
整理幾個常用的使用方法

## 主要內容

都以Post為例  
自訂Data模型參數，預設是json的方式
```C#
//這兩種寫法相等
public string Post(A vm);
public string Post([FromBody] A vm);
//form-data
public string Post([FromForm] A vm);
//x-www-urlencoded需要特別加屬性
[Consumes("application/x-www-form-urlencoded")]
public string Post([FromForm] A vm);
```
如果是用基本型別的參數的話有特別的情況  
```C#
//預設是queryString 這兩種相等
public string Post(string str);
public string Post([FromQuery] string str);
//json **特別不同
public string Post([FromBody] string str);
//在請求時必須只傳入"\"value\""
//因為net6 在model binding時使用Text.Json
//如果試著反序列化就會拋錯
//失敗` The JSON value could not be converted to System.String.`
JsonSerializer.Deserialize<string>("{\"str\":\"value\"}");
//正確
JsonSerializer.Deserialize<string>("\"value\"");

//from-data&form-urlencoded方式和上面自訂型別的一樣
```

## 小結

除了參數少的`QueryString`直接寫多個參數  
其他簡單一點就是全部都定義類別來接值  
就可以解決大部分的情況  
有些問題在之前.netFramework是不會遇到的  
應該與`Text.Json`相關(.netFramework是使用`Json.NET`)  
所以開發上要留意一下
 
  
 