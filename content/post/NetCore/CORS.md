---
title: "Netcore處理跨域請求"
description: "EnableCors"
date: 2022-02-17T08:40:41+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Cors
tags: [
    "Cors",
    "NetCore",
]
---

## 主要內容

一般不允許跨域的請求，若有前後端放在不同網域或特殊用途可用
```C#
//startup.cs
public void ConfigureServices(IServiceCollection services)
{
    //Cors允許跨域原則
    services.AddCors(options =>
    {
        options.AddPolicy(name: "CustomPolicy1", //規則名稱
        builder =>
        {
            builder
                //.AllowAnyOrigin()
                .WithOrigins("https://abc.com")//不可有尾巴斜線
                .AllowAnyHeader() //或特定header
                .AllowAnyMethod();
        });
    });
}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
app.UseRouting();
app.UseCors();//一定要在userouting和useendpoing之間
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllerRoute(
        name: "default",
        pattern: "{controller=Home}/{action=Index}/{id?}");
});
}

```
`掛載至Action`
```C#
////使用時只要掛Action上或Controller...等
[EnableCors("CustomPolicy1")]
public ActionResult Post(){

}
```  

### 小結

跨域請求測試用`PostMan`是測不出來的  
正確跨域請求回傳的`header`應會有`Access-Control-Allow-Origin`
`Access-Control-Allow-Origin: *`則為全部  
