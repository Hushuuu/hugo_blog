---
title: "在Net Core 使用 ViewComponent"
description: "實現部分檢視PartialView的功能"
date: 2021-04-21T17:50:19+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "ViewComponent"
]
---

## 前言

`NetCore`也可以使用原本`HtmlHelper`的`PartialView`的方式或用`TagHelper`來呼叫  
本篇介紹如何使用更豐富的`ViewComponent`

## 主要內容

1.在專案底下新增資料夾`ViewComponents`  
2.新增一個`XXXXViewComponent.cs`檔案  
3.檔案內容須符合相關規則(可只擇一規則)才會被辨認為`ViewComponent`  

```C#
//規則:檔名為XXXXViewComponent
[Microsoft.AspNetCore.Mvc.ViewComponent] //規則: 類別掛上[ViewComponent] Attribute
public class PkindComponent : Microsoft.AspNetCore.Mvc.ViewComponent //規則: 類別繼承ViewComponent
{
    public PkindComponent()
    {
    }
    public IViewComponentResult Invoke()
    {
        return View();
    }
}
```
`View`模板預設路徑要放在
`Views/Shared/Components/XXXX/Default.cshtml`  
就可以開始設計`View`

### 如何使用

在需要呼叫`ViewComponent` 的檢視下加上

```C#
@addTagHelper *, 專案名稱
//再打上<vc 就會自動跑出選項代表成功了
<vc:XXXX></vc:XXXX>
```

### 應用

也有非同步調用方法`InvokeAsync` 和含有參數往資料庫撈資料的變化
```C#
    [Microsoft.AspNetCore.Mvc.ViewComponent]
    public class HotSaleComponent : Microsoft.AspNetCore.Mvc.ViewComponent
    {
        private Test2021Context _context;
        public HotSaleComponent(Test2021Context context)
        {
            _context = context;
        }
        public async Task<IViewComponentResult> InvokeAsync()
        {
            var list = await _context.PRODUCT //向資料庫拿資料並將list或模型傳回檢視
                .ToListAsync();
            return View(list));
        }
    }
```

## 參考連結

>* [url1](https://ithelp.ithome.com.tw/articles/10205881)
>* [url2](https://docs.microsoft.com/zh-tw/aspnet/core/mvc/views/partial?view=aspnetcore-5.0)
>* [url3](https://blog.darkthread.net/blog/aspnetcore-view-component/)