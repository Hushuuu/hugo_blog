---
title: "MVC-上傳檔案轉為Base64String"
description: "上傳檔案後轉換為Base64或在前端上傳前就轉換為Base64String"
date: 2021-03-25T11:15:44+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "Javascript",
    "file",
]
---

## 前言

`Html`的`Img`標籤可以將`src`設為轉換後的`Base64 String`也可以顯示出圖片  
若`WebApi`需要交流檔案也可以將檔案轉換為`Base64`進行溝通  

## Post前轉換

建立一個按鈕事件將轉換後的`base64String` `Post`至後端
```javascript
$(document).on('click', '#subBtn', function () {
        demo().then((obj) => {
            $.ajax({
                url: "../",            // url位置
                type: "POST",       
                contentType: "application/json",
                data: JSON.stringify(obj),// 輸入的資料
                success: function (response) { }// 
            });
        });
    });
```

使用`FileReader`來讀取上傳的檔案  
因為`readAsDataURL`是非同步讀取  
若不做非同步等待處理  
檔案還未處理完就會被Post出去造成傳遞不到正確資料  
故需將函式套上`async`並`await`等候所有檔案都處理完  

```javascript
const demo = async () => {
    var obj = {};
    var filearray = [];
    var file_name = [];
    let files = $("input[name=upload_file]")[0].files;
    for (let i = 0; i < files.length; i++) {
        var reader = new FileReader();
        reader.readAsDataURL(files[i]); //data url
        const result = await new Promise((resolve, reject) => {
            reader.onload = function (e) {
                var buffer = e.target.result;  //是data url
                const base64String = buffer //只取得base64 string
                    .replace("data:", "")
                    .replace(/^.+,/, "");
                resolve(base64String);
            }
        });
        filearray.push(result);
        file_name.push(files[i].name);
    }
    obj = {
        file_byte: filearray,
        file_name: file_name
    };
    return obj;
};
```
## Post後轉換

上傳至後端才做轉換的處理相較之下就簡單許多  

```C#
 for (int i = 0; i < Request.Files.Count; i++)
{
    HttpPostedFileBase file = Request.Files[i];
    MemoryStream ms = new MemoryStream();
    file.InputStream.CopyTo(ms);
    byte[] btarr = ms.ToArray();
    string basestr = Convert.ToBase64String(btarr); 
}
```

## 小結

`async` `await` `Promise`這些非同步相關的使用上需要花點時間理解  
主要目的為實現等待：檔案非同步讀取完全做完才上傳出去  


## 參考連結

>* [url1](https://www.oxxostudio.tw/articles/201908/js-async-await.html)
>* [url2](https://developer.mozilla.org/en-US/docs/Web/API/FileReader/readAsDataURL)