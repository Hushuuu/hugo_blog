---
title: "Notification - 判斷WebView"
description: "JS判斷是否為WebView來取得Token"
date: 2021-09-06T16:36:44+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Notification
tags: [
    "Notification",
]
---

## 前言

透過用戶代理`useragent`偵測瀏覽器  
來判斷是否當下為`WebView`  

## 主要內容

```javascript
 var rules = ['WebView', '(iPhone|iPod|iPad)(?!.*Safari\/)', 'Android.*(wv|\.0\.0\.0)'];
 var regex = new RegExp(`(` + rules.join('|') + `)`, 'ig');
 var matchResult = navigator.userAgent.match(regex);
 var isWebview = matchResult != null && matchResult.length > 0;
 if (isWebview) {
    if (navigator.userAgent.match(/iPhone|iPad|iPod/i) && typeof window.webkit != 'undefined') {
        try {//webview需對應設定
            window.webkit.messageHandlers.Login.postMessage("Login");//呼叫手機端Login方法
        } catch (err) {           
        }
        return;
    } else if (typeof androidJs != 'undefined') { //webview需對應設定
        androidJs.login();//呼叫手機端login方法
        return;
    }
 }
```

## 參考連結

>* [url1](https://developer.mozilla.org/zh-TW/docs/Web/HTTP/Browser_detection_using_the_user_agent)