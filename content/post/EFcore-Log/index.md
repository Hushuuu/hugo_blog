---
title: "EntityFramework Core使用NLlog紀錄Sql"
description: "使用Nlog來記錄EFcore下的DBCommand"
date: 2021-04-12T15:00:25+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - EntityFramework Core
tags: [
    "NLog",
    "NetCore",
]
---

## 前言
 
`EF Core5.0之後增加` `LogTo` 可使用`StramWriter`的方式寫在`DBContext.cs`裡  
但沒試出寫在`Startup.cs ConfigureServices`
```C#
private readonly StreamWriter _logStream = new StreamWriter("mylog.txt", append: true);

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(_logStream.WriteLine);

public override void Dispose()
{
    base.Dispose();
    _logStream.Dispose();
}

public override async ValueTask DisposeAsync()
{
    await base.DisposeAsync();
    await _logStream.DisposeAsync();
}
```  

普通的`Console.Write`方法
```C#
services.AddDbContext<YourContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"))
                .EnableSensitiveDataLogging(); //顯示敏感資料(參數)
    options.LogTo(Console.WriteLine, new[] { DbLoggerCategory.Database.Name }, LogLevel.Information);
});
```

## NLog來記錄

先打開`Nuget`安裝
`NLog`
`NLog.Web.AspNetCore`  

`Program.cs`
```C#
    public static void Main(string[] args)
    {
        var logger = NLogBuilder.ConfigureNLog("NLog.config").GetCurrentClassLogger();
        try
        {
            CreateHostBuilder(args).Build().Run();
        }
        catch (Exception ex)
        {
            logger.Error(ex, "Get Error.");
            throw;
        }
        finally
        {
            NLog.LogManager.Shutdown();
        }
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
    {
        webBuilder.UseStartup<Startup>()
                    .UseNLog();
    });
        
```

設定`appsettings.Development.json`的`LogLevel`
```C#
{
  "Logging": {
    "LogLevel": {
      "Microsoft.EntityFrameworkCore.Database.Command": "Debug"
    }
  }
}
```

產生`NLog.config`
```C#
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Info"
      internalLogFile="c:\temp\internal-nlog.txt">

	<!-- enable asp.net core layout renderers -->
	<extensions>
		<add assembly="NLog.Web.AspNetCore"/>
	</extensions>

	<!-- the targets to write to -->
	<targets>
		<!-- write logs to file  -->
		<target xsi:type="File" name="allfile" fileName="D:\nlog-all-${shortdate}.log"
				layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />

		<!-- another file log, only own logs. Uses some ASP.NET core renderers -->
		<target xsi:type="File" name="sql-file" fileName="D:\nlog-Sql-${shortdate}.log"
				layout="${longdate}|${event-properties:item=EventId_Id}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}|url: ${aspnet-request-url}|action: ${aspnet-mvc-action}" />
	</targets>

	<!-- rules to map from logger name to target -->
	<rules>
		<!--All logs, including from Microsoft-->
		<logger name="*" minlevel="Trace" writeTo="allfile" />

		<!-- BlackHole without writeTo -->
		<logger name="*" maxlevel="Debug" minlevel="Debug" writeTo="sql-file" />

		<!--Skip non-critical Microsoft logs and so log only own logs-->
		<logger name="Microsoft.*" maxlevel="Info" final="true" />
	</rules>
</nlog>
```

## 小結

使用`NLog`要注意的是設定的`LogLevel`層級  
`EFCore LogTo`若有試出方法再更新  


## 參考連結

>* [url1](https://docs.microsoft.com/zh-tw/ef/core/logging-events-diagnostics/simple-logging)
>* [url2](https://blog.miniasp.com/post/2020/12/12/Logging-in-Entity-Framework-Core)
>* [url3](https://dotblogs.com.tw/Null/2020/04/14/210320)