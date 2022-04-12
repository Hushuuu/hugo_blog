---
title: "WinForm列印PrintDocument"
description:
date: 2022-04-12T08:42:19+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Winform
tags: [
    "Winform",
    "C#",
]
---

## 主要內容

可將欲列印之內容，使用`PrintDocument`類別來列印出來  
需用繪圖的方式來繪製出結果  

### 建立Panel元件和PrintDocument

設定好Panel的畫面及內容後，也建立`Bitmap`物件並將`Panel`繪製為`Bitmap`  

```C#
  var tempPanel = panel1;
  var _NewBitmap = new Bitmap(tempPanel.Width, tempPanel.Height);
   tempPanel.DrawToBitmap(_NewBitmap, new Rectangle(0, 0, _NewBitmap.Width, _NewBitmap.Height));

    //可設定使用的印表機名稱
    //printDocument1.PrinterSettings.PrinterName = "";
    
    StandardPrintController spc = new StandardPrintController();//初始化列印控制器不跳列印中
     printDocument1.PrintController = spc;
    this.printDocument1.Print();//

```

`PrintDocument的PrintPage事件中繪製結果`

```C#
private void printDocument1_PrintPage(object sender, PrintPageEventArgs e)
    {
        e.Graphics.CompositingQuality = System.Drawing.Drawing2D.CompositingQuality.HighSpeed;//品質
        e.Graphics.DrawImage(_NewBitmap
            , 0, 0, _NewBitmap.Width
            , _NewBitmap.Height
            );
    }   

```

## 參考連結

>* [url1](https://www.itread01.com/content/1550175511.html)