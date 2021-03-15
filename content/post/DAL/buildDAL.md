---
title: "MVC-資料存取(DataAccess)"
description: "從資料庫連線到查詢及執行，將行為寫成方法，將方法封裝成類別"
date: 2021-03-15T11:32:53+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - MVC
    - SQL
tags: [
    "SQL"
]
---

## 前言

在開發時與資料庫間的連線橋梁，較常見的為ADO.NET或是EF的方式，此篇只介紹ADO.NET   
使用ADO.NET最基本就是開啟SqlConnection連線建立SqlCommand操作  

## 第一步

直接連線並查詢出結果的範例
```C#
string connStr = "連線字串";
string sql = "SELECT * FROM Product Where id=@id";
SqlConnection Conn = new SqlConnection(connStr);
Conn.Open();
SqlCommand Cmd = new SqlCommand(sql);
Cmd.Connection = Conn;
Cmd.Parameters.Clear();
Cmd.Parameters.AddWithValue("id","1");
using(SqlDataReader sdr = Cmd.ExecuteReader()){
    while(sdr.Read()){
        Console.WriteLine(sdr.GetString(0));
    }
}
Conn.Close();
```
但若是每個資料庫的操作都需要打上這一大段的Code程式碼不僅變得冗長且難以統一維護  
這時候就可以利用封裝的概念將code包起來  

先建立一個新類別檔`DataAccess.cs`
```C#
 public class DataAccess
    {
       public string ConnectionString {get; set;}
       public SqlConnection Conn {get;set;}
       protected SqlCommand Cmd {get;set;}
       public DataAccess(SqlConnection Conn){
           this.Conn = Conn;
           this.ConnectionString = Conn.ConnectionString;
           InitCmd();
       }
       protected void InitCmd(){
           if(Cmd == null){
               Cmd = new SqlCommand();
               Cmd.Connection = Conn;
           }
       }
       protected void OpenConnIfClosed(){
           if(Conn.State == ConnectionState.Closed){
               Conn.Open();
           }
       }
       protected void CloseConn(){
           Conn.Close();
       }
    }
```

將對資料庫的`Connection`及`Command`初始化行為建立在類別中。  
可以根據`Transaction`需求及`Query`再擴展

```C#
 protected SqlTransaction Transaction { get; set; }  
 private bool UseTransaction { get; set; }
 //SQL查詢
 protected T SqlQuery_result<T>(string strSql, Dictionary<string, object> parameters=null,CommandType cmdType = CommandType.Text
            )
    {
        T tobj = default(T);
        try
        {
            OpenConnIfClosed();
            Cmd.CommandType = cmdType;
            if (UseTransaction && Transaction != null)
            {
                Cmd.Transaction = Transaction;
            }
            Cmd.CommandText = strSql;
            Cmd.Parameters.Clear();
            if (parameters != null && parameters.Count > 0)
            {
                foreach (var item in parameters)
                {
                    if (item.Value == null)
                        Cmd.Parameters.AddWithValue(item.Key, DBNull.Value);
                    else
                        Cmd.Parameters.AddWithValue(item.Key, item.Value);
                }
            }
            using (SqlDataReader odr = Cmd.ExecuteReader())
            {
                if (odr.HasRows)
                {
                    odr.Read();
                    Type y = odr[0].GetType();
                    if (typeof(T) == y)
                    {
                        tobj = (T)Convert.ChangeType(odr[0],typeof(T));
                    }
                    else
                    {
                        tobj = default;
                    }
                }
            }
            Cmd.Parameters.Clear();
            if (!UseTransaction)
            {
                CloseConn();
            }
            return tobj;
        }
        catch (Exception e)
        {
            if (!UseTransaction)
            {
                Dispose();
            }
            throw e;
        }
    }  
```
`SqlQuery_result`有使用泛型`T`，可不事先決定這Function回傳的資料型態。  
但此方法只可以查詢單筆單個欄位，如果想要查詢`Select * from Product`這種結果的話就要再調整。
可以利用`Reflection`反射，來對應泛型的屬性。此篇先不多提。  

除了Query之外還有執行的需求，也一樣擴充我們的類別
```C#
protected int ExecuteSqlCommand(string strSQL, Dictionary<string, object> parameters = null, CommandType cmdType = CommandType.Text
            )
    {
        int effectRows = -1;
        try
        {
            OpenConnIfClosed();
            Cmd.CommandType = cmdType;
            if (UseTransaction && Transaction != null)
            {
                Cmd.Transaction = Transaction;
            }
            Cmd.CommandText = strSQL;
            Cmd.Parameters.Clear();
            if (parameters != null)
            {
                foreach (var item in parameters)
                {
                    Cmd.Parameters.AddWithValue(item.Key, item.Value == null ? DBNull.Value : item.Value);
                }
            }
            effectRows = Cmd.ExecuteNonQuery();
            if (!UseTransaction)
            {
                CloseConn();
            }
        }
        catch (Exception e)
        {
            if (!UseTransaction)
            {
                Dispose();
            }
            throw e;
        }
        return effectRows;
    }
```
## 再簡化

到這裡已經把初始化連線、建立Command、查詢及執行都封裝成一個類別了！  
但方法的存取修飾詞為什麼是`protected`?  
使用`DataAccess`類別時，需要在`Controller`中`new`連線物件並寫增刪修查語法後Call物件的方法。  
如果要更加的簡化`Controller`裡的Code，可以再將增刪修查的動作提取出來，使得`Controller`結構一目了然  

舉例資料表`User`，建立一個類別檔`UserDB.cs`  
繼承`DataAccess`類別來建立增刪修查的方法  
*父類別`protected`的屬性或方法為了限制給繼承`DataAcess`的類別可使用

```C#
public class UserDB : DataAccess
{
    public UserDB(SqlConnection Conn) : base(Conn) { }
    public List<User> ListUser(string Qid = null, string Qname = null, string Qaccount = null)
    {
        Dictionary<string, object> parameters = new Dictionary<string, object>();
        string sql = "select * from [User] where 1=1 ";

        if (Qid != null)
        {
            sql += " and id=@qid ";
            parameters.Add("qid", Qid);
        }
        if (Qname != null)
        {
            sql += " and name=@qname ";
            parameters.Add("qname", Qname);
        }
        if (Qaccount != null)
        {
            sql += " and account=@qaccount ";
            parameters.Add("qaccount", Qaccount);
        }
        List<User> list = SqlQuery<User>(sql,parameters);
        return list;
    }              
    public void CreateUser(string id,string name,string account)
    {
        Dictionary<string, object> parameters = new Dictionary<string, object>();
        string sql = @" insert into [User] (id,name,account) 
                values(@id,@name,@account) ";
        parameters.Add("id", id);
        parameters.Add("name", name);
        parameters.Add("account", account);
        int x = ExecuteSqlCommand(sql, parameters);
    }
}
```
## 實際使用

這樣就把`List`和`Create`的部分也包起來了。  
實際在`Controller`使用只要數行就可以解決  
```C#
string connString = "";
UserDB _UserDB  = new UserDB(connString);
List<User> users = _UserDB.ListUser();
//Create
_UserDB.CreateUser(id,name,account);
```

## 小結

感謝同事大神提供非常大的協助。一開始對物件及封裝的概念非常不清楚，對於資料的存取也只會傻傻地硬拚。  
多次的摸索後，看同樣的東西每一次都能有不太一樣的理解跟體悟。  
若使用EF的方式來存取，可節省掉寫Sql語句的部分可以說是方便很多。  
但EF在使用上會有些看不見的地雷或不方便。得實際遇到不同情況再想辦法解決了。

