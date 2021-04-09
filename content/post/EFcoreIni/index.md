---
title: "EntityFramework Core開始使用"
description: "初步設定及使用"
date: 2021-04-09T10:17:00+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Net Core
tags: [
    "Net Core",
    "EntityFramework Core",
]
---

## 前言

.NetCore(net5)  
Entity Framework Core  
紀錄如何初始化

## 主要內容

使用`PackageManager`安裝  
```C#
EntityFrameworkCore.SqlServer
EntityFrameworkCore.Design
EntityFrameworkCore
EntityFrameworkCore.SqlServer.Tools
```
建立 DB模型cs  
```C#
public class Blog
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```
建立`DbContext.cs`
```C#
 public partial class Test2021Context : DbContext
    {
        public Test2021Context()
        {
        }

        public Test2021Context(DbContextOptions<Test2021Context> options)
            : base(options)
        {
        }

        public virtual DbSet<Blog> Blogs { get; set; }

        //相依注入後此方法可註解掉
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            if (!optionsBuilder.IsConfigured)
            {
                optionsBuilder.UseSqlServer("Server=.\\sqlexpress;Database=Test2021;Trusted_Connection=True;MultipleActiveResultSets=true");
            }
        }
    }

```

進行`Mirgration`及同步資料庫  
```C#
PM> Add-Migration InitialCreate
PM> Update-Database -V
//若不要直接更新要產出SQL
PM> Migration-Script
//可手動調整Migration出來的cs檔再做Update
```

`DbContext`和模型也可以透過逆向工程來產生  
`-force`覆寫現有的檔案  
`-outputdir`產出資料夾
`-context`名稱預設為dbnameContext.cs
```C#
Scaffold-DbContext 'Server=.\sqlexpress;Database=Test1202;Trusted_Connection=True;MultipleActiveResultSets=true' Microsoft.EntityFrameworkCore.SqlServer -OutputDir Data -Force
```


將連線字串放進`appsettings.json`  
```C#
"ConnectionStrings": {
    "DefaultConnection": "Server=localhost\\SQLEXPRESS;Database=Test2021;Trusted_Connection=True"
  }
```

### 相依注入
`Startup.cs`  
```C#
public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews();
            services.AddDbContext<Test2021Context>(options =>
            {
                //啟用 Logging 觀察 SQL 指令
                //連參數一起顯示
                options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"))
                .UseLoggerFactory(LoggerFactory.Create(builder => builder.AddConsole()
                            .AddDebug()
                            .AddFilter(level=>level==LogLevel.Information))); //LoggerFactory方式
                .LogTo(Console.Write, new[] { DbLoggerCategory.Database.Name }, LogLevel.Information) //LogTo的方式
                .EnableSensitiveDataLogging();
            });
        }

public void Configure(IApplicationBuilder app, IWebHostEnvironment env,Test2021Context dbContext)
{
    // 建立資料庫            
    dbContext.Database.EnsureCreated();
    //略
}
```
`Controller`可以套用範本直接產生出CRUD再微調，效率好很多  
範本也會幫忙建立建構子及注入
```C#
public class BlogController : Controller
{
    private readonly Test2021Context _context;

    public BlogController(Test2021Context context)
    {
        _context = context;
    }
}
```

## 小結

紀錄指令  
```C#
//逆向工程產生DbContext及模型
Scaffold-DbContext 'Server=.\sqlexpress;Database=Test1202;Trusted_Connection=True;MultipleActiveResultSets=true' Microsoft.EntityFrameworkCore.SqlServer -OutputDir Data -Force
//CodeFirst
Add-Migration InitialCreate
Update-Database -V
//若不要直接更新要產出SQL
Script-Migration 
//可手動調整Migration出來的cs檔再做Update
```

## 參考連結

>* [url1](https://blog.darkthread.net/blog/efcore-notes-1/)
>* [url2](http://greens2314.blogspot.com/2018/09/aspnet-core-2-api-entity-framework-core.html)
>* [url3](https://blog.miniasp.com/post/2020/12/12/Logging-in-Entity-Framework-Core)