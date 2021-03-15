---
title: "MVC-產出Excel"
description: "介紹幾種產出Excel的方式(NPOI,EPPlus)"
date: 2021-03-15T17:15:18+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "Excel"
]
---

## 前言

介紹NPOI及EPPlus套件來將資料產出成Excel。

## 主要內容

NPOI的方式產出報表，XSSFWorkbook為.xlsx格式HSSF則為較舊的.xls  

```C#
public ActionResult getExcel()
{
    var result = List<Data>;//資料
    //建立Excel
    XSSFWorkbook xssfworkbook = new XSSFWorkbook(); //建立活頁簿
    ISheet sheet = xssfworkbook.CreateSheet("sheet"); //建立sheet

    //設定樣式
    ICellStyle headerStyle = xssfworkbook.CreateCellStyle();
    IFont headerfont = xssfworkbook.CreateFont();
    headerStyle.Alignment = HorizontalAlignment.Center; //水平置中
    headerStyle.VerticalAlignment = VerticalAlignment.Center; //垂直置中
    headerfont.FontName = "微軟正黑體";
    headerfont.FontHeightInPoints = 20;
    headerStyle.SetFont(headerfont);

    //新增標題列
    sheet.CreateRow(0); //需先用CreateRow建立,才可通过GetRow取得該欄位
    sheet.GetRow(0).CreateCell(0).SetCellValue("商品清單");
    sheet.AddMergedRegion(new CellRangeAddress(0, 0, 0, 2)); //合併A~C欄儲存格
    sheet.GetRow(0).GetCell(0).CellStyle = headerStyle; //套用樣式
    sheet.CreateRow(1).CreateCell(0).SetCellValue("商品編號");
    sheet.GetRow(1).CreateCell(1).SetCellValue("商品名");
    sheet.GetRow(1).CreateCell(2).SetCellValue("單價");
    sheet.GetRow(1).CreateCell(3).SetCellValue("庫存");
    //填入資料
    int rowIndex = 2;
    for (int row = 0; row < result.Count(); row++)
    {
        sheet.CreateRow(rowIndex).CreateCell(0).SetCellValue(result[row].pro_id);
        sheet.GetRow(rowIndex).CreateCell(1).SetCellValue(result[row].pro_name);
        sheet.GetRow(rowIndex).CreateCell(2).SetCellValue(Convert.ToDouble(result[row].price));
        sheet.GetRow(rowIndex).CreateCell(3).SetCellValue(Convert.ToDouble(result[row].qty));
        rowIndex++;
    }

    System.IO.MemoryStream ms = new System.IO.MemoryStream();
    xssfworkbook.Write(ms);
    string handle = Guid.NewGuid().ToString();
    TempData[handle] = ms.ToArray();
    return Json(data: new { FileGuid = handle, FileName = "商品清單.xlsx" }) ;
}
```
因為是透過AJAX方式進後端，回傳這部分將檔案的`MemoryStream Array`利用`TempData`傳回  
並取得一個Guid來當這資料的KEY，回傳KEY和檔案名稱回View。  
Ajax回傳至前端後，利用iframe的方式實現下載(打下載的Action)  
### Ajax完成回傳的ata
```javascript
    var response = data;
    //iframe
    $("#ifrm").remove();
    var ifrm = document.createElement("iframe");
    ifrm.setAttribute("src", '/Product/Download?fileGuid=' + response.FileGuid
        + '&filename=' + response.FileName);
    ifrm.style.display = "none";
    ifrm.id = "ifrm";
    let dv = $("<div></div>").html(ifrm);
    $('body').append(dv);
```
### 下載部分
```c#
    [HttpGet]
    public virtual ActionResult Download(string fileGuid, string fileName)
    {
        if (TempData[fileGuid] != null)
        {
            byte[] data = TempData[fileGuid] as byte[];
            return File(data, "application/vnd.ms-excel", fileName);
        }
        else
        {
            return new EmptyResult();
        }
    }
```
不過有另一種更簡明易懂的方式  
`Action`可以直接回傳`File Result`就可以實現下載  
使用`Form Post`的方法並開新頁`target="_blank"` 直接回傳檔案型別
```C#
    public ActionResult getExcel2(IWorkbook workbook)
    {
        System.IO.MemoryStream ms = new System.IO.MemoryStream();
        workbook.Write(ms);
        return File(ms.ToArray(), "application/vnd.ms-excel", string.Format($"商品清單.xlsx"));
    }       
```
## 小結

產生Excel之後，也可以選擇先產生存在Server端，Client端再去取得Server端檔案路徑進行下載。  
本文的方式都不會在Server端產生檔案。可以節省Server端空間也不用有清除歷史檔案的需求。  
不過也有需在Server端產檔的可能，再另外做調整。


## 參考連結

>* [url1](https://t.codebug.vip/questions-986624.htm)
>* [url2](https://www.tpisoftware.com/tpu/articleDetails/1654)