---
title: "Vue Component(datepicker)"
description:
date: 2022-02-26T14:25:43+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - VueJs
tags: [
    "VueJs",
    "datepicker"
]
---

## 前言

使用Vuejs 2練習模板`Component`  
並處理`jquery datepicker`無法雙向綁定問題  

### 設計

`偵測datepicker的onSelect事件來觸發元素的input監聽事件`
```javascript
//註冊Component要在實例vue物件之前
 Vue.component('vue_date', {
            template: '<input type="text" v-datepicker class="datepicker" :class="inputclass" :value="value" v-on:input="update($event.target.value)">',
            directives: {
                datepicker: {
                    inserted(el, binding, vNode) {
                        $(el).datepicker({
                            autoclose: true,
                            format: 'yyyy-mm-dd',
                            onSelect: function (e) {
                                //pick觸發
                                vNode.context.$emit('input', e);
                            }
                        });
                    }
                }
            },
            props: {
                'value': String,
                "inputclass": {
                    type: Object, default: function () { return { "form-control": true } }
                },
                "id": String,
            },
            methods: {
                update(v) {
                    //input觸發
                    this.$emit('input', v);
                }
            }
        })
        //vue instant
    const vm = new Vue({
        data() {
            return {
                mydate: ""
            }
        }
    }).$mount('#vueDiv');//掛載
```

`實際在Html使用`  
```html
<vue_date v-model="mydate"></vue_date>
``` 

## 參考連結

>* [url1](https://stackoverflow.com/questions/49515965/v-model-not-working-with-bootstrap-datepicker)