---
title: "MVC-使用AntiForgery Token防止跨網站(XSRF/CSRF)攻擊"
description: ""
date: 2021-03-29T15:56:17+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "CSRF"
]
---

## 前言

MVC本身可在`Form Post`前使用`AntiForgeryToken`  
在`Action`上掛上`[ValidateAntiForgeryToken]`來檢查驗證  
如果要在`Ajax`使用可以自行建立驗證  

## 內建的AntiForgeryToken

```C#
@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
}
```
後端掛在動作上驗證
```C#
[ValidateAntiForgeryToken]
public ActionResult Create(){ 
}
```

## Ajax套用自行建立驗證

產生AntiForgeryToken
```C#
@functions{
    public static string GetAntiForgery()
    {
        string cookieToken, formToken;
        AntiForgery.GetTokens(null, out cookieToken, out formToken);
        return String.Concat(cookieToken, "@.@", formToken);
    }
}
```
```javascript
function deleteUser() {
    var token = $('@Html.AntiForgeryToken()').val();
    //防偽標記放入headers
    //也可以將防偽標記放入data
    $.ajax({
        type: 'POST',
        url: "/User/Delete",
        headers: { '__RequestVerificationToken': token },
        cache: false,
        data: { "id": $("#delid").val()},
        complete: function (data) {
            alert(data.responseJSON["Data"]);
            document.location.href = "/User/Index";
        }
    });
}
```
後端自行建立`Attribute`驗證
```C#
public class MyValidateAntiForgeryToken : AuthorizeAttribute
    {
        public override void OnAuthorization(AuthorizationContext filterContext)
        {
            var request = filterContext.HttpContext.Request;
            if (request.HttpMethod == WebRequestMethods.Http.Post)
            {
                if (request.IsAjaxRequest())
                {
                    var antiForgeryCookie = request.Cookies[AntiForgeryConfig.CookieName];
                    var cookieValue = antiForgeryCookie != null
                    ? antiForgeryCookie.Value
                    : null;
                    //從cookies 和 Headers 中 驗證防偽標記
                    //這裡可以加try-catch
                    AntiForgery.Validate(cookieValue, request.Headers["__RequestVerificationToken"]);
                }
                else
                {
                    new ValidateAntiForgeryTokenAttribute()
                    .OnAuthorization(filterContext);
                }
            }
        }
    }
```
```C#
[MyValidateAntiForgeryToken]
    public ActionResult Delete(string id){}
```


## 參考連結

>* [url1](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/386205/)