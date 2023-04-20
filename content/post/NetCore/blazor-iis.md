---
title: "Blazor Server Iis部屬"
description: "IIS部屬Blazor相關設定，網站相對路徑"
date: 2023-04-20T15:19:02+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Blazor
tags: [
    "Blazor",
    "Net6",
]
---

## 前言

如果部屬使用網站的方式，網站路徑不會帶有應用程式別名  
```markdown
localhost:0000/
```
但如果要掛在網站下的應用程式部屬，網站根路徑就會帶有應用程式名稱  
```markdown
localhost:7128/MyBlazorApp
```

部屬後，原先開發時的導向亂掉了，連靜態檔案/js/css載入也失敗  
因為部屬環境與開發環境差了一層應用程式名的關係

## 專案設定

步驟1 : `_Layout.cshtml`  
```markdown
找到 <base href="~/"/>
改為 <base href="/MyBlazorApp/"/>
```
步驟2 : `Program.cs`  
```C#
app.UsePathBase("/MyBlazorApp");
//UseStaticFiles在上面加上
UseStaticFiles()
```
步驟3 : `開發環境預設啟動URL Properties/launchSettings.json`  
```json
"profiles": {
    "MyBlazorApp": {
      "commandName": "Project",
      "dotnetRunMessages": true,
      "launchBrowser": true,
      "applicationUrl": "https://localhost:7128;http://localhost:5258",
      "launchUrl": "https://localhost:7128/MyBlazorApp", //加上這行
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
```

## IIS部屬設定

IIS部屬的名稱大小寫要一致，否則也會請求失敗  

## 其他

1. 引入css script的相對路徑 開頭不加任何`~/`  
ex:` <script src="js/jquery-3.6.3.min.js"></script>`  
` <link href="css/site.css" rel="stylesheet" />`  
2. Razor Page 導向`NavigateTo`直接打元件名稱，不加`/`  
ex:`NavigationManager.NavigateTo("Login",true);`  
3. 使用LocalRedirect 帶 `~/`  
ex: `return LocalRedirect("~/Login");`  
4. 若有另外建立Api要呼叫時  
`URL= NavigationManager.BaseUri + "api/User/GetName";`  


## 小結

要注意的點還蠻多的，可能有更好的辦法一勞永逸但還沒找到  
總而言之要注意開發環境與部屬環境的相對路徑關係  

## 參考連結

>* [url1](https://www.c-sharpcorner.com/article/routing-101-in-blazor/)