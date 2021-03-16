---
title: "MVC-Table排序與分頁"
description: "上下換頁功能和標頭排序"
date: 2021-03-16T14:43:46+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "pager",
    "html",
]
---

## 前言

前端有很多實用的table套件如DataTable.js可以直接實現排序換頁搜尋  
本篇主要練習如何手動刻出類似的功能。

## 分頁

分頁的概念為設定一頁需顯示幾筆，再將資料分為幾等分來決定  
要顯示第幾頁的資料，`根據資料庫的種類用 Rownum Limit等`  
還可以用`LinQ Skip() Take()` 

```C#
 var list = query.Skip(startIndex).Take(pageSize).toList();
 //舉例一頁顯示10筆，要顯示第3頁的資料
 //Skip前兩頁的資料 => 10*2 
 //Skip 20筆 Take 10筆
```
#### 分頁導覽列

傳回前端的資料除了這十筆，也需要將分頁的資訊傳入進行判斷  
`page當前頁數` `EndPage最後頁` `TotalItemCount總筆數`等  
建立導覽列
```C#
if (page == 1)
{
    <a class="btn-default btn-sm disabled" id="last_btn">上頁</a>
}
else
{
    int nextnum = page - 1;
    <a class="btn-default btn-sm" id="last_btn" href="@url?page=@nextnum">上頁</a>
}
if (page == Endpage)
{
    <a class="btn-default btn-sm disabled" id="next_btn">下頁</a>
}
else
{
    int nextnum = page + 1;
    <a class="btn-default btn-sm" id="next_btn" href="@url?page=@nextnum">下頁</a>
}
```
概念為按下換頁將頁數傳至後端再進行資料過濾  
接著再加上排序的功能  
  
點擊標頭來排序並標記上顏色  
將排序的欄位和順序及頁數資訊
AJAX post進後端查詢  
傳回html TABLE直接取代
```c#
var ordering_field = Session["ordering_field"] ?? "";
var ordering_kind = Session["ordering_kind"] ?? "";
$(document).on('click', '#tb1 thead tr th', function () { 
    //去除顏色     
     $(this).closest('tr').find('th').each(function (index) {
            $(this).removeAttr('style');
        });
    let order_field = $(this).attr('name');
    let order_kind = "Desc";
    if (ordering_field == order_field && ordering_kind == "Desc") {
        order_kind = "Asc";
        $(this).css('background-color', 'yellow');
    }
    else {
        $(this).css('background-color', 'pink');
    }
    let Lvm = {
        page:@Model.page,
        EndPage:@Model.EndPage,
        pageSize:@Model.pageSize,
        TotalItemCount:@Model.TotalItemCount,
        OrderField: order_field,
        OrderKind: order_kind
    };
    let obj = {
        Lvm: Lvm
    }
    $.ajax({
        type: "POST",
        url: "@Url.Action("OrderByField","Order")",
        data: obj,
        success: function (data) {
            $('#tb1 tbody').html(data);
            ordering_field = order_field;
            ordering_kind = order_kind;
        }
    });
});
```
AJAX回傳`PartialView`結果，`Partial`產出`Table`直接取代原先Html  
Session可以記住排序的欄位順序甚至查詢條件  
再切換頁面或其他動作時可恢復原本過濾後的結果  

```c#
return PartialView("_SortOrderView", Lvm);
```
## 小結

若沒有特別的要求，其實使用前端套件來的更方便快速  
但在刻畫功能過程中確實也能收穫不少！  
*在使用LinQ排序時可引用一個套件方便做欄位的DESC及ASC
`Using Linq.Dynamic;`
