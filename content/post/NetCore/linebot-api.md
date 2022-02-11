---
title: "Linebot Api 設定圖文選單"
description:
date: 2022-02-11T16:49:51+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Line
tags: [
    "Line",
    "NetCore",
]
---

## 前言

設定`Richmenu`的API網址有換過。紀錄一下操作流程

### 增加圖文選單
```markdown
POST https://api.line.me/v2/bot/richmenu  
Authorization: Bearer {channel access token}  
Content-Type: application/json  
  
成功結果回傳"richMenuId": "{rich menu id}"  
```
`Json Content Example`
```markdown
{
    "size": {
      "width": 2500,
      "height": 843
    },
    "selected": false,
    "name": "richmenu-tsai",
    "chatBarText": "選單1",
    "areas": [
      {
        "bounds": {
          "x": 0,
          "y": 0,
          "width": 833,
          "height": 843
        },
        "action": {
          "type": "message",
          "label": "指令",
          "text": "指令"
        }
      },
      {
        "bounds": {
          "x": 833,
          "y": 0,
          "width": 833,
          "height": 843
        },
        "action": {
          "type": "uri",
          "label": "google",
          "uri": "www.google.com"
        }
      },
      {
        "bounds": {
          "x": 1666,
          "y": 0,
          "width": 833,
          "height": 843
        },
        "action": {
          "type": "uri",
          "label": "yahoo",
          "uri": "www.yahoo.com.tw"
        }
      }
   ]
}
```
### 設定圖文選單的圖片
```markdown
POST https://api-data.line.me/v2/bot/richmenu/{rich menu id}/content
Authorization: Bearer {channel access token}
```

### 設定預設的圖文選單
`POST https://api.line.me/v2/bot/user/all/richmenu/{rich menu id}`

### 刪除圖文選單
`DELETE https://api.line.me/v2/bot/richmenu/{rich menu id}`

## 參考連結

>* [url1](https://ithelp.ithome.com.tw/articles/10229397)