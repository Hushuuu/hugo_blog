---
title: "Html之命名規則使用"
description:
date: 2023-08-17T10:38:45+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Html
tags: [
    "Html",
]
---

## 前言

HTML裡在變數或屬性的命名因人而異有不同習慣  
如駝峰式`camelCase`=>`userId`  
如串燒式`kebab-case`=>`user-id`  

但要注意以下兩點  
1. html大小寫不敏感所以一般使用`kebab-case`命名，然後在javascript取屬性值會轉為`camelCase`  
2. 在`vue-template`也可以使用`cameCcase`但不建議  

```html
<div id="a" data-user-id=123/>
<script>
    console.log(document.querySelector('#a').dataset.userId); //123
</script>
```

## 延伸-Vue

屬性Prop  
```html
<template :user-id="'123'"/>
<script setup>
   const props =  defineProps({
      userId: String,
   })     
   console.log(props.userId); // 123
</script>
```
模板Template  
可用首字母大寫`PascalCase`或`kebab-case`引入  
`PascalCase`定義組件，在template使用兩種寫法都可以  
但`kebab-case`定義則template只可使用`kebab-case` 
```javascript
import MenuComponent from '@/components/MenuComponent.vue'
 components: {
    'menu-component': MenuComponent,
    MenuComponent
  },
<template>
  <MenuComponent /> <!--only PascalCase -->
  <menu-component /> <!--PascalCase or kebab-case -->
</template>

```



