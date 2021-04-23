---
title: "NetCore 啟用Session"
description: "Session取值給值及擴充方法來轉換物件"
date: 2021-04-23T10:39:28+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "Session"
]
---

## 前言

安裝套件` Microsoft.AspNetCore.Session`  

`Startup.cs > ConfigureServices`中加入
```C#
// 將 Session 存在 ASP.NET Core 記憶體中
services.AddDistributedMemoryCache();
services.AddSession(options =>
{               
    options.Cookie.HttpOnly = true; //防止XSS攻擊者存取Cookies
});

```
`Startup > Configure`中加入
```C#
// SessionMiddleware 加入 Pipeline
app.UseSession();
```

## 開始使用

```C#
//設定Session
HttpContext.Session.SetString("SessionKey", "SessionValue");
//取得Session
string sVal = HttpContext.Session.GetString("SessionKey");
```

`Session`如果需要存物件進去就需要寫擴充的方法來實現
新增`SessionExtensions.cs`
```C#
  //HttpContext.Session.GetObject<T>("SessionKey");
  //HttpContext.Session.SetObject<T>("SessionKey",object);
  public static class SessionExtensions
    {
        public static void SetObject<T>(this ISession session, string key, T value)
        {
            session.SetString(key, JsonConvert.SerializeObject(value));
        }

        public static T GetObject<T>(this ISession session, string key)
        {
            var value = session.GetString(key);
            return value == null ? default(T) : JsonConvert.DeserializeObject<T>(value);
        }
    }
```

### Razor Page 中使用

兩種方法都可以取到值
```C#
@inject Microsoft.AspNetCore.Http.IHttpContextAccessor HttpContextAccessor
@{
    string vall = HttpContextAccessor.HttpContext.Session.GetString("SessionKey");
}
```
```C#
@using Microsoft.AspNetCore.Http
string vall = Context.Session.GetString("SessionKey");
```

## 參考連結

>* [url1](https://blog.johnwu.cc/article/ironman-day11-asp-net-core-cookies-session.html)
>* [url2](https://smalltowntechblog.wordpress.com/2017/10/31/asp-net-core-mvc-%E7%9A%84-session-%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F/)