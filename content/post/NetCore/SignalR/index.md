---
title: "Net Core 使用 SignalR實現即時通訊"
description: "SignalR的初步應用"
date: 2021-04-29T14:15:05+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "SignalR"
]
---

## 前言

在使用`SignalR`之前可以先行了解`WebSocket`的原理  
`Websocket`是一個持久化的協議，長期保持連線  
若需要回應時再進行回應。  

而除了`WebSocket`，也有`long poll`或利用`AjaxLoop`  
的方式來實現即時通訊。但另外兩種方式相較之下較為消耗資源。  

## 初始步驟

`Nuget`尋找並安裝`SignalR`相關套件  
如`Microsoft.AspNetCore.SignalR.Core`  

###  用戶端程式庫
在 [方案總管] 中，以滑鼠右鍵按一下專案，然後選取 [新增][用戶端程式庫] > 。  
在 [新增用戶端程式庫] 對話方塊中，針對 [提供者] 選取 [unpkg]。  
針對 [程式庫]，輸入 @microsoft/signalr  
選取 [選擇特定檔案]、展開 [散發者/瀏覽器] 資料夾，然後選取 signalr.js 與 signalr.min.js。  
將 [目標位置] 設定為 wwwroot/lib/signalr/，然後選取 [安裝]。  

### 建立Signal Hub

建立`Hubs`資料夾  
在 `Hubs `資料夾中，建立`ChatHub.cs`  

```C#
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}>
```

### 註冊SignalR 

在`Startup.cs > ConfigureServices`方法中加上  
```C#
  services.AddSignalR();
```
在`Startup.cs > Configure`中加上
```C#
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("/chathub");
});
```

### 用戶端Html

```html
<div class="container">
    <div class="row">&nbsp;</div>
    <div class="row">
        <div class="col-2">User</div>
        <div class="col-4"><input type="text" id="userInput" /></div>
    </div>
    <div class="row">
        <div class="col-2">Message</div>
        <div class="col-4"><input type="text" id="messageInput" /></div>
    </div>
    <div class="row">&nbsp;</div>
    <div class="row">
        <div class="col-6">
            <input type="button" id="sendButton" value="Send Message" />
        </div>
    </div>
</div>
<div class="row">
    <div class="col-12">
        <hr />
    </div>
</div>
<div class="row">
    <div class="col-6">
        <ul id="messagesList"></ul>
    </div>
</div>

</div>

<input type="button" id="btn2" value="btn2 Message" />
<script src="~/lib/jquery/dist/jquery.js"></script>
<script src="~/js/signalr/dist/browser/signalr.js"></script>

<script>
//chatHub是建立的Hub類別名稱第一個字母要小寫
var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable send button until connection is established
if (document.getElementById('sendButton') != null) {
    document.getElementById("sendButton").disabled = true;
}

//後端指定特定對象或全部send
connection.on("ReceiveMessage", function (user, message) {
    var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    var encodedMsg = user + " says " + msg;
    var li = document.createElement("li");
    li.textContent = encodedMsg;
    document.getElementById("messagesList").appendChild(li);
});

connection.start().then(function () {
    if (document.getElementById("sendButton") != null) {
        document.getElementById("sendButton").disabled = false;
    }
}).catch(function (err) {
    return console.error(err.toString());
});

if (document.getElementById('sendButton') != null) {
    document.getElementById("sendButton").addEventListener("click", function (event) {
        var user = document.getElementById("userInput").value;
        var message = document.getElementById("messageInput").value;
        //將訊息傳至後端
        connection.invoke("SendMessage", user, message).catch(function (err) {
            return console.error(err.toString());
        });
        event.preventDefault();
    });
}

</script>
```

在網頁上開啟開發人員工具F12  
查看`Console`有無錯誤訊息及正常連線


## 參考連結

>* [url1](https://iter01.com/535122.html)
>* [url2](https://docs.microsoft.com/zh-tw/aspnet/core/tutorials/signalr?view=aspnetcore-2.1&tabs=visual-studio)