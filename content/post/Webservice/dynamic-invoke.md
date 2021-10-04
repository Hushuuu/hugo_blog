---
title: "動態調用Webservice"
description:
date: 2021-10-04T14:38:32+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Webservice
tags: [
    "C#",
    "Webservice",
]
---

## 主要內容

建立`DynamicInvokeWebservice.cs`
```C#
 public class DynamicInvokeWebservice
    {

        public DynamicInvokeWebservice()
        {
            //
            // TODO: 在此加入建構函式的程式碼
            //
        }

        static SortedList<string, Type> _typeList = new SortedList<string, Type>();


        #region InvokeWebService


        static string GetCacheKey(string url, string className)
        {
            return url.ToLower() + className;
        }
        static Type GetTypeFromCache(string url, string className)
        {
            string key = GetCacheKey(url, className);
            foreach (KeyValuePair<string, Type> pair in _typeList)
            {
                if (key == pair.Key)
                {
                    return pair.Value;
                }
            }

            return null;
        }
        static Type GetTypeFromWebService(string url, string className)
        {
            string @namespace = "EnterpriseServerBase.WebService.DynamicWebCalling";
            if ((className == null) || (className == ""))
            {
                className = GetWsClassName(url);
            }

            //获取WSDL
            WebClient wc = new WebClient();
            Stream stream = wc.OpenRead(url + "?WSDL");
            ServiceDescription sd = ServiceDescription.Read(stream);
            ServiceDescriptionImporter sdi = new ServiceDescriptionImporter();
            sdi.AddServiceDescription(sd, "", "");
            CodeNamespace cn = new CodeNamespace(@namespace);

            //生成客户端代理类代码
            CodeCompileUnit ccu = new CodeCompileUnit();
            ccu.Namespaces.Add(cn);
            sdi.Import(cn, ccu);
            //CSharpCodeProvider csc = new CSharpCodeProvider();
            //ICodeCompiler icc = 因; -> 已過時
            CodeDomProvider provider = CodeDomProvider.CreateProvider("CSharp");

            //设定编译参数
            CompilerParameters cplist = new CompilerParameters();
            cplist.GenerateExecutable = false;
            cplist.GenerateInMemory = true;
            cplist.ReferencedAssemblies.Add("System.dll");
            cplist.ReferencedAssemblies.Add("System.XML.dll");
            cplist.ReferencedAssemblies.Add("System.Web.Services.dll");
            cplist.ReferencedAssemblies.Add("System.Data.dll");

            //编译代理类
            //CompilerResults cr = icc.CompileAssemblyFromDom(cplist, ccu); -> 因為csc.CreateCompiler()已過時
            CompilerResults cr = provider.CompileAssemblyFromDom(cplist, ccu);
            if (true == cr.Errors.HasErrors)
            {
                System.Text.StringBuilder sb = new System.Text.StringBuilder();
                foreach (System.CodeDom.Compiler.CompilerError ce in cr.Errors)
                {
                    sb.Append(ce.ToString());
                    sb.Append(System.Environment.NewLine);
                }
                throw new Exception(sb.ToString());
            }

            //生成代理实例，并调用方法
            System.Reflection.Assembly assembly = cr.CompiledAssembly;
            Type t = assembly.GetType(@namespace + "." + className, true, true);

            return t;
        }

        /// <summary>
        /// 动态调用web服务
        /// timeout: 5分鐘
        /// </summary>
        /// <param name="url"></param>
        /// <param name="methodName"></param>
        /// <param name="args"></param>
        /// <returns></returns>
        public static object InvokeWebService(string url, string methodName, object[] args)
        {
            return InvokeWebService(url, null, methodName, args, 300000);
        }

        /// <summary>
        /// 动态调用web服务
        /// </summary>
        /// <param name="url"></param>
        /// <param name="methodName"></param>
        /// <param name="args"></param>
        /// <param name="timeout"></param>
        /// <returns></returns>
        public static object InvokeWebService(string url, string methodName, object[] args, int? timeout)
        {
            return InvokeWebService(url, null, methodName, args, timeout);
        }

        public static object InvokeWebService(string url, string className, string methodName, object[] args, int? timeout)
        {
            try
            {
                Type t = GetTypeFromCache(url, className);
                if (t == null)
                {
                    t = GetTypeFromWebService(url, className);


                    //添加到缓冲中
                    string key = GetCacheKey(url, className);
                    _typeList.Add(key, t);
                }


                object obj = Activator.CreateInstance(t);
                MethodInfo mi = t.GetMethod(methodName);

                if (mi == null)
                {
                    throw new Exception("沒有WS:" + methodName);
                }

                if (timeout.HasValue)
                {
                    //增加以下設定timeout時間
                    PropertyInfo propInfo = obj.GetType().GetProperty("Timeout");
                    propInfo.SetValue(obj, timeout, null);
                }

                return mi.Invoke(obj, args);
            }
            catch (Exception ex)
            {
                //throw new Exception(ex.InnerException.Message, new Exception(ex.InnerException.StackTrace));
                throw ex;
            }
        }

        private static string GetWsClassName(string wsUrl)
        {
            string[] parts = wsUrl.Split('/');
            string[] pps = parts[parts.Length - 1].Split('.');

            return pps[0];
        }
        #endregion
    }
```

### 使用

基本上直接調用`InvokeWebService`方法即可
```C#
var url = "Webservice的網址";
var methodName = "HellWorld";

//參數照定義的method順序傳入
List<string> args = new List<string>();
args.Add("123456");
args.Add("asdfg");
var obj = DynamicInvokeWebservice.InvokeWebService(url, methodName, args.ToArray());
```
