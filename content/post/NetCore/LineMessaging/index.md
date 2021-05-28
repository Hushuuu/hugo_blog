---
title: "NetCore上建立LINE BOT"
description: "使用LineMessaging Api開發"
date: 2021-05-27T10:09:55+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "LineBot",
]
---

## 前言

LINE官方有不同語言的API套件，C#的話則有社群提供。主要為`pierre3`提供的`LineMessaging Api`和`twdeveloper`提供的`LineBotSDK`可選用  
本篇使用`LineMessaging Api`  

## 初始設計

建立一個新專案，範本選`.net core Web`空白的專案  
`Nuget`安裝`Line.Messaging`

將`Linebot`的`Channel secret`和`Channel access token`放在 `appsettings.json`  
```c#
"LineBot": {
    "channelSecret": "xxx",
    "accessToken": "xxx"
  }
```
建立`LineBotConfig.cs`
```C#
public class LineBotConfig
{
    public string channelSecret { get; set; }
    public string accessToken { get; set; }
}
```
設定`Startup.cs`
```C#
public Startup(IConfiguration configuration)
{
    Configuration = configuration;
}

public IConfiguration Configuration { get; }
// This method gets called by the runtime. Use this method to add services to the container.
// For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<LineBotConfig, LineBotConfig>((s) => new LineBotConfig
    {
        channelSecret = Configuration["LineBot:channelSecret"],
        accessToken = Configuration["LineBot:accessToken"]
    });

    services.AddHttpContextAccessor();
    services.AddRazorPages().AddRazorRuntimeCompilation();
    services.AddHttpClient();
}

// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "api",
            pattern: "api/{controller=Home}/{action=Index}/{id?}");
        endpoints.MapRazorPages();
    });
}
```
### 建立LineBotApp.cs 
```C#
public class LineBotApp : WebhookApplication
    {
        private readonly LineMessagingClient _messagingClient;
        private readonly IHttpClientFactory _clientFactory;
        private readonly ILogger _logger;
        private Dictionary<string, object> userParams;
        private string conversationId; // DirectLine ConversationId
        public LineBotApp(LineMessagingClient lineMessagingClient, IHttpClientFactory clientFactory, ILogger logger)
        {
            _messagingClient = lineMessagingClient;
            _clientFactory = clientFactory;
            _logger = logger;
        }
        protected override async Task OnMessageAsync(MessageEvent ev)
        {
            try
            {
                var result = null as List<ISendMessage>;
                switch (ev.Message)
                {
                    //文字訊息
                    case TextEventMessage textMessage:
                        {
                            //頻道Id
                            var channelId = ev.Source.Id;
                            //使用者Id
                            var userId = ev.Source.UserId;
                            result = new List<ISendMessage> { new TextMessage($"you said: {ev.Message.Text}") };
                        }
                        break;
                                             
                }
                if (result != null)
                    await _messagingClient.ReplyMessageAsync(ev.ReplyToken, result);

            }
            catch (Exception ex)
            {
                _logger.LogError(JsonConvert.SerializeObject(ex));
            }
        }       
    }
```
### 建立`Controller/LineBotController.cs`
```C#
[Route("api/linebot")]
public class LineBotController : Controller
{
    private readonly IHttpContextAccessor _httpContextAccessor;
    private readonly HttpContext _httpContext;
    private readonly LineBotConfig _lineBotConfig;
    private readonly ILogger _logger;
    private readonly IHttpClientFactory _clientFactory;
    public LineBotController(IServiceProvider serviceProvider,
        LineBotConfig lineBotConfig, ILogger<LineBotController> logger, IHttpClientFactory clientFactory)
    {
        _httpContextAccessor = serviceProvider.GetRequiredService<IHttpContextAccessor>();
        _httpContext = _httpContextAccessor.HttpContext;
        _lineBotConfig = lineBotConfig;
        _logger = logger;
        _clientFactory = clientFactory;
    }

    //完整的路由網址就是 https://xxx/api/linebot/run
    [HttpPost("run")]
    public async Task<IActionResult> Post()
    {
        try
        {
            var events = await _httpContext.Request.GetWebhookEventsAsync(_lineBotConfig.channelSecret);
            var lineMessagingClient = new LineMessagingClient(_lineBotConfig.accessToken);
            var lineBotApp = new LineBotApp(lineMessagingClient, _clientFactory, _logger);
            await lineBotApp.RunAsync(events);
        }
        catch (Exception ex)
        {
            //需要 Log 可自行加入
            //_logger.LogError(JsonConvert.SerializeObject(ex));
        }
        return Ok();
    }
}
```
`GetWebhookEventsAsync`方法因為`NetCore`似乎不使用`HttpRequestMessage`了所以需要根據`request`型別擴充方法  
使用`參考文章作者Mir`調整後的方法  
```c#
  public static class WebhookRequestMessageHelper
    {
        public static async Task<IEnumerable<WebhookEvent>> GetWebhookEventsAsync(this HttpRequest request, string channelSecret, string botUserId = null)
        {
            if (request == null) { throw new ArgumentNullException(nameof(request)); }
            if (channelSecret == null) { throw new ArgumentNullException(nameof(channelSecret)); }

            var content = "";
            using (var reader = new StreamReader(request.Body))
            {
                content = await reader.ReadToEndAsync();
            }

            var xLineSignature = request.Headers["X-Line-Signature"].ToString();
            if (string.IsNullOrEmpty(xLineSignature) || !VerifySignature(channelSecret, xLineSignature, content))
            {
                throw new InvalidSignatureException("Signature validation faild.");
            }

            dynamic json = JsonConvert.DeserializeObject(content);

            if (!string.IsNullOrEmpty(botUserId))
            {
                if (botUserId != (string)json.destination)
                {
                    throw new UserIdMismatchException("Bot user ID does not match.");
                }
            }
            return WebhookEventParser.ParseEvents(json.events);
        }

        internal static bool VerifySignature(string channelSecret, string xLineSignature, string requestBody)
        {
            try
            {
                var key = Encoding.UTF8.GetBytes(channelSecret);
                var body = Encoding.UTF8.GetBytes(requestBody);

                using (HMACSHA256 hmac = new HMACSHA256(key))
                {
                    var hash = hmac.ComputeHash(body, 0, body.Length);
                    var xLineBytes = Convert.FromBase64String(xLineSignature);
                    return SlowEquals(xLineBytes, hash);
                }
            }
            catch
            {
                return false;
            }
        }

        private static bool SlowEquals(byte[] a, byte[] b)
        {
            uint diff = (uint)a.Length ^ (uint)b.Length;
            for (int i = 0; i < a.Length && i < b.Length; i++)
                diff |= (uint)(a[i] ^ b[i]);
            return diff == 0;
        }
    }
```
### Nlog
順勢加入`Nlog`來記錄錯誤及可以`debug`  
路徑設定為方便部屬至`Azure`時查閱
```c#
//nlog.config
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Info"
      internalLogFile="D:\home\site\AspNetCoreNlog\Logs\internal-nlog.txt">

	<extensions>
		<add assembly="NLog.Web.AspNetCore"/>
	</extensions>

	<targets>
		<target xsi:type="File" name="allfile" fileName="D:\home\site\AspNetCoreNlog\Logs\all\nlog-all-${shortdate}.log"
				layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />

		<target xsi:type="File" name="ownFile-web" fileName="D:\home\site\AspNetCoreNlog\Logs\own\nlog-own-${shortdate}.log"
				layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}|url: ${aspnet-request-url}|action: ${aspnet-mvc-action}" />
	</targets>

	<rules>
		<logger name="*" minlevel="Trace" writeTo="allfile" />
		<logger name="*" minlevel="Trace" writeTo="ownFile-web" />
	</rules>
</nlog>
```
`Program.cs`
```c#
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseNLog()//加入NLOG
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```
### 部屬至Azure

建立新的`App service`選擇免費方案  
建立好之後回專案視窗->建置->發行  
發行至`Azure`->選建立的`app service`  
發行成功後再到`Line official account manager`  
設定`webhook`網址`https://xxx/api/linebot/run`  
發送訊息測試
## 小結
下方參考連結作者`fysh711426`提供的範例及圖文更加豐富，此篇只為記錄個人實作上的步驟
## 參考連結

>* [url1](https://ithelp.ithome.com.tw/articles/10217452)