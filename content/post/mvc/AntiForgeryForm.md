---
title: "CSRF-AntiForgeryToken(ajax)"
description: "包含Ajax處理"
date: 2022-01-26T16:39:45+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - CSRF
tags: [
    "MVC",
    "Ajax",
]
---

## 前言

為了防止CSRF攻擊，再提交表單時通常會使用`AntiForgeryToken`  
但在使用Ajax Post時也要注意

## 主要內容

可以在頁面上藏一個`有token的form`  
```html
<form id="__AjaxAntiForgeryForm" action="#" method="post">@Html.AntiForgeryToken()</form>  
```
再post之前將token塞進去。
```javascript
var AddAntiForgeryToken = function (data) {
        data.__RequestVerificationToken = $('#__AjaxAntiForgeryForm input[name=__RequestVerificationToken]').val();
        return data;
    };
    $.ajax({
            type: "post",
            url: '',
            data: AddAntiForgeryToken({id:"1"}),
            success: function (response) {
                // ....
            }
        });   
```

## 參考連結

>* [url1](https://stackoverflow.com/questions/4074199/jquery-ajax-calls-and-the-html-antiforgerytoken)