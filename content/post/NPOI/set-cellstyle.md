---
title: "NPOI - CellStyle篇"
description:
date: 2021-08-26T11:52:40+08:00
image:
draft: false
license:
hidden: false
comments: true
categories:
  - NPOI
tags: ["NPOI"]
---

## 前言

NPOI 設定框線、字型、顏色、對齊、換行...等常用樣式設定

### 建立表

```C#
XSSFWorkbook xssfworkbook = new XSSFWorkbook(); //建立Workbook
ISheet sheet = xssfworkbook.CreateSheet(sheetName); //建立sheet
sheet.CreateRow(0).CreateCell(0).SetCellValue("標題"); //第一次一定要先建立NewRow NewCell
```

### 框線

`BorderStyle`框線樣式  
`BorderColor`框線顏色

```C#
ICellStyle myStyle = xssfworkbook.CreateCellStyle();//建立樣式
myStyle.BorderTop = BorderStyle.Thin;//設定上框線
myStyle.BorderBottom = BorderStyle.Thin;//設定下框線
myStyle.BorderLeft = BorderStyle.Thin;//設定框線
myStyle.BorderRight = BorderStyle.Thin;//設定框線
```

### 字型

`FontName`字體名稱:新細明體  
`FontHeightInPoints`字體大小(double):12  
`IsBold`是否粗體(bool)  
`Color`顏色(short) HSSFColor.Black.Index

```C#
IFont cFont = workboook.CreateFont();
cFont.FontName = "微軟正黑體";
cFont.FontHeightInPoints = 12;
cFont.IsBold = true;
cFont.Color = HSSFColor.Red.Index;
myStyle.SetFont(cFont);
```

### 對齊

```C#
myStyle.Alignment = HorizontalAlignment.Center; //水平置中
myStyle.VerticalAlignment = VerticalAlignment.Center; //垂直置中
```

### 換行

```C#
myStyle.WrapText = true;
```

### 顏色

`ICellStyle`中設定顏色通常都是傳入`short`型別，如傳入`HSSFColor.Yellow.Index`  
若想用`RGB`或`ARGB`來自定特定顏色則只在`XSSFCellStyle`可以做設定

`特別注意:單元格的背景顏色似乎是設定FillForegroundColor而不是設定FillBackgroundColor`  
`而且必須同時設定FillPattern才會有效果`

#### 一般顏色設定

```C#
ICellStyle myStyle = workbook.CreateCellStyle();//建立樣式
myStyle.FillForegroundColor = HSSFColor.Pink.Index;
myStyle.FillPattern = FillPattern.SolidForeground;
```

#### 特殊顏色設定

```C#
XSSFCellStyle cStyle = (XSSFCellStyle)workboook.CreateCellStyle(); //樣式
byte[] rgbByte = new byte[]{192,0,0};//深紅色RGB
XSSFColor rgbColor = new XSSFColor(rgbByte);
cStyle.FillForegroundColor = rgbColor;
myStyle.FillPattern = FillPattern.SolidForeground;
```

### 包成 Function 來設定

單元格的樣式若稍有差異就要寫好幾行程式，包成方法來傳入參數自動設定。
仍可增加更多參數來設定樣式

```C#

public static class ExcelStyleHelper
{
    public static void SetRegionStyle(this ISheet sheet, CellRangeAddress region, ICellStyle style)
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
    /// <summary>
    /// XSSFCellStyle較多參數可設定
    /// </summary>
    /// <param name="workboook">XSSFWorkbook</param>
    /// <param name="hAlignment">水平對齊</param>
    /// <param name="vAlignment">垂直對齊</param>
    /// <param name="fontSize">字體大小</param>
    /// <param name="fontName">字體名稱</param>
    /// <param name="Btop">上框線</param>
    /// <param name="Bright">右框線</param>
    /// <param name="Bdown">下框線</param>
    /// <param name="Bleft">左框線</param>
    /// <param name="CanWrapText">可換行</param>
    /// <param name="isLocked">鎖定</param>
    /// <param name="isFontBold">粗體</param>
    /// <param name="xssffontColor">字體顏色byte[rgb]</param>
    /// <param name="xssffgColor">前景顏色byte[rgb]</param>
    /// <param name="dataFormat">格式文字是@ 數字是0 千分位#,##0</param>
    /// <returns></returns>
    public static XSSFCellStyle genXSSFCellStyle(XSSFWorkbook workboook
        , HorizontalAlignment hAlignment = HorizontalAlignment.Center, VerticalAlignment vAlignment = VerticalAlignment.Center
        , double fontSize = 14, string fontName = "微軟正黑體"
        , BorderStyle Btop = 0, BorderStyle Bright = 0, BorderStyle Bdown = 0, BorderStyle Bleft = 0
        , bool CanWrapText = false, bool isLocked = false, bool isFontBold = false
        , byte[] xssffontColor = null, byte[] xssffgColor = null, string dataFormat = "@")
    {
        XSSFCellStyle cStyle = (XSSFCellStyle)workboook.CreateCellStyle(); ////設定樣式
        cStyle.Alignment = hAlignment; //水平置中
        cStyle.VerticalAlignment = vAlignment; //垂直置中
        XSSFFont cFont = (XSSFFont)workboook.CreateFont();
        cFont.FontName = fontName;
        cFont.FontHeightInPoints = fontSize;
        cFont.IsBold = isFontBold;
        if (xssffontColor != null)
        {
            cFont.SetColor(new XSSFColor(xssffontColor));
        }
        cStyle.SetFont(cFont);
        cStyle.BorderTop = Btop;
        cStyle.BorderRight = Bright;
        cStyle.BorderBottom = Bdown;
        cStyle.BorderLeft = Bleft;
        cStyle.WrapText = CanWrapText;
        cStyle.IsLocked = isLocked;
        if (xssffgColor != null)
        {
            cStyle.FillPattern = FillPattern.SolidForeground;
            cStyle.SetFillForegroundColor(new XSSFColor(xssffgColor));
        }
        cStyle.DataFormat = workboook.CreateDataFormat().GetFormat(dataFormat); //設定格式
        return cStyle;
    }

    //set region merge
    public static CellRangeAddress SetRegionMerge(this ISheet sheet, int firstRow, int lastRow, int firstCol, int lastCol)
    {
        var region = new CellRangeAddress(firstRow, lastRow, firstCol, lastCol);
        sheet.AddMergedRegion(region);
        return region;
    }
}

```

## 小結

設定樣式也可以透過讀取設計好的 Excel 檔之`Sheet`來產出  
但處理上可能較為不彈性。適合較單調簡易的讀檔填值。

其他仍有像是數值格式`DataFormat`的設定未紀錄  
有用到再另外補上。
