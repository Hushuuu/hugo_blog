---
title: "Vue Component"
description:
date: 2022-02-25T14:25:43+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - VueJs
tags: [
    "VueJs",
    "javascript",
]
---

## 前言

使用Vuejs 2練習模板`Component`  
假設要設計`bootstrap4`彈窗的模板  
`prop`命名最好不要有大寫否則在`html`模板需要處理成`-`    
`ex:foodSize=>food-size`  

### 設計

`設計template html`
```html
<template id="vue_modalTemplate">
    <div class="modal fade" tabindex="-1" role="dialog" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">       
                    <slot name="header">
                        <h5 class="modal-title">{{msg_title}}</h5>
                    </slot>
                </div>
                <div class="modal-body">
                    <slot name="body">
                        {{msg_body}}
                    </slot>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">關閉</button>
                    <button v-if="submit_func" class="btn btn-sm btn-success" type="button" v-on:click="submit_func">{{submit_text}}</button>
                </div>
            </div>
        </div>
    </div>
</template>
```

```javascript
//註冊Component要在實例vue物件之前
Vue.component('vue_modal', { //vue component name
        template: '#vue_modalTemplate', //vue template selector
        props: {
            "submit_func": Function, //提交方法
            "submit_text": String, //
            "msg_title": { type: String, default: function () { return "訊息"; } },
            "msg_body": {  type: String, default: function () { return "訊息主體"} },
        }
    });
    const vm = new Vue({
        data() {
            return {
                common_msg_title: "",
                common_msg_body:"",
            }
        },
        methods: {
            showMsg(title,msg) {
                this.common_msg_title = title;
                this.common_msg_body = msg;
                $("#common_msg_modal").modal('show');
            }
        }
    }).$mount('#vueDiv');//掛載
```

`實際在Html使用`  
```html
<vue_modal id="common_msg_modal" :msg_title="common_msg_title" :msg_body="common_msg_body">
    </vue_modal>
```
`只要在js呼叫vue實例中的showMsg方法就能叫出訊息視窗`
```javascript
vm.showMsg('標題','訊息內容');
```

### 傳遞進子元件

若將`input tag`也包成子元件(尚未修正完成) 
```javascript
//Component
Vue.component('inputcomponent', {
        template: '#inputTemplate',
        props: {
            "fieldobj": Array, //定義輸入欄位的名稱類型甚至對應下拉來源
            "childobj": {  //子元件綁定
                type: Object,
                default: function () {
                    return {}
                }
            }
        },
        watch: {
            childobj: {  //監聽這屬性有異動
                deep: true, //若是監聽物件這種call by ref的要設定深層
                handler(val) { //(newValue,oldValue)
                    this.$emit('update', val); //從下要呼叫上的事件用emit
                    //此處呼叫inputcomponent綁定的update事件
                }
            }
        }
    });
//Vue 實例
const vm = new Vue({
        data() {
            return {
                fieldObj: [ //假設三個input欄位屬性有一個下拉式 來源也定義好
                    { fieldname: "contact_addr", fieldtype: "text",  },
                    { fieldname: "contact_tel", fieldtype: "number", },
                    {
                        fieldname: "take_kind", fieldtype: "select",
                        optionlist: [{ "Text": "0", "Value": 0 },
                            { "Text": "1", "Value": 1 }
                        ]
                    }
                ],
                childobj: { }, //
                objFromChild: {}, //從子元件傳過來的
                common_msg_title: "",
                common_msg_body:"",
            }
        },
        methods: {
            submitEdit: function () {
                console.log("");
            },
            updateInfo(val) {
                //this.$set(this.objFromChild, idx, val);
                this.objFromChild = val;
            },
        }
    }).$mount('#vueDiv');
``` 
```html
<template id="inputTemplate">
    <div>
        <div v-for="obj in fieldobj" class="input-group mb-3">
            <div class="input-group-prepend">
                <span class="input-group-text">{{obj.fieldname}}</span>
            </div>
            <input v-if="obj.fieldtype!='select'" v-bind:type="obj.fieldtype" class="form-control" :name="obj.fieldname" v-model="childobj[obj.fieldname]">
            <select v-if="obj.fieldtype=='select'" class="form-control" v-model="childobj[obj.fieldname]">
                <option v-for="item in obj.optionlist" v-bind:value="item.Value">
                    {{item.Text}}
                </option>
            </select>
        </div>
    </div>
</template>

 <vue_modal id="log-2"
                 :submit_func="submitEdit"
                 :submit_text="'提交'"
                 :msg_title="'提交資料'">
        <span slot="header">
        </span>
        <div slot="body">
            <inputcomponent :fieldobj="fieldObj" 
                            v-bind="childobj"
                            @@update="updateInfo">
            </inputcomponent>
        </div>
    </vue_modal>

```
`<component v-bind="childobj"></component>`可綁定並解構
`vm的fieldObj被綁進子元件的fieldobj prop`



## 小結

`slot 若在呼叫component沒有特別寫則會帶預設的` 
`slot 寫的變數都為父的變數，子元件不牽涉進slot區塊`
`注意子元件和父元件最好不要雙向綁定，多個子元件改到同一個父實例物件可能會錯亂`  
`利用prop傳入元件 emit事件傳出來個別處理`  

## 參考連結

>* [url1](https://book.vue.tw/CH2/2-4-slots.html)
>* [url2](https://book.vue.tw/CH2/2-2-communications.html)