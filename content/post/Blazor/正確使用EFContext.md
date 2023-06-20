---
title: "Blazor正確使用EFContext"
description: "較為安全不會混用的方式"
date: 2023-06-20T16:21:12+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Blazor
tags: [
    "Blazor",
    "EF",
]
---

## 前言

`Blazor`在使用`EF Context`要特別注意生命週期  

一般的注入方式`services.AddDbContext`  
預設的生命週期是`Scoped`  
在每個客戶端請求連線時，建立一個新的實例  

但是在`Blazor`中即使換頁，並不會重新請求重載頁面  
代表兩個畫面之間注入的`DbContext`是同一個  
與一般開發MVC時每個`Action`都會建立新的實體不同  

如有嚴謹的`DbContext`的追蹤與變更則不太會有問題  
但避免有例外情況，例如在不同頁面的存檔卻導致改到了非預期的資料表  
當下就很難找出問題點了!!!  

## Solution1 Transient 

在注入`DbContext`時選擇生命週期`Transient` 
每次要求元件時就建立一個新的  
缺點並未實測。
```C#
//program.cs
{
services.AddDbContext<TpmcDbContext>(options => options.UseOracle()
    ,contextLifetime: ServiceLifetime.Transient);
}

```

## Solution2 DbContextFactory

注入`ContextFactory`，在`Action`中使用再建立`Context` 
```C#
//program.cs
//lifeTime預設是singleton永久共用
{
    builder.Services.AddDbContextFactory<>();
}
//XXXX.razor
 [Inject]
public IDbContextFactory<TpmcDbContext> _contextFactory { get; set; }

 async Task RefreshData()
{
    using var db = _contextFactory.CreateDbContext();
    var list = db.TableA.ToList();
}
```

## 參考連結

>* [url1](https://www.youtube.com/watch?v=aaQsmkh1BkQ)
>* [url2](https://learn.microsoft.com/en-us/aspnet/core/blazor/blazor-server-ef-core?view=aspnetcore-7.0)