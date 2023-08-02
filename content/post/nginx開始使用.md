---
title: "Nginx開始使用"
description:
date: 2023-07-11T08:42:15+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Nginx
tags: [
    "Nginx",
    "publish",
]
---

## 指令

開啟程式點擊`nginx.exe`  
或`$ start nginx`

重載 Nginx 設定檔&啟動  
`$ nginx -s reload`

停止 Nginx  
`$ nginx -s stop`

測試 Nginx 的設定檔並顯示狀態  
`$ nginx -t` 

## 設定檔

Nginx預設設定檔位置在`\conf\nginx.conf`  
```conf
 server{
	listen	8100; #監聽port
	server_name	localhost; #server網址
	
	location / { #網址判讀 / or xxx/
            root   D:\MVC\2023\MyVueTest1\vueapp\dist; #根目錄位址
            index  index.html index.htm; #頁面
	        try_files $uri $uri/ /index.html; #vue spa使用這行來處理頁面導回index
        }

    }


```
## 平衡附載
在`config`設定  
```conf
 upstream ServiceInstance{  
    #透過upstream 節點定義多個server來使用，實現平衡負載
    #可設定權重weight/ip_hash等參數
    server localhost:9529;  
	server localhost:9530; 
    } 
    server{
	listen	8099; #監聽port
	server_name	localhost; #server網址
	
	location / { #網址判讀 / or xxx/
            proxy_pass http://ServiceInstance/ ; #proxy代理轉處理
        }

    }
```

## 常見錯誤

`啟動錯誤An attempt was made to access a socket in a way forbidden by its access permissions`  

可以先排查哪個process占用了要監聽的port  
`$ netstat -aon | findstr "1234"`

若找不清楚可以試重啟 WinNAT 服務  
`net stop winnat`
`net start winnat`


## 參考連結

>* [url1](https://hoohoo.top/blog/nignx-module-upstream-simple-load-balancing/)