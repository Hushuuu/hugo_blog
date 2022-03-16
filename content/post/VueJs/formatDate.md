---
title: "VueJs 處理Json格式下日期Date"
description: "/Date(1373950800000)/ 的處理"
date: 2021-08-26T16:37:23+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - VueJs
tags: [
    "VueJs",
    "Format"
]
---

## 前言

`Json Serilize`後`Datetime`格式被轉為`/Date(1373950800000)/`  

## 轉換處理

```javascript
//傳入轉換字串和是否要含時間
function FormateDate(str,withTime){
    var d = eval(`new ` + str.substr(1, str.length - 2));
    var ar_date = [d.getFullYear(), d.getMonth() + 1, d.getDate()];
    var ar_time = [d.getHours(), d.getMinutes(), d.getSeconds()];
    for (var i = 0; i < ar_date.length; i++) ar_date[i] = dFormat(ar_date[i]);
    for (var i = 0; i < ar_time.length; i++) ar_time[i] = dFormat(ar_time[i]);
    return withTime == true ? ar_date.join(`-`) + " " + ar_time.join(`: `) : ar_date.join(`-`);
}
function dFormat(i) 
{ 
    return i < 10 ? "0" + i.toString() : i; 
}
//Using
let str = "/Date(1373950800000)/";
let result  = FormateDate(str,true); //yyyy-MM-dd HH:mm:ss
let result2 = FormateDate(str); //yyyy-MM-dd
```

## Vue擴充
可使用`Vue.prototype`的方式直接擴充一個方法就不用每次建立實例時要額外多寫一段  
```javascript
Vue.prototype.$formateDate = function (str, withTime) {
    var d = eval(`new ` + str.substr(1, str.length - 2));
    var ar_date = [d.getFullYear(), d.getMonth() + 1, d.getDate()];
    var ar_time = [d.getHours(), d.getMinutes(), d.getSeconds()];
    for (var i = 0; i < ar_date.length; i++) ar_date[i] = dFormat(ar_date[i]);
    for (var i = 0; i < ar_time.length; i++) ar_time[i] = dFormat(ar_time[i]);
    return withTime == true ? ar_date.join(`-`) + " " + ar_time.join(`: `) : ar_date.join(`-`);

    function dFormat(i) { return i < 10 ? "0" + i.toString() : i; }
};
//using vue.$formatDate()
```


## 小結

可以直接將轉換`function`一起包進`Vue`的實例的`methods`中  
就能跟著畫面渲染轉換。

* 若有傳回後端接收的情形，也要注意轉換格式讓`Datetime`型別辨識並綁定的到  


## 參考連結

>* [url1](https://www.itread01.com/content/1509993517.html)