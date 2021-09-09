---
title: "Notification - IOS篇"
description:
date: 2021-09-09T10:51:04+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Notification
tags: [
    "Notification",
    "C#",
]
---

## 前言

`IOS`使用`APNS server`要額外使用組件`HttpTwo` `HttpTwo.HPack` `jose-jwt`  

## 主要內容

首先要先拿到`Apple Developer`的`apns key .p8檔`  
建立`HttpClient`物件使用`HTTP/2`  
讀取`.p8`檔並使用`jose-jwt`來產出`authorization token`  

### 類別建立

```C#
//回傳結果+狀態碼
public class AppleSendResult
{
    public bool isSuccess { get; set; }

    public string message { get; set; }

    public Guid? apns_id { get; set; }

    public HttpStatusCode httpStatusCode { get; set; }

    public string reason { get; set; }

    //記著request的device token送完可判斷狀態碼來得知是否token還有效
    public string token { get; set; } 
}
//Request 模型
public class APNAps
{
    //alert 可為object or string直接當訊息body
    public string alert { get; set; } 
}
//Request
public class APNSendRequest
{
    //aps的物件結構可參照官方文件增加其他屬性
    public APNAps aps { get; set; }
}
//推送的種類(一般為alert)
public enum ApplePushTypeKind
{
    alert,
    background,
    voip,
    complication,
    fileprovider,
    mdm,
}
private string auth_kid { get; set; } //apns auth key keyID
private string auth_iss { get; set; } //開發者帳號TeamID
private string apns_topic { get; set; } //開發專案名稱
private string accessToken { get; set; } //token
private DateTime? accessTokenValidTime { get; set; } //token合法時間
```
### 讀取.p8檔產出Token

```C#
/// <summary>
/// 重新產生access token
/// </summary>
public  virtual void ReNewAccessToken()
{
    //取得產生Token的時間(UTC)秒為單位
    var expiration = DateTime.Now.ToUniversalTime() - new DateTime(1970, 1, 1, 0, 0, 0, 0, DateTimeKind.Utc);
    var expirationSeconds = (long)expiration.TotalSeconds;

    var header = new Dictionary<string, object>();
    header.Add("alg", "ES256"); //加密演算法
    header.Add("kid", auth_kid);

    string payload = "{\"iss\":\"" + auth_iss + "\",\"iat\":\"" + expirationSeconds + "\"}";

    accessToken = Jose.JWT.Encode(payload, scrt, Jose.JwsAlgorithm.ES256, header);

    accessTokenValidTime = DateTime.Now.AddMinutes(60);
}
/// <summary>
/// 使用認證令牌
/// </summary>
/// <param name="p8FilePath">p8檔案路徑</param>
/// <param name="apns_topic">bundle id</param>
/// <param name="auth_kid">kid</param>
/// <param name="auth_iss">iss</param>
public virtual void UseAuthorization(string p8FilePath, string apns_topic, string auth_kid, string auth_iss)
{
    var text = File.ReadAllText(p8FilePath);
    List<string> split = text.Split('\n').ToList();
    split.RemoveAt(split.Count - 1);
    split.RemoveAt(0);
    var k = string.Join("", split);
    var bytes = Convert.FromBase64String(k);
    var secrt = CngKey.Import(bytes, CngKeyBlobFormat.Pkcs8PrivateBlob);

    this.scrt = secrt;
    this.auth_kid = auth_kid;
    this.auth_iss = auth_iss;
    this.apns_topic = apns_topic;

    ReNewAccessToken();
}
```

### Http/2處理

```C#
public class Http2CustomHandler : WinHttpHandler
{
    protected override Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, System.Threading.CancellationToken cancellationToken)
    {
        request.Version = new Version("2.0");
        return base.SendAsync(request, cancellationToken);
    }
}
```

### 推送方法
```C#
public virtual AppleSendResult Send<T>(string token, T postdataModel
            , ApplePushTypeKind pushTypeKind = ApplePushTypeKind.alert, int apns_expiration = 0, int apns_priority = 10)
            where T : APNSendRequest
{

    httpClient = new HttpClient(new Http2CustomHandler());

    if (DateTime.Now >= accessTokenValidTime) { ReNewAccessToken(); }

    AppleSendResult result = new AppleSendResult()
    {
        token = token,
    };
    //若是正式環境要將網址的sandbox拿掉  
    string url = $"https://api.sandbox.push.apple.com/3/device/{token}";  

    var uri = new Uri(url);

    string postdata_json = JsonConvert.SerializeObject(postdataModel);
    byte[] byteArray = Encoding.UTF8.GetBytes(postdata_json);

    var httpRequestMessage = new HttpRequestMessage();

    httpRequestMessage.RequestUri = uri;
    httpRequestMessage.Method = HttpMethod.Post;

    httpRequestMessage.Headers.Add("apns-push-type", pushTypeKind.ToString());
    httpRequestMessage.Headers.Add("apns-expiration", apns_expiration.ToString());
    httpRequestMessage.Headers.Add("apns-priority", apns_priority.ToString());
    httpRequestMessage.Headers.Add("authorization", $"bearer {accessToken}");
    httpRequestMessage.Headers.Add("apns-topic", this.apns_topic);
    httpRequestMessage.Headers.Add("apns-id", Guid.NewGuid().ToString());

    httpRequestMessage.Content = new ByteArrayContent(byteArray);

    var responseMessage = httpClient.SendAsync(httpRequestMessage).Result;

    var responseBody = responseMessage.Content.ReadAsStringAsync().Result;

    HttpStatusCode responseCode = responseMessage.StatusCode;
    result.httpStatusCode = responseCode;

    string tmp_apns_id = responseMessage.Headers.GetValues("apns-id").First();
    if(Guid.TryParse(tmp_apns_id, out Guid res_apns_id))
    {
        result.apns_id = res_apns_id;
    }
    if (responseCode != HttpStatusCode.OK)
    {
        var responseObj = JsonConvert.DeserializeObject<APNSendResult>(responseBody);
        result.reason = responseObj.reason;
        result.isSuccess = false;
        result.message = "失敗。" + responseObj.reason;
    }
    else
    {
        result.isSuccess = true;
        result.message = "成功。";
    }
    return result;
}

```

## 小結

當通知傳到裝置後，根據`Response`可以知道是否成功及`DeviceToken`有效性  
`Request`的模型還有更多參數可以設定   
較詳細的處理就需要看官方文件慢慢調整了。  

## 參考連結

>* [url1](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification)
>* [url2](https://medium.com/@josephchen.jojo/%E5%9F%BA%E6%96%BChttp-2%E7%9A%84ios-apns-%E6%8E%A8%E6%92%AD-f39c6b31eb51)
>* [url3](http://monkeybinbin-blog.logdown.com/posts/1120802-apns-token-based-authentication-csharp)