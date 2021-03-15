---
title: "無法轉換類型為CrystalDecisions.ReportAppServer.Controllers.ReportSourceClass"
description: "Unable to cast COM object of type 'CrystalDecisions.ReportAppServer.Controllers.ReportSourceClass' to interface type 'CrystalDecisions.ReportAppServer.Controllers.ISCRReportSource'. This operation failed because the QueryInterface call on the COM component for the interface with IID '{98CDE168-C1BF-4179-BE4C-F2CFA7CB8398}' failed"
date: 2021-03-15T10:58:27+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - CrystalReport
tags: [
    "Issue",
    "CrystalReport",
]
---

## 前言

較舊之Winfrom專案，有些報表是用SAP的CrystalReport來產生報表。  
若使用VS2019的環境必須先至SAP下載安裝相關之Runtime或是專門給開發人員的CRforVisualStudio  
安裝完畢後可能還會遇到套件版本不相容之問題需另解決。

## 主要內容

若是在產生報表時出現錯誤訊息的話，試試在`Webconfig/Appconfig` 下增加此段  
並調整 `oldVersion` 和 `newVersion` 之版本號多嘗試幾次  
```markdown
<runtime>
  <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.CrystalReports.Engine" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportSource" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>    
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.Shared" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.Web" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.Windows.Forms" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.ClientDoc" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.CommonControls" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.CommonObjectModel" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.Controllers" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.CubeDefModel" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.DataDefModel" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.DataSetConversion" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>    
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.ObjectFactory" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.Prompting" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.ReportDefModel" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
    <dependentAssembly>
      <assemblyIdentity name="CrystalDecisions.ReportAppServer.XmlSerialize" publicKeyToken="692fbea5521e1304" culture="neutral"/>
      <bindingRedirect oldVersion="13.0.2000.0" newVersion="13.0.3500.0"/>
    </dependentAssembly>
  </assemblyBinding>  
</runtime>
```

## 小結

組件版本號若沒有對上，在建置前可能就會報錯  
若建置有成功，也可能到產生報表前才報錯。  
這問題一步步去試，多嘗試幾次調整版本或安裝

## 參考連結

>* [SAP](https://wiki.scn.sap.com/wiki/display/BOBJ/Crystal+Reports%2C+Developer+for+Visual+Studio+Downloads)