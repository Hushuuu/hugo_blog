---
title: "NetCore-專案檔設定"
description: "不重新覆蓋避免覆蓋webconfig"
date: 2022-03-03T10:38:39+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - NetCore
tags: [
    "NetCore"
    
]
---

## 主要內容

專案下右鍵=>編輯專案檔
```C#
//不重新編譯webconfig
<!--防止 Web SDK 轉換檔案 web.config-->
<PropertyGroup>
<TargetFramework>net5.0</TargetFramework>
    <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
//發行不覆蓋
 <ItemGroup>
	    <MsDeploySkipRules Include="CustomSkipFolder">  <!--不覆蓋此目錄下的檔案-->
		    <ObjectName>dirPath</ObjectName>
		    <AbsolutePath>wwwroot\\test</AbsolutePath>
	    </MsDeploySkipRules>
    </ItemGroup>
```