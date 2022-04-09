---
title: "動態切換 EF6 設定檔中的SQL Provider"
date: 2022-01-19T22:02:49+08:00
draft: false
tags: 
  - EntityFramework
description: "我們都知道Entity Framework 6的設定是儲存在 app.config/Web.config，但是如果我想要在程式中動態切換他的SQL Provider時候應該怎麼做呢?"
---

我們都知道Entity Framework 6的設定是儲存在 `app.config/Web.config`，但是如果我想要在程式中動態切換他的SQL Provider時候應該怎麼做呢?


# Use Config by file
正常的 `app.config / web.config` 都有以下設定，記錄著有哪些 SQL Provider，還有連線字串的資訊。
``` xml
<!-- 必備橋段 -->
<entityFramework>
    <providers>
      <provider invariantName="System.Data.SqlClient" type="System.Data.Entity.SqlServer.SqlProviderServices, EntityFramework.SqlServer" />
    
      <!-- SQLite Provider -->
      <provider invariantName="System.Data.SQLite.EF6" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
      <provider invariantName="System.Data.SQLite" type="System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6" />
    </providers>
</entityFramework>

<!-- 選配橋段 -->
<connectionStrings>
    <add name="DefaultConnection" connectionString="data source=lab.db" providerName="System.Data.SQLite" />
</connectionStrings>
```

# Use DbConfiguration by code
實作 `DbConfiguration`，等同於 xml entityFramework/providers/provider 設定
有三個步驟
1. 註冊 `EF Provider Factory`
2. 註冊 `EF Provider Service`
3. 註冊 `Db Provider Factory`
``` cs
public class SqliteConfig : DbConfiguration
{
    public SqliteConfig()
    {
        // Register EF Provider Factory
        SetProviderFactory("System.Data.SQLite", System.Data.SQLite.EF6.SQLiteProviderFactory.Instance);
        // SetProviderFactory("System.Data.SQLite.EF6", System.Data.SQLite.EF6.SQLiteProviderFactory.Instance);

        var type = Type.GetType("System.Data.SQLite.EF6.SQLiteProviderServices, System.Data.SQLite.EF6");
        var field = type.GetField("Instance", BindingFlags.NonPublic | BindingFlags.Static);
        var dbProviderServices = field.GetValue(null) as DbProviderServices;
        
        // Register EF Provider Service
        SetProviderServices("System.Data.SQLite", dbProviderServices);
        // SetProviderServices("System.Data.SQLite.EF6", dbProviderServices);

        // Register DbProviderFactory
        DbProviderFactories.RegisterFactory("System.Data.SQLite", System.Data.SQLite.SQLiteFactory.Instance);
    }
}
```

## 方法一：使用 `DbConfigurationTypeAttribute` 套用
使用 Attribute 指定設定檔物件，不過這樣就缺乏彈性，不是我要的。
``` cs
[DbConfigurationType(typeof(SqliteConifg))]
public DataContext : DbContext
{
    // skip...
}
```

## 方法二：使用 `DbConfiguration.SetConfiguration`
使用 靜態方法設定設定檔物件，這樣就可以達到動態切換 Provider的需求了。
``` cs
DbConfiguration.SetConfiguration(new SqliteConfig())
```


# 參考資料
[SQLite Code First 和 Migration (2)](https://dotblogs.com.tw/yc421206/2020/02/14/sqlite_code_first_migration_2)
[Code-based Configuration in Entity Framework](https://www.entityframeworktutorial.net/entityframework6/code-based-configuration.aspx)