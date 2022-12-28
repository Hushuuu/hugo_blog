---
title: "Jquery Datepicker日期選單沒出現/沒反應"
description: "可能是有出現但位置不對"
date: 2022-12-28T11:25:59+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Js
tags: [
    "jquery",
    "datepicker",
]
---

## 前言

在使用`jquery datepicker`的時候遇到了日期選單沒出現沒反應的問題  
原本以為是瀏覽器的關係。後來找到原因是選單產生的位置不對  
可能是瀏覽器的文字縮放比例或是系統的縮放造成  

## 主要內容

根據參考資料，只要在`beforeShow`加上自訂的判斷來計算偏移的`offset`  
即可解決此問題

```javascript
function setDatepickerPos(input, inst) {
        var rect = input.getBoundingClientRect();
        // use 'setTimeout' to prevent effect overridden by other scripts
        setTimeout(function () {
            var scrollTop = $("body").scrollTop();
    	    inst.dpDiv.css({ top: rect.top + input.offsetHeight + scrollTop });
        }, 0);
    }
$('#MyDatepicker').datepicker({
	dateFormat: "yy-mm-dd",
	changeMonth: true,
	changeYear: true,
	defaultDate: +0,
	inline: true,
	beforeShow: function (input, inst) { setDatepickerPos(input, inst) },
});  
```

## 小結

之後再另外整併將jquery datepicker包進vue的conpoment版本

## 參考連結

>* [url1](https://aries.me/2020/08/03/jquery-ui-1-12-datepicker-wrong-position-issue-and-fix/)