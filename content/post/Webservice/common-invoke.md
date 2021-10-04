---
title: "WebService 建立和呼叫"
description: "建立和一般呼叫的介紹"
date: 2021-10-04T17:09:11+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Webservice
tags: [
    "Webservice",
    "C#",
]
---

## 前言

`Webservice`有點像是`Web API`的前身  
如果了解`Web API`的原理很快就能上手  

## 建立

`VisualStudio`開新專案->web應用程式->空的專案  
加入->新增項目->Web服務->.asmx檔案

建立的檔案會預設給一個`HelloWorld`的`Method`
```C#
 /// <summary>
    ///WebService1 的摘要描述
    /// </summary>
    [WebService(Namespace = "http://tempuri.org/")]
    [WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
    [System.ComponentModel.ToolboxItem(false)]
    // 若要允許使用 ASP.NET AJAX 從指令碼呼叫此 Web 服務，請取消註解下列一行。
    // [System.Web.Script.Services.ScriptService]
    public class WebService1 : System.Web.Services.WebService
    {

        [WebMethod]
        public string HelloWorld(string name)
        {
            return "Hello World " + name;
        }
    }
```

專案建置起來並到此asmx頁面  
可見到此`Webservice`的資訊  
點進`HelloWorld Method`後可見呼叫的格式  
其中的`<name></name>`就是我們要傳入的參數

```xml
POST /WebService1.asmx HTTP/1.1
Host: localhost
Content-Type: text/xml; charset=utf-8
Content-Length: length
SOAPAction: "http://tempuri.org/HelloWorld"

<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <HelloWorld xmlns="http://tempuri.org/">
      <name>string</name>
    </HelloWorld>
  </soap:Body>
</soap:Envelope>
```

### PostMan

首先介紹如何用`PostMan`來呼叫剛剛建立的`WebService`  
1.Post的Url是建置運行後的asmx網址`https://localhost:44370/WebService1.asmx`  
2.`Header`的設定照著`asmx`頁的資訊設定如`Content-Type:text/xml; charset=utf-8`，`Host: localhost`，`SOAPAction: "http://tempuri.org/HelloWorld"`  
3.`Body`貼上整段`xml`範本並填上參數的部分，格式選`XML`  
4.`Send Request`後成功後正常也會得到XML格式的回應    

### 加入服務參考

另一個方式為在需要Call web service的專案下`加入服務參考`  
1.專案加入->服務參考  
2.位址填入asmx網址`https://localhost:44370/WebService1.asmx`  
3.命名空間`ServiceReference1`可修改  
4.確定加入後專案的`Connected Services`就會增加  

```C#
ServiceReference1.WebService1SoapClient client = new ServiceReference1.WebService1SoapClient();
string rs = client.HelloWorld("kevin"); //rs = Hello World kevin  
```


