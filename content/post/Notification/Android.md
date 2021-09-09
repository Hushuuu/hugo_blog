---
title: "Notification - Android篇"
description:
date: 2021-09-07T09:45:46+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Notification,
tags: [
    "Notification",
    "C#",
]
---

## 前言

`Android`用`FCM`傳送通知步驟較`IOS Apns`為簡易。  
`Request`加`ApiKey`驗證傳給FCM Server即可  

## 主要內容

須先取得`Firebase` `Android`開發的`api key`  
詳閱`Firebase Cloud Messaging(FCM) `官方文件後，建立傳入傳出的類別  
`request`較重要的欄位為`registration_ids`代表欲傳送的`device id`  
傳送訊息`FCM`有定義兩種`type`，`notification message `和`data message `  
根據使用的種類，App接收端的處理也稍有不同，接收到並解析來顯示通知  
此篇不多介紹App端的處理

`定義類別`
```C#
//回傳結果
 public class FCMResultData
{
    public string error { get; set; }

    public string message_id { get; set; }

    public string registration_id { get; set; }

}
//FCM回傳結果
public class FCMSendResult
{
    public string multicast_id { get; set; }

    public int? success { get; set; }

    public int? failure { get; set; }

    public int? canonical_ids { get; set; }

    public string message { get; set; }

    public List<FCMResultData> results { get; set; }

}
//紀錄結果+狀態碼
public class AndroidSendResult : FCMSendResult
{
    public bool isSuccess { get; set; }

    public string resultMessage { get; set; }

    public HttpStatusCode httpStatusCode { get; set; }

}
//Request
public class FCMSendData
{
    public string tickerText { get; set; }

    public string contentTitle { get; set; }

    public string message { get; set; }

}
public class FCMSendRequest<T>
        where T : FCMSendData
{
    public List<string> registration_ids { get; set; }

    public T data { get; set; } //data message
}

```
```C#

public virtual AndroidSendResult SendList<T>(List<string> registration_ids, T send_data)
    where T : FCMSendData
{
    AndroidSendResult result;
    //fcm推播網址
    var uri = new Uri("https://fcm.googleapis.com/fcm/send");

    FCMSendRequest<T> post_data = new FCMSendRequest<T>()
    {
        registration_ids = registration_ids,//裝置id清單
        data = send_data, //推播訊息模型
    };

    string postdata_json = JsonConvert.SerializeObject(post_data);

    var httpRequestMessage = new HttpRequestMessage();

    httpRequestMessage.RequestUri = uri;
    httpRequestMessage.Method = HttpMethod.Post;
    httpRequestMessage.Headers.Add("Authorization", $"key ={api_key}");//fcm token key

    httpRequestMessage.Content = new StringContent(postdata_json, Encoding.UTF8, "application/json");
    
    httpClient = new HttpClient();
    httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    var responseMessage = httpClient.SendAsync(httpRequestMessage).Result;
    var responseBody = responseMessage.Content.ReadAsStringAsync().Result;
    //推送結果
    result = JsonConvert.DeserializeObject<AndroidSendResult>(responseBody);

    HttpStatusCode responseCode = responseMessage.StatusCode;
    result.httpStatusCode = responseCode;

    if (result.success.HasValue)
    {
        if(result.success >= 1)
        {
            result.isSuccess = true;
            result.resultMessage = "OK";
        }
        else
        {
            result.isSuccess = false;
            result.resultMessage = "無成功數量。";
        }
    }
    else
    {
        result.isSuccess = false;
        result.resultMessage = "有異常結果，無回應成功或失敗數量。";
    }

    return result;
}
```

## 小結

當通知傳到裝置後，根據`Response`可以知道是否成功及`DeviceID`有效性  
`Request`的模型還有更多參數可以設定   
較詳細的處理就需要看官方文件慢慢調整了。  


## 參考連結

>* [url1](https://firebase.google.com/docs/cloud-messaging/http-server-ref)
>* [url2](https://medium.com/%E7%A8%8B%E5%BC%8F%E8%A3%A1%E6%9C%89%E8%9F%B2/android-fcm-notification-message-vs-data-message-368dc0d4c1b4)