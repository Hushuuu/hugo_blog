---
title: "EntityFramework查詢結果回傳DataTable"
description: 利用DbCommand的方式來查詢並回傳DataTable
date: 2021-03-15T09:55:01+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - EntityFramework
    - Sql
tags: [
    "EntityFramework",
    "Sql",
]
---

## 前言

以DatabaseFirst來說不管是查詢還是更改的動作都是和資料庫綁定的實體資料模型Model進行交流  
查詢的結果會是模型的類別，如果結果要為DataTable，EF也可以使用Command的方式。

## 主要內容

查詢結果回傳DataSet方法  
傳入`sql CommandText` 以及數組`KeyValuePair`參數
```C#
public DataSet EF_SQL_DS(string sql,Dictionary<string,object> parameters)
{
    XXXXEntities Db = new XXXXEntities();//更改為自己的Entities
    DataSet ds = new DataSet();
    DbCommand cmd = Db.Database.Connection.CreateCommand();
    cmd.CommandText = sql;
    foreach(KeyValuePair<string,object> p in parameters)
    {
        DbParameter dbp = cmd.CreateParameter();
        dbp.ParameterName = p.Key;
        if (p.Value != null)
        {
            dbp.Value = p.Value;
        }
        else
        {
            dbp.Value = DBNull.Value;
        }
        cmd.Parameters.Add(dbp);
    }
    Db.Database.Connection.Open();
    var reader = cmd.ExecuteReader();
    var tb = new DataTable();
    tb.Load(reader);
    ds.Tables.Add(tb);
    reader.Close();
    Db.Database.Connection.Close();
    return ds;
}
```

## 小結

利用`LinQ Select`出自訂類別應較為方便泛用。  
若利用EF的Command須注意是否會和Entities的方式混用導致非預期的資料不正確或錯誤。
