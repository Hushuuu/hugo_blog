---
title: "Ajax非同步接收blob結果下載檔案"
description:
date: 2023-08-01T15:22:07+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Ajax
tags: [
    "Ajax",
    "JavaScript",
]
---

## 前言

一般下載檔案常見使用開新頁面請求回傳檔案的方式  
非同步的請求也可以正確接收檔案觸發瀏覽器下載  
透過接收Blob型別的response來解析  

## 主要內容

`axios`
```javascript
axios.get('url',{responseType: 'blob' })
.then(res => {
    const { headers, data } = res,
        blob = new Blob([data])
        ;
    let fname = 'samp.txt'
    const url = window.URL.createObjectURL(blob); //可以接收一個File 或 Blob objet，並回傳一段URL。瀏覽器建立關聯到本地端檔案的連結
    const a = document.createElement('a');
    a.style.display = 'none';
    a.href = url;
    a.download = fname;
    document.body.appendChild(a);
    a.click();
    window.URL.revokeObjectURL(url); //釋放
});
```
`fetch`
```javascript
fetch('url')
    .then(resp => resp.blob())
    .then(blob => {
        const url = window.URL.createObjectURL(blob); //可以接收一個File 或 Blob objet，並回傳一段URL。瀏覽器建立關聯到本地端檔案的連結
        const a = document.createElement('a');
        a.style.display = 'none';
        a.href = url;
        a.download = 'samp.txt';
        document.body.appendChild(a);
        a.click();
        window.URL.revokeObjectURL(url); //釋放
    })
    .catch(() => alert('oh no!'));
```
若要取得回傳的檔案名稱
```javascript
//取得回傳的結果header並解析content-disposition區塊
let filename = getFileName(headers["content-disposition"]);

function getFileName(disposition) {
    console.log(disposition)
    const utf8FilenameRegex = /filename\*=UTF-8''([\w%.-]+)(?:; ?|$)/i;
    const asciiFilenameRegex = /^filename=(["']?)(.*?[^\\])\1(?:; ?|$)/i;

    let fileName = null;
    if (utf8FilenameRegex.test(disposition)) {
        fileName = decodeURIComponent(utf8FilenameRegex.exec(disposition)[1]);
    } else {
        // prevent ReDos attacks by anchoring the ascii regex to string start and
        //  slicing off everything before 'filename='
        const filenameStart = disposition.toLowerCase().indexOf('filename=');
        if (filenameStart >= 0) {
            const partialDisposition = disposition.slice(filenameStart);
            const matches = asciiFilenameRegex.exec(partialDisposition);
            if (matches != null && matches[2]) {
                fileName = matches[2];
            }
        }
    }
    return fileName;
}
```