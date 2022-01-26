---
title: "找出form input元素並處理值集合"
description: "可模擬form submit"
date: 2022-01-26T16:49:44+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Javascript
tags: [
    "Javascript",
    "Jquery"
]
---

## 前言

找出每個Input元素並串好值可以拿來提交表單或是ajax post

## 主要內容

### 常用input tag
```html
<form method="post" id="queryForm" action="">
    @Html.AntiForgeryToken()
    <input type="color" name="color" />
    <input type="checkbox" value="1" name="chks[0]" />
    <input type="checkbox" value="2" name="chks[1]" />
    <input type="range" name="rng" />
    <select multiple name="selects">
        <option value="1">A</option>
        <option value="2">B</option>
        <option value="3">C</option>
    </select>
    <input type="radio" name="rdo" value="10" />
    <input type="radio" name="rdo" value="20" />
    <input type="radio" name="rdo" value="30" />
    <textarea name="textarea"></textarea>
    <button type="button" onclick="ppTobk();">submit</button>
</form>
```

### 共用method
```Javascript
/**
* 將值集合轉成物件
* @@param paras 值集合
*/
function paras_to_object(paras) {
    let obj = {};
    for (let i = 0; i < paras.length; i++) {
        json_data[paras[i].name] = paras[i].val;
    }
    return obj;
}
/**
    * 將input值串好form接的格式
    * @@param jqselector jq選擇器字串
    */
function form_get_paras(jqselector) {
    let paras = [];
    $(jqselector).each(function () {
        $(this).find('input,textarea').each(function () {
            let typ = $(this).attr('type');
            if (typ == "checkbox" || typ == "radio") {
                if ($(this).prop('checked')) {
                    let p = {
                        name: $(this).attr('name'),
                        val: $(this).val()
                    };
                    if (p.name) {
                        paras.push(p);
                    }
                }
            }
            else if (typ == "file" || typ == "image") { }
            else {
                if ($(this).val() && $(this).attr('name')) {
                    let p = {
                        name: $(this).attr('name'),
                        val: $(this).val()
                    };
                    paras.push(p);
                }
            }
        });
        $(this).find('select').each(function () {
            if ($(this).attr('multiple') == "multiple") {
                let valarr = $(this).val();
                for (let i = 0; i < valarr.length; i++) {
                    let p = {
                        name: $(this).attr('name') + "[" + i + "]",
                        val: valarr[i]
                    };
                    if (p.name) {
                        paras.push(p);
                    }
                }
            }
            else {
                let p = {
                    name: $(this).attr('name'),
                    val: $(this).val()
                };
                if (p.name) {
                    paras.push(p);
                }
            }
        })
    });
    return paras;
}
/**
* insert hidden inputs to form
* @@param tmpform 傳入Form
* @@param paras 參數array
*/
function form_ins_inputs(tmpform, paras) {
    for (let i = 0; i < paras.length; i++) {
        let hideInput = document.createElement("input");
        hideInput.type = "hidden";
        hideInput.name = paras[i].name;//傳入引數名
        hideInput.value = paras[i].val;//傳入資料
        tmpform.appendChild(hideInput);
    }
}
/**
* 開新窗POST
* @@param url 網址
* @@param name 自定義一個name
* @@param jqselector 抓參數的jq選擇
*/
function openPostWindow(url, name, jqselector) {
    var tempForm = document.createElement("form");
    tempForm.id = "tempForm1";
    tempForm.method = "post";
    tempForm.action = url; //url
    tempForm.target = name;

    let paras = form_get_paras(jqselector);
    form_ins_inputs(tempForm, paras);

    tempForm.addEventListener('onsubmit', function () { openWindow(name); })
    document.body.appendChild(tempForm);

    //手動submit form
    tempForm.submit();
    document.body.removeChild(tempForm);

    function openWindow(name) {
        window.open('about:blank', name);
    }
}
```

### 提交

`Sumbit`
```javascript
let $form = $('#queryForm');
$form.attr('action', "");
//$form.attr('action', url).attr('target', '_blank'); //new window
$form.submit();
```

`Open Window Submit`
```javascript
function ppTobk() {
    openPostWindow('@Url.Action("TestPost")', 'pppp', '#queryForm');
}
```
`Ajax Post`
```javascript
<form id="__AjaxAntiForgeryForm" action="#" method="post">@Html.AntiForgeryToken()</form>

var AddAntiForgeryToken = function (data) {
    data.__RequestVerificationToken = $('#__AjaxAntiForgeryForm input[name=__RequestVerificationToken]').val();
    return data; //CSRF
};
function ppTobk() {
    let paras_data = paras_to_object(form_get_paras("#queryForm"));
    $.ajax({
        type: "post",
        url: '',
        data: AddAntiForgeryToken(paras_data),
        success: function (response) {
            // ....
            console.log("OK");
        }
    });
}
```

## 小結
