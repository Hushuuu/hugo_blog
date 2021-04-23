---
title: "NetCore 更改預設ModelBinding ErrorMessage"
description: "將預設錯誤訊息英文改為中文"
date: 2021-04-23T10:04:41+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "ModelBinding"
]
---

## 前言

`NetCore`就算更改了`Local Language`若沒做多國語言資源，似乎沒有預設中文的樣板  
除了可以在Attribue裡面自定義錯誤訊息來改成中文但共用屬性每個都改太麻煩了
可以使用自訂義類別實作 `IValidationMetadataProvider`介面來實現更改預設訊息

## 主要內容

建立cs檔
```c#
public void CreateValidationMetadata(ValidationMetadataProviderContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException();
        }
        var validators = context.ValidationMetadata.ValidatorMetadata;

        // add [Required] for value-types (int/DateTime etc)
        // to set ErrorMessage before asp.net does it
        var theType = context.Key.ModelType;
        var underlyingType = Nullable.GetUnderlyingType(theType);

        if (theType.IsValueType &&
            underlyingType == null && // not nullable type
            validators.Where(m => m.GetType() == typeof(RequiredAttribute)).Count() == 0)
        {
            validators.Add(new RequiredAttribute());
        }
        foreach (var obj in validators)
        {
            if (!(obj is ValidationAttribute attribute))
            {
                continue;
            }
            fillErrorMessage<RequiredAttribute>(attribute,
                "'{0}'是必填欄位.");
            fillErrorMessage<MinLengthAttribute>(attribute,
                "'{0}' 的最小長度是 {1}.");
            fillErrorMessage<MaxLengthAttribute>(attribute,
                "'{0}' 的最大長度是 {1}.");
            fillErrorMessage<EmailAddressAttribute>(attribute,
                "非合法格式的電子信箱.", true);
            // other attributes like RangeAttribute, CompareAttribute, etc
        }
    }
```
在`Startup.cs`中的`ConfigureServices`裡加上
```C#
services.AddControllersWithViews()
            .AddMvcOptions(m => {
                m.ModelMetadataDetailsProviders.Add(new MyModelMetadataProvider());
            });
```


## 小結

若要將`ErrorMessage`顯示多種語言的話。就需要用資源檔來做`localization`  
只有需要顯示中文的話此篇方法比較簡潔。

## 參考連結

>* [url1](https://stackoverflow.com/questions/59284038/how-to-localize-standard-error-messages-of-validation-attributes-in-asp-net-core)
>* [url2](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-3.1#dataannotations-localization)