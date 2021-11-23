---
title: "產生QR Code"
description: 
date: 2021-10-21T14:57:01+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - C#
tags: [
    "C#",
    "AspNet",
]
---

## 主要內容

```C#
using System.Drawing;
using System.Drawing.Imaging;
using ZXing;                  // for BarcodeWriter
using ZXing.QrCode;           // for QRCode Engine
using System.IO;
//////圖片用ViewBag傳回base64格式
var writer = new BarcodeWriter  //dll裡面可以看到屬性
{
    Format = BarcodeFormat.QR_CODE,
    Options = new QrCodeEncodingOptions //設定大小
    {
        Height = 200,
        Width = 200,
    }
};
//產生QRcode
var img = writer.Write("www.google.com");
Bitmap myBitmap = new Bitmap(img);
MemoryStream ms = new MemoryStream();
myBitmap.Save(ms, ImageFormat.Jpeg);
byte[] arr = new byte[ms.Length];
ms.Position = 0;
ms.Read(arr, 0, (int)ms.Length);
ms.Close();
string base64str = Convert.ToBase64String(arr);

ViewBag.IMG = base64str;
```
