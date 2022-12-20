---
title: "IIS Allow Download Apk"
description: "站台允許下載apk類型檔案"
date: 2022-12-19T16:01:15+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - IIS
tags: [
    "IIS",
    "Android",
]
---

## 前言

設定`web.config`允許站台訪問靜態檔案類型`apk`

## 主要內容

在`web.config`底下找到此段增加或更新  

```markdown
<system.webServer>
		<staticContent>
			<mimeMap fileExtension=".apk" mimeType="application / vnd.android.package-archive"/>
		</staticContent>
	</system.webServer>
```

## 參考連結

>* [url1](https://dotblogs.com.tw/alanlun/2018/02/08/145616)