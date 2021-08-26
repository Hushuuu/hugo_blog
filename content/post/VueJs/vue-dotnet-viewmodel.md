---
title: ".Net MVC ViewModel 套上VueJs"
description: 將ViewModel序列化為JSON字串後綁定進Vue
date: 2021-08-26T14:48:29+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - VueJs
tags: [
    "MVC",
    "VueJs",
]
---

## 前言

版本`vue2.x`  
將`ViewModel`序列化後也可綁定到`Vue`物件上  

## 主要內容

在`cshtml`內定義好`model`後序列化  
用`Json.Encode`或是`JsonConvert`處理後  
回傳`MvcHtmlString`類別  
```c#
@{
    var Datajson = new MvcHtmlString(Json.Encode(Model));
}
```
建立`Vue`實體，綁定`Element`  
資料`data`定義一個`dataList`接收剛剛序列化的變數  
```javascript
var vueDiv = new Vue({
        el: "#vueDiv",
        data: {
            dataList:@Datajson,
        },
    });
```
`v-if 沒筆數就不顯示`  
`v-for 迴圈印出datList每個項目`
```html
<table v-if="dataList.count>0">
    <thead>
        <tr>
            <th>標題</th>
            <th>名稱</th>
        </tr>
    </thead>
    <tbody>
        <tr v-for="(item,index) in dataList" :id="item.id">
            <td>{{item.title}}</td>
            <td>{{item.name}}</td>
        </tr>
    </tbody>
</table>
```