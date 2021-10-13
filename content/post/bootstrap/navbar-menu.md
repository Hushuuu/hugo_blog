---
title: "Bootstrap Navbar Menu"
description: "導覽列選單含子選單使用"
date: 2021-10-13T09:32:14+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Bootstrap
tags: [
    "Bootstrap",
    "CSS",
]
---

## v3版本

增加一個`li`後`a`tag觸發子選單`ul` `submenu`
```html
<div class="navbar-collapse collapse">
<ul class="nav navbar-nav">
    <li class="dropdown">
        <a href="#" data-toggle="dropdown">選單2<b class="caret"></b></a>
        <ul class="dropdown-menu">
            <li class="dropdown-submenu">
                <a href="#" data-toggle="dropdown">次選單2</a>
                <ul class="dropdown-menu">
                    <li class="dropdown-submenu">
                        <a href="#" data-toggle="dropdown">次次選單3</a>
                        <ul class="dropdown-menu">
                            <li><a href="#">次次次選單1</a></li>
                        </ul>
                    </li>
                </ul>
            </li>
        </ul>
    </li>
</ul>
</div>
```
```CSS
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
```

## v4版本

```html
<nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
    <div class="container">
        <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
            <ul class="navbar-nav flex-grow-1">
                <li class="dropdown">
                    <a class=" dropdown-toggle nav-link text-dark" href="#">menu</a>
                    <ul class="dropdown-menu">
                        <li><a class="dropdown-item" href="#">Submenu</a></li>
                        <li><a class="dropdown-item" href="#">Submenu0</a></li>
                        <li class="dropdown-submenu">
                            <a class="dropdown-item dropdown-toggle" href="#">Submenu 1</a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="#">Subsubmenu1</a></li>
                                <li><a class="dropdown-item" href="#">Subsubmenu1</a></li>
                            </ul>
                        </li>
                        <li class="dropdown-submenu">
                            <a class="dropdown-item dropdown-toggle" href="#">Submenu 2</a>
                            <ul class="dropdown-menu">
                                <li><a class="dropdown-item" href="#">Subsubmenu2</a></li>
                                <li><a class="dropdown-item" href="#">Subsubmenu2</a></li>
                            </ul>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
    </div>
</nav>
```
```CSS
.navbar-nav li:hover > ul.dropdown-menu {
    display: block;
}
.dropdown-submenu {
    position: relative;
}
.dropdown-submenu > .dropdown-menu {
    top: 0;
    left: -97%; /*從左邊展出*/
    margin-top: -6px;
}
/* rotate caret on hover */
.dropdown-menu > li > a:hover:after {
    text-decoration: underline;
    transform: rotate(-90deg);
}
.dropdown-submenu > a {
    color: #212529;
    text-decoration: none;
}
```
