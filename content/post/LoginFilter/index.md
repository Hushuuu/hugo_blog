---
title: "MVC 登入驗證"
description: "利用ActionFilter來判斷是否有登入，否則拒絕訪問導回登入頁面"
date: 2021-03-15T09:26:53+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "Login",
    "ActionFilter",
]
---

## 前言

關於過濾器，可以提前了解一下Asp.net MVC的生命週期。  
可透過放在不同週期階段的過濾器來達成不同的需求。  
有關生命週期相關可參考此[連結](https://nwpie.blogspot.com/2017/05/5-aspnet-mvc.html)  
本文主要介紹`Action Filter`

## 主要內容

我們可在專案底下新增一個`ActionFilters`的資料夾  
在裡面新增一個`LoginFilter.cs`檔  
裡面就可以寫自定義的Filter  
繼承`ActionFilterAttribute`並複寫`OnActionExecuting`方法  
這邊條件我拿一個登入後設定的Seesion來做判斷。  
不通過則`filterContext.Result`設定導向回登入頁
```C#
public class LoginFilter: ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        if (HttpContext.Current.Session["Login_id"] == null)
        {
            filterContext.Result = new RedirectToRouteResult(new RouteValueDictionary(new
            {
                controller = "Home",
                action = "Login"
            }));
        }           
    }
}
```

## 掛上Filter

建立好自定義的Filter後，使用的方式可根據想套用的範圍大小來掛上Filter。  
可以在Action上頭掛上`[LoginFilter]`，或者是掛在Controller上來套用整個控制器的Action

