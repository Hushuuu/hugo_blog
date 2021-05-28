---
title: "ajax在NetCore Razor Page得到回應bad request"
description: "net core預設有檢查ValidateAntiForgeryToken，沒設定好亂傳post會得到400 BAD REQUEST"
date: 2021-05-27T15:52:39+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "CSRF",
    "RazorPage",
]
---

## 前言

在使用Razor Page時，想傳入post請求卻得到`bad request`  
應該是在net core下有預設檢查`ValidateAntiForgeryToken`來防止跨站攻擊  
故需要在POST前加上token來通過檢查

## 主要內容

### 可利用HtmlHelper直接產生出token
```c#
@Html.AntiForgeryToken()
```
### 在ajax中加入token
```javascript
 $.ajax({
        type: "POST",
        url: "",
        data: "",
        dataType: "json",
        beforeSend: function (xhr) {
            xhr.setRequestHeader("requestverificationtoken",
                $('input:hidden[name="__RequestVerificationToken"]').val());
        },
        success: function (response) {
        }
    });    
```

## 小結

另外`Razor Pages` 是透過 `?handler` 決定呼叫哪一段程式  
只要在 `AJAX URL` 加上 `?handler=MethodName`  
`Razor Page cs`檔寫 `OnGetMethodName()` 或 `OnPostMethodName()` 即可辨識要進哪個方法


## 參考連結

>* [url1](https://demo.tc/post/.net%20core%20csrf%20%E9%98%B2%E7%AF%84%E6%A9%9F%E5%88%B6%E9%81%87%E4%B8%8A%20ajax)
>* [url2](https://blog.darkthread.net/blog/razor-pages-ajax-call/)