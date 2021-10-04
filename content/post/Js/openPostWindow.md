---
title: "Windowopen開新視窗並Post資料"
description: 點擊開新窗可用post方式至後端並設定視窗大小設定
date: 2021-10-04T14:19:59+08:00
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

## 前言

一般用`window.open`開窗傳遞參數常用`get`方式在`url`後方串接參數  
若想用`post`方式，則在開窗前模擬`form submit post`  
並在`onsubmit`前`open`。  

## 主要內容

```javascript
function openPostWindow(url, name, val1, val2) {

    var tempForm = document.createElement("form");
    tempForm.id = "tempForm1";
    tempForm.method = "post";
    tempForm.action = url; //url
    tempForm.target = name;

    let hideInput = document.createElement("input");
    hideInput.type = "hidden";  
    hideInput.name = "val1";//傳入引數名
    hideInput.value = val1;//傳入資料
    tempForm.appendChild(hideInput);

    let hideInput2 = document.createElement("input");
    hideInput2.type = "hidden";
    hideInput2.name = "val2";
    hideInput2.value = val2;
    tempForm.appendChild(hideInput2);

    tempForm.addEventListener('onsubmit', function () { openWindow(name);})
    document.body.appendChild(tempForm);

    //手動submit form
    tempForm.submit();
    document.body.removeChild(tempForm);

}
function openWindow(name) {
    window.open('about:blank', name);
    //window.open(strUrl, strWindowName, [strWindowFeatures]); strWindowName 和 form 的 target相對應
    //open 還可設定視窗參數如height=400, width=400, top=0, left=0, toolbar=yes, menubar=yes, scrollbars=yes, resizable=yes,location=yes, status=yes
}    
```

## 參考連結

>* [url1](https://www.itread01.com/content/1546525466.html)