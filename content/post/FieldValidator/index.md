---
title: "asp.net MVC 後端Model欄位驗證"
description: "說明幾種Model Binding驗證欄位的方法"
date: 2021-03-15T09:03:12+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
tags: [
    "Model",
    "Validator",
]
---

## 前言

說明幾種`Model Binding`驗證欄位的方法。   
包含正則式，長度限制，必填，自訂驗證。  

## 主要內容

以下面例子 `account` 的欄位的驗證  
`StringLength()` 長度限制  
`RegularExpression()` 利用正則式  
`Required` 必要欄位  
`CheckAccount` 則為自訂的驗證

```C#
[Display(Name = "帳號")]
[StringLength(20)]
[RegularExpression(@"[a-zA-Z0-9]*$", ErrorMessage = "帳號僅能有英文或數字")]
[Required]
[CheckAccount(ErrorMessage = "重複")]
public string account { get; set; }
```

自訂驗證部分可以建立一個類別繼承`ValidationAttribute`，再複寫 `IsValid方法
```C#
public class CheckAccountAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext validationContext)
    {
        //return base.IsValid(value, validationContext);
        if (value != null)
        {            
            if (/*條件式*/)
            {
                return new ValidationResult(FormatErrorMessage(validationContext.DisplayName));
            }
            else
            {
                return ValidationResult.Success;
            }
        }
        return null;
    }
}
```

## 驗證結果

在後端接收到資料後可使用`Model.State`來判斷Model欄位驗證是否通過。
```C#
 if(ModelState.IsValid){
    //驗證通過
 }
```