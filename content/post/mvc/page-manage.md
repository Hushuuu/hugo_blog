---
title: "MVC-頁面及權限管理"
description: "利用資料表建立頁面間父子關係，再用群組的概念將頁面做權限管控"
date: 2021-03-16T08:49:37+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
---

## 前言

此篇根據頁面資料表間的父子關係，利用遞迴來產生Menu的選單  
並透過群組來進行頁面權限的管控  

## 主要內容

基本的頁面資料表結構，重要的是`parent_id`代表此頁面父階層頁面的`page_id`  
```C#
public class PageModel
{
    [Display(Name = "頁號")]
    public string page_id { get; set; }
    [Display(Name = "頁面名稱")]
    public string page_name { get; set; }
    [Display(Name = "Controller")]
    public string controller { get; set; }
    [Display(Name = "Action")]
    public string action { get; set; }
    [Display(Name = "父頁面")]
    public string parent_id { get; set; }
}
```
產生選單的部分使用`bootstrap`的`navbar`  
並透過`Partial View`的方式來載入管控的頁面  
```C#
<div class="container">
    <div class="navbar-header">
        <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
        </button>
        @Html.ActionLink("Home", "Index", "Home", new { area = "" }, new { @class = "navbar-brand" })
    </div>
    <div class="navbar-collapse collapse">
        @Html.Partial("_MenuPart")
    </div>
</div>
```
需先在登入後產生`Menu`前將`Page`的資料List放進`Session`  
`Partial View`使用遞迴尋找父子關係來產生節點
```c#
@{
    List<PageModel> nodeList = Session["page_list"] as List<PageModel>;
    if(nodeList == null)
    {
        nodeList = new List<PageModel>();
    }
}
@TopMenu(nodeList)
@helper TopMenu(List<PageModel> nodeList)
{
     //沒有父節點代表最上方MENU
     var NoParentList = nodeList.Where(x => x.parent_id == null ).ToList();
    <ul class="nav navbar-nav">
        @foreach (GroupLoginDtl node in NoParentList)
        {
            //此節點沒有孩子代表自身為連結
            if (!nodeList.Any(z => z.parent_id == node.page_id ))
            {
                string url = "https://" + Request.Url.Authority + "/" + node.controller + "/" + node.action;
                <li><a href="@url">@node.page_name</a></li>
            }
            else
            {
                //找到所有孩子產生一個dropdown以自身為觸發點
                var childrenList = nodeList.Where(x => x.parent_id == node.page_id ).ToList();
                <li class="dropdown"><a class="dropdown-toggle" data-toggle="dropdown" href="#">@node.page_name <b class="caret"></b></a>@DropDownMenu(childrenList, nodeList)</li>
            }
        }
    </ul>
}
@helper DropDownMenu(List<GroupLoginDtl> childrenList, List<GroupLoginDtl> nodeList)
{
    <ul class="dropdown-menu">
        @foreach (var node in childrenList)
        {
            //孩子若沒有孩子就是連結
            string url = "https://" + Request.Url.Authority+"/"+node.controller + "/" + node.action;
            if (!nodeList.Any(x => x.parent_id == node.page_id ))
            {
                <li><a href="@url">@node.page_name</a></li>
            }
            else
            {
                //找到孩子的孩子們以自身為觸發點產生submenu並遞迴產生ul dropdown-menu
                var childrenList2 = nodeList.Where(x => x.parent_id == node.page_id ).ToList();
                <li class="dropdown-submenu"><a href="#">@node.page_name</a>@DropDownMenu(childrenList2, nodeList)</li>
            }
        }
    </ul>
}
```
因此篇為引入`bootstrap3` 多層選單子節點的部分需要做`css`的調整
```html
 <style>
    /*多層下拉選單設定*/
    .dropdown-submenu {
        position: relative;
    }
    .dropdown-submenu > .dropdown-menu {
        top: 0;
        left: 100%;
        margin-top: -6px;
        margin-left: -1px;
        -webkit-border-radius: 0 6px 6px 6px;
        -moz-border-radius: 0 6px 6px 6px;
        border-radius: 0 6px 6px 6px;
    }
    .dropdown-submenu:hover > .dropdown-menu {
        display: block;
    }
    .dropdown-submenu > a:after {
        display: block;
        content: " ";
        float: right;
        width: 0;
        height: 0;
        border-color: transparent;
        border-style: solid;
        border-width: 5px 0 5px 5px;
        border-left-color: #cccccc;
        margin-top: 5px;
        margin-right: -10px;
    }
    .dropdown-submenu:hover > a:after {
        border-left-color: #ffffff;
    }
    .dropdown-submenu.pull-left {
        float: none;
    }
</style>
```
### 權限群組

做好頁面管理後，可以建立群組資料表來將每個群組賦予不同的權限  
每個群組的明細為所有頁面，調整此頁是否可進入/可出現在Menu  
做好群組後，只需要調整`Session`為該登入者群組明細  
並在`Partial View`增加判斷是否需產生此節點。
```C#
public class GroupM
{
    [Display(Name = "群組編號")]
    public string group_id { get; set; }
    [Display(Name = "群組名稱")]
    public string group_name { get; set; }
}
public class GroupD
{
    [Display(Name = "可看")]
    public bool can_see { get; set; }
    [Display(Name = "可進入")]
    public bool can_enter { get; set; }  
    [Display(Name = "頁號")]
    public string page_id { get; set; }
    [Display(Name = "頁面名稱")]
    public string page_name { get; set; }
    [Display(Name = "Controller")]
    public string controller { get; set; }
    [Display(Name = "Action")]
    public string action { get; set; }
    [Display(Name = "父頁面")]
    public string parent_id { get; set; }   
}
```
### 訪問權限

雖然在`Menu`上是看不到不被允許的頁面節點  
但仍可以透過網址來訪問該頁  
所以當做好群組權限設定後，在進`Action`之前就檔掉不被允許的訪問  
利用[此篇]("https://hushuuu.github.io/2021/03/15/mvc-%E7%99%BB%E5%85%A5%E9%A9%97%E8%AD%89/")介紹的`Filter`來達成
```C#
 public override void OnActionExecuting(ActionExecutingContext filterContext)
{
    string controller = filterContext.ActionDescriptor.ControllerDescriptor.ControllerName;
    string action = filterContext.ActionDescriptor.ActionName;
    List<PageModel> groupList = HttpContext.Current.Session["PageModel"] as List<PageModel>;
    if(!groupList.Any(z=>z.can_enter==true && z.controller==controller && z.action == action))
    {
        filterContext.Result = new RedirectToRouteResult(new RouteValueDictionary(new
        {
            controller = "Home",
            action = "Index"
        }));
    }
}
```

## 小結

頁面權限的管控有很多種方式，也可以透過不同權限載入不同`Layout`的方式  
`子選單`顯示的部分，`bootstrap`較新版本或是其他方式應也可達成需求  
總結在有父子關聯的處理上，使用遞迴的概念是非常實用的。

## 參考連結

>* [url1](https://blog.sig.tw/2014/04/bootstrap-add-submenu.html)