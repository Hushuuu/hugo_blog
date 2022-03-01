---
title: "NetCore使用Appsetting"
description: "注入強型別"
date: 2022-02-17T09:45:19+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "NetCore",
    "Appsetting",
]
---

## 主要內容

NetCore在取得
```C#
//預設Startup已有注入Configuration
public IConfiguration Configuration { get; }
public IWebHostEnvironment _env { get; set; }
public Startup(IConfiguration configuration, IWebHostEnvironment env)
{
    Configuration = configuration;
    _env = env;
}
//一般取值
_config.GetValue<string>("AAAAA");
```
### 強型別取得
`AppSetting.cs建立靜態類別`
```C#
  public static class AppSetting
    {
        private static IConfiguration config;
        public static void Initialize(IConfiguration Configuration)
        {
            config = Configuration;
        }
        private static string GetAppSettings(string name)
        {
            return config.GetValue<string>(name);
        }
        #region 平台相關設定


        /// <summary>
        /// log 保留天數
        /// </summary>
        public static int LogExistDays
        {
            get
            {
                int days = 7;
                int.TryParse(GetAppSettings("LogExistDays"), out days);
                return days;
            }
        }

        /// <summary>
        /// 是否啟用web request log
        /// </summary>
        public static bool IsRequestLog_ForWeb
        {
            get
            {
                bool enable = false;
                string tmp = GetAppSettings("IsRequestLog_WEB");
                if (!string.IsNullOrEmpty(tmp) && !bool.TryParse(tmp, out enable))
                {
                    enable = tmp == "Y" ? true : false;
                }
                return enable;
            }
        }

        #endregion
    }
```

`在Startup裡面注入`
```C#
public void ConfigureServices(IServiceCollection services)
{
    AppSetting.Initialize(Configuration);
}
```