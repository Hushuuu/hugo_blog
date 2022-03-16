---
title: "Js Note"
description: "未整理JS note"
date: 2021-10-13T14:53:29+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Javascript
tags: [
    "Javascript",
]
---

## Closure 閉包

和作用域有關係，可用來避免變數汙染  
或讓變數不被隨意存取改變  
用jQuery可簡易達成  
```javascript
$(function(){
    //do your coding
});

$(document).ready(function(){

});
```

### 轉千分位

```javascript
/**
 * 轉千分位
 * @param {any} num
 */
function toCurrency(num) {
    var parts = num.toString().split('.');
    parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ',');
    return parts.join('.');
}
```
### Date擴充
```JAVASCRIPT
//增加日期
Date.prototype.addDays = function (days) {
    var date = new Date(this.valueOf());
    date.setDate(date.getDate() + days);
    return date;
}
//轉為yyyy-MM-dd
Date.prototype.toInputString = function () {
    let date = new Date(this.valueOf());
    return date.toISOString().split('T')[0];
}
```