---
title: "Linq常用的語法與基本介紹"
description: "與資料息息相關的Linq"
date: 2021-03-16T16:46:20+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Linq
tags: [
    "linq"
]
---

## 前言

LINQ在資料的過濾以及查詢方面非常方便，不只可以在EF中轉為SQL語句  
也能對平常的List型別進行過濾查詢。

## 主要內容

Linq有兩種表示式，有一種是使用Lambda表示式  
```C#
 var p = from t in products
                where t.Name == "牙膏"
                select t;
//Lambda
var p2 = products.Where(t => t.Name == "牙膏");
```

### JOIN
```C#
//left join linq
var q = from e in Db.Page_M 
        orderby e.page_stack
        join s in ( from s in Db.Group_D where s.group_id==group_id select s)
        on new { page_id = e.page_id } equals
        new { page_id = s.page_id } into subGrp
        from s in subGrp.DefaultIfEmpty()
        select new GroupDModel
        {
            page_id = e.page_id,
            page_name = e.page_name,
            controller = e.controller,
            action = e.action,
            can_enter = s.can_enter ?? false,
            can_see = s.can_see ?? false,
            page_stack = e.page_stack
        };
//left join lambda
var qq = Db.Page_M.OrderBy(x=>x.page_stack)
    .GroupJoin(Db.Group_D.Where(x=>x.group_id==group_id), 
    e => e.page_id, 
    s => s.page_id,
    (e, s) => new
    {
        page = e,
        group = s
    })
    .SelectMany(s => s.group.DefaultIfEmpty(), (e, s) => new GroupDModel
    {
        page_id = e.page.page_id,
        page_name = e.page.page_name,
        controller = e.page.controller,
        action = e.page.action,
        can_enter = s.can_enter ?? false,
        can_see = s.can_see ?? false,
        page_stack = e.page.page_stack
    });
```
比較複雜的語法得仔細檢查是否結果無誤
```C#
 List<BuyDtl> all_list = query.ToList();
 all_list.GroupBy(x => x.pro_id).Select(x => x.Key)
    .ToList()
    .ForEach(z =>
    {
        all_list.Where(x => x.pro_id == z)
        //groupBy日期
        .GroupBy(x => new { pro_id = x.pro_id, doc_date = x.BuyMst.doc_date })
        .Select(x => new { doc_date = x.Key.doc_date, s_qty = x.Sum(y => y.qty) })
        .ToList()
        .ForEach(c =>
        {
            ChartPointData<DateTime, decimal?> cpd = new ChartPointData<DateTime, decimal?> {
            x = c.doc_date.Value,
            y = c.s_qty
            };
        });
    });
```
```C#
 IQueryable<BuyDtl> query = Db.BuyDtl.Where(x => x.BuyMst.doc_date == today);
    //當天全部
    ChartDatasets<string, decimal?> chart_ds = new ChartDatasets<string, decimal?>();
    List<ChartPointData<string, decimal?>> lst_p = new List<ChartPointData<string, decimal?>>();
    query.GroupBy(y => new { pro_id = y.pro_id })
    .Select(z => new { pro_id = z.Key.pro_id, s_qty = z.Sum(c => c.qty) })
    .ToList()
    .ForEach(v =>
    {
        string proname = Db.Product.Where(y => y.pro_id == v.pro_id).Select(y => y.pro_name).FirstOrDefault();
    });
```
## 小結

linq is good

## 參考連結

>* [url1](https://ithelp.ithome.com.tw/articles/10194251)