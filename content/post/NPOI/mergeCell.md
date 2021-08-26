---
title: "NPOI 合併儲存格與框線樣式"
description: "合併儲存格之後框線樣式會跑版"
date: 2021-08-26T10:41:03+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NPOI
tags: [
    "NPOI",
    
]
---

## 前言

NPOI 是可讀取及產出Excel的套件。同JAVA的POI  

## EXCEL概觀

若要建立Excel首先要先建立`Workbook`  
`Workbook`裡可建立數個`Sheet`  
`Sheet`就為一張一張的表  
每個表內從第0行(Row)第0欄(Column)開始第一個單元格  

產出的格式若是較新版的Excel(.xlsx) 使用`XSSFWorkbook`  
若為舊版Excel格式(.xls)則使用`HSSFWorkbook`  
這邊以XSSF作範例，將A0合併至B2  

```C#
XSSFWorkbook xssfworkbook = new XSSFWorkbook(); //建立Workbook
ISheet sheet = xssfworkbook.CreateSheet(sheetName); //建立sheet
sheet.CreateRow(0).CreateCell(0).SetCellValue("標題"); //第一次一定要先建立NewRow NewCell
CellRangeAddress region = new CellRangeAddress(0,1,0,1);//指定區域A0~B2  
sheet.AddMergedRegion(region);//合併區域
ICellStyle myStyle = xssfworkbook.CreateCellStyle();//建立樣式 
myStyle.BorderTop = BorderStyle.Thin;//設定上框線
myStyle.BorderBottom = BorderStyle.Thin;//設定下框線
myStyle.BorderLeft = BorderStyle.Thin;//設定框線
myStyle.BorderRight = BorderStyle.Thin;//設定框線
sheet.GetRow(0).GetCell(0).CellStyle = myStyle; //設定樣式

//產出檔案並回傳
System.IO.MemoryStream ms = new System.IO.MemoryStream();
xssfworkbook.Write(ms);
return File(ms.ToArray(), "application/vnd.ms-excel", "myExcel.xlsx");

```

### 合併後框線失效

上面的例子產出報表後，並未符合預期的樣子  
合併後的框線樣式跑掉了。  
這是因為框線樣式只有設定到A0這個元素格  
合併後右下的框線就被合併起來了  
故最簡單解決方式為把其他元素格也設定框線樣式  

### 解決方案

新增Function傳入合併的區域Region，將此Region的格子全部設定樣式  
```C#
public void SetRegionBorder(IWorkbook workbook, ISheet sheet, CellRangeAddress region, ICellStyle style)
{
    for (int i = region.FirstRow; i <= region.LastRow; i++)
    {
        IRow row = sheet.GetRow(i) ?? sheet.CreateRow(i);
        for (int j = region.FirstColumn; j <= region.LastColumn; j++)
        {
            ICell cell = row.GetCell(j) ?? row.CreateCell(j);
            cell.CellStyle = style;
        }
    }
}

//call method
CellRangeAddress region = new CellRangeAddress(0,1,0,1);//指定區域A0~B2  
SetRegionBorder(xssfworkbook, sheet, region, myStyle);

```

## 小結

使用`ISheet IRow ICell ICellStyle` 等等介面可通用於`XSSF`和`HSSF`的類別  
兼容上較方便，但較豐富的設定則無法使用介面來設定，需另外處理  

較詳細的用法擇日另篇再筆記。。
