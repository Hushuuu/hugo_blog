---
title: "MVC-包含明細的儲存"
description: "一對多一次Post至後端的儲存"
date: 2021-03-16T15:50:53+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "html",
]
---

## 前言

在一對多的資料表下，一個表頭會對應好幾筆的明細  
開發儲存明細時，該如何實現。  

## 主要內容

若要新增資料要包含新增多筆明細。  
可以在新增頁上建立一個`Table`  
並進行新增一筆及刪除一筆的行為  
最後儲存時在將`table`進行post回去的處理
  
一種方法是在畫面上讓使用者挑選輸入明細  
輸入完畢塞一筆進`html`的`table`
```C#
<p><h4>訂單明細</h4></p>
<div class="choseProd">
    <div class="col-md-12">
        <div class="col-md-2">
            <h5>選擇商品至明細</h5>
        </div>
        <div class="col-md-2">
            @Html.DropDownList("ChoseProd", Model.AllProducts, new { @class = "form-control" })
        </div>
        <div class="col-md-1">
            @Html.EditorFor(m => m.QtyForSel, new { htmlAttributes = new { @class = "form-control", @Min = "0" } })
        </div>
        <div class="col-md-4">
            <button class="btn btn-default" id="addBtn" type="button">新增</button>
        </div>
    </div>
</div>
```
新增按鈕呼叫`applist()`AJAX POST查詢商品價格後回傳  
串接HTML(顯示欄位+隱藏的Input欄位)  
`Append`到明細`Table`  
```C#
function applist() {
    var chosepod = $("#ChoseProd").val();
    var qtyforsel = $("#QtyForSel").val();
    var price = 0;
    $.ajax({
        type: "POST",
        url: "@Url.Action("GetProdPrice", "Order")",
        data: {
            prod_id: chosepod
        },
        success: function (data) {
            var jo = JSON.parse(data);
            price = jo.price;
            var chosepod = $("#ChoseProd").val();
            var chosepod_text = $("#ChoseProd option:selected").text();
            var qtyforsel = $("#QtyForSel").val();

            var tb_append = '<tr><td name="pro_name">' + chosepod_text + '<input name="pro_name" value="' + chosepod_text + '" hidden/>'
                + '</td><td name="price">' + price + '<input name="pirce" value="' + price + '" hidden/></td><td name="qty">'
                + qtyforsel + '<input name="qty" value="' + qtyforsel + '" hidden/></td><td name="pro_id" hidden>' + chosepod
                + '<input name="pro_id" value="' + chosepod + '" hidden/></td>'
                + '<td><a class="btn btn-danger btn-sm deltr" onclick="">X</a></td></tr>'
            $('#tb1').append(tb_append);
        }
    });
}
```
後端Action
```C#
public ActionResult GetProdPrice(string prod_id)
{
    var prod = Db.Product.Where(x=>x.pro_id==prod_id).Select(x=>new { pro_id=x.pro_id,pro_name=x.pro_name,price=x.price}).Single();
    string result = JsonConvert.SerializeObject(prod);
    return Json(result);
}
```

### 儲存行為處理

後端接收明細的參數資料型態應為`List<>`  
故在POST之前需每一筆的`input`欄位要做`name`的調整  
`Array`的話需要改成`arr[0]` `arr[1]`後端才辨識的出來  
```javascript
 $(this).find("input").eq(3).attr("name", "orderdtllist[" + j + "].pro_id");
```

## 小結

一對多的儲存上較為複雜一些  
也可以在明細增加編輯功能來修改單筆就好  
要注意的是明細與表頭的關聯  
還有後端Binding參數的正確於否  