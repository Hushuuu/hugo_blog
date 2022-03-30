---
title: "Text.Json 客製JsonConverter"
description: "後端回傳Json時處理for Datetime格式"
date: 2022-02-17T08:40:41+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Json
tags: [
    "Json",
    "NetCore",
]
---

## 主要內容

建立類別繼承JsonConverter來掛在要轉換的屬性上  
```C#
 public class OnlyDateConverter : JsonConverter<DateTime>
    {
        public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        {
            //Debug.Assert(typeToConvert == typeof(DateTime));
            return DateTime.Parse(reader.GetString());
        }

        public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
        {
            writer.WriteStringValue(value.ToString("yyyy-MM-dd"));
        }
    }
    public class DateWithTimeConverter : JsonConverter<DateTime>
    {
        public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
        {
            return DateTime.Parse(reader.GetString());
        }

        public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
        {
            writer.WriteStringValue(value.ToString("yyyy-MM-dd HH:mm:ss"));
        }
    }
```
`掛載至屬性`
```C#
////使用時只要掛在想要轉換的屬性上
/// <summary>
/// 日期
/// </summary>
[JsonConverter(typeof(OnlyDateConverter))]
public DateTime? date { get; set; }

[JsonConverter(typeof(DateWithTimeConverter))]
public DateTime? date2 { get; set; }
```

`轉換時設定`
```C#
JsonSerializerOptions options = new JsonSerializerOptions();
options.Converters.Add(new OnlyDateConverter());
return Json(obj2, options);
```
`startup設定 注意這設定只會影響到Controller.Json retrun`
`除了日期轉換，順便處理會被轉小寫+中文字被編碼的問題` 
```C#
//JSON預設修改
services.AddMvc()
.AddJsonOptions(options =>
{
    //原本是 JsonNamingPolicy.CamelCase，強制頭文字轉小寫，我偏好維持原樣，設為null
    options.JsonSerializerOptions.PropertyNamingPolicy = null;
    //允許基本拉丁英文及中日韓文字維持原字元
    options.JsonSerializerOptions.Encoder =
        JavaScriptEncoder.Create(UnicodeRanges.BasicLatin, UnicodeRanges.CjkUnifiedIdeographs);
    //datetime 預設處理只有日期
    options.JsonSerializerOptions.Converters.Add(new OnlyDateConverter());
});
```

`掛載轉換器的優先順序`  
1.[JsonConverter] 套用至屬性。  
2.加入至集合的轉換器 Converters 。  
3.[JsonConverter] 套用至自訂實數值型別或 POCO。  
4.如果在集合中註冊了某個類型的多個自訂轉換器 Converters ，則會使用第一個傳回 true 的轉換器 CanConvert 。  

## 小結

`net core`預設使用的`Text.Json`據說比較輕量但使用上起來得設定一些設定才能用的順手  
若是手動呼叫`JsonSerializer`不會吃到startup的設定，必須自行給`option`或擴充共用


## 參考連結

>* [url1](https://docs.microsoft.com/zh-tw/dotnet/standard/serialization/system-text-json-converters-how-to?pivots=dotnet-5-0)
>* [url2](https://blog.darkthread.net/blog/aspnet-core-json-setting/)
>* [url3](https://medium.com/@mvpdw06/net-core-3-1-%E5%BE%8C%E8%BD%89%E7%A7%BB-newtonsoft-json-%E8%87%B3-system-text-json-9727d774f92d)