---
title: "JsonResult JsonConvert與日期Datetime"
description:
date: 2021-10-13T15:56:04+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "C#",
    "Json",
]
---

## 前言

`MVC`回傳的`ActionResult`  
可以用繼承他的`JsonResult`來回傳  
使用`return Json(obj)` 會將物件序列化後回傳至前端。  
但型別為`Datetime`會拿到`Date(12347838383333)`格式之資料  
處理方式有三種  
* JS處理轉回日期格式  
* 自訂回傳的JsonResult  
* 用JsonConvert並套上轉換屬性  

## Javascript
use FormatDate("Date(12347838383333)",true) ture則顯示時間  
```javascript
function FormatDate (str, withTime) {//格式化後端傳回的date資料
    var d = eval(`new ` + str.substr(1, str.length - 2));
    var ar_date = [d.getFullYear(), d.getMonth() + 1, d.getDate()];
    var ar_time = [d.getHours(), d.getMinutes(), d.getSeconds()];
    for (var i = 0; i < ar_date.length; i++) ar_date[i] = dFormat(ar_date[i]);
    for (var i = 0; i < ar_time.length; i++) ar_time[i] = dFormat(ar_time[i]);
    return withTime == true ? ar_date.join(`-`) + " " + ar_time.join(`: `) : ar_date.join(`-`);

    function dFormat(i) { return i < 10 ? "0" + i.toString() : i; }
}
```
## 自訂JsonResult
```C#
public class CustomJsonResult : JsonResult
{
    public CustomJsonResult(bool hasTime=false)
    {
        _hasTime = hasTime;
        if (_hasTime)
        {
            _dateFormat = "yyyy-MM-dd HH:mm:ss";
        }
        else
        {
            _dateFormat = "yyyy-MM-dd";
        }
    }
    private bool _hasTime;
    private string _dateFormat;

    public override void ExecuteResult(ControllerContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException("context");
        }

        HttpResponseBase response = context.HttpContext.Response;

        if (!String.IsNullOrEmpty(ContentType))
        {
            response.ContentType = ContentType;
        }
        else
        {
            response.ContentType = "application/json";
        }
        if (ContentEncoding != null)
        {
            response.ContentEncoding = ContentEncoding;
        }
        if (Data != null)
        {
            // Using Json.NET serializer
            var isoConvert = new IsoDateTimeConverter();
            isoConvert.DateTimeFormat = _dateFormat;
            response.Write(JsonConvert.SerializeObject(Data, isoConvert));
        }
    }
}
//使用時傳入true則表示顯示時間
return new CustomJsonResult(false)) { Data = list };
```
## 定義模型中套上JsonConvert轉換
若用`JsonConvert`來序列化日期原本是會得到`2009-02-15T00:00:00Z`之格式  
可自定義轉換格式。
```C#
public class DateConverter : IsoDateTimeConverter
{
    public DateConverter()
    {
        base.DateTimeFormat = "yyyy-MM-dd";
    }
}
public class XXX
{
    public string Id { get; set; }
    [Newtonsoft.Json.JsonConverter(typeof(DateConverter))]
    public DateTime doc_date { get; set; }
}

```
將物件序列化時，套上`DateConvert`的屬性會根據設定的格式轉換。  
回傳`Content`並設定`Content-type`為`json`  

```C#
string jsonstr = JsonConvert.SerializeObject(list);
return Content(jsonstr, "application/json");
```

## 參考連結

>* [url1](https://stackoverflow.com/questions/726334/asp-net-mvc-jsonresult-date-format)
>* [url2](https://stackoverflow.com/questions/23348262/using-json-net-to-return-actionresult)