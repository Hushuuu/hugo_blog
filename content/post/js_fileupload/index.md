---
title: "MVC-檔案上傳Post"
description: "上傳以及接收的方法。"
date: 2021-03-25T10:44:23+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "file",
    "javascript",
]
---

## FormPost

如果要用FormPost的方式上傳檔案  
注意將Form的`Content-Type `設定為`enctype="multipart/form-data"`  
`<input type="file" accept=".png, .jpg, .jpeg" name="Filedata" class="btn btn-default" />`  
`FormData`Post到後端後可以這樣來接收  
```C#
public ActionResult Create(HttpPostedFileBase Filedata)
{
    //可利用正則式過濾副檔名
if (Filedata != null &&
    Regex.IsMatch(Path.GetExtension(Filedata.FileName), "^[.](jpg|png|jpeg|gif|bmp)$", RegexOptions.IgnoreCase))
    {
        //存到資料夾
        string tmpPath1 = Server.MapPath("~/Images/");
        if (!Directory.Exists(tmpPath1))
            Directory.CreateDirectory(tmpPath1);
        var FileName = Path.GetFileName(Filedata.FileName);
        var FilePath = Path.Combine(Server.MapPath("~/Images/"), FileName);
        Filedata.SaveAs(FilePath);
        string save_path = "/Images/" + FileName;
        pcm.pic_path = save_path;
    }
}
```

## XMLHttpRequest上傳

建立`FormData`利用`XMLHttpRequest`發出請求  
監聽上傳事件可顯示出上傳進度    
```javascript
//上傳檔案
var file = null;
$('#Filedata1')[0].addEventListener('change', readFile, false);
    function readFile() {
        file = this.files[0];
    }
function upload() {
    let xhr = new XMLHttpRequest();
    let fd = new FormData();
    fd.append("fileName", file);
    //監聽事件
    xhr.upload.addEventListener("progress", uploadProgress, false);
    //傳送檔案和表單自定義引數
    xhr.overrideMimeType("application/json");
    xhr.open("POST", "@Url.Action("XHR_UPLOAD","HOME")", true);
    xhr.send(fd);
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            let response = JSON.parse(xhr.responseText);
            let pic_path = response["imgSrc"];
            $("#pic_path").val(pic_path);
            $("#uploadModal").modal("hide");
            $("#img1").show();
            $("#img1").attr('src', pic_path);
        }
    };
}
function uploadProgress(evt) {
    if (evt.lengthComputable) {
        //evt.loaded：檔案上傳的大小 evt.total：檔案總的大小
        var percentComplete = Math.round((evt.loaded) * 100 / evt.total);
        //載入進度條，同時顯示資訊
        $("#percent").html(percentComplete + '%');
        $("#progressNumber").css("width", percentComplete + "px");
    }
}
```
進度條顯示  
```html
<div style="background:#848484;width:100px;height:10px;margin-top:5px">
    <div id="progressNumber" style="background:#428bca;width:0px;height:10px">
    </div>
</div>
<font id="percent">0%</font>
```
後端  
```C#
public ActionResult XHR_UPLOAD()
{
    //Request取得上傳的File
    //回傳上傳後檔案路徑及資訊JsonString
    string str = "";
    if (Request.Files.Count > 0)
    {
        str = "{";
        //當資料夾不存在時，建立此資料夾
        if (!Directory.Exists(Server.MapPath("~/Images/")))
            Directory.CreateDirectory(Server.MapPath("~/Images/"));
        HttpPostedFileBase f = Request.Files[0];
        string file_name = f.FileName;
        string file_path = Server.MapPath("~/Images/") + file_name;
        string save_path = "/Images/" + file_name;
        f.SaveAs(file_path);
        str += "\"imgSrc\":\"" + save_path + "\",";
        str += "\"imgName\":\"" + file_name + "\"";
        str += "}";
    }
    return Content(str);
}
```

## 小結

也可以試試`Ajax`的方式上傳檔案  
要注意就是後端能不能接到`File`  
接不到就需調整參數如`Post`的`Content-type`  

