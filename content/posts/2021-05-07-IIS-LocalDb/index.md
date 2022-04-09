---
title: "IIS 使用 LocalDB 問題"
date: 2021-05-07T23:28:13+08:00
draft: false
tags: 
  - SQL Server
  - LocalDB
description: "開發時候使用LocalDB使用得很順手，因此將網站放到本機的IIS時，也會想要沿用開發時候使用的LocalDb，原本以為是很單純的一件事，沒想到後面有這麼多的問題。"
---
開發時候使用LocalDB使用得很順手，因此將網站放到本機的IIS時，也會想要沿用開發時候使用的LocalDb，原本以為是很單純的一件事，沒想到後面有這麼多的問題。

# 當IIS下 直接使用 LocalDB，出現 Local DB Runtime Error
出現以下錯誤

> A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: SQL Network Interfaces, error: 50 - Local Database Runtime error occurred. Cannot create an automatic instance. See the Windows Application event log for error details.


* 解決方法  
修改 `C:\Windows\System32\inetsrv\config\applicationHost.config` 檔案
將 `loadUserProfile` 和 `setProfileEnvironment` 都改成 `true`
``` xml
<applicationPoolDefaults>
	<!-- 省略 -->
	<processModel 
		identityType="ApplicationPoolIdentity" 
		loadUserProfile="true" 
		setProfileEnvironment="true"/>
	<!-- 省略 -->
</applicationPoolDefaults>
```

# IIS 上的網站可以正常執行，但是連的是另一個資料庫
因為 IIS 是使用另一個使用者執行，但是localDB資料庫有分使用者

* 解法
建立共享的 LocalDB 個體
``` cmd
sqllocaldb i // 查看目前有哪些 localdb 個體

// 請參照以下流程
sqllocaldb share MSSQLLocalDB MSSQLLocalDBShared
sqllocaldb stop MSSQLLocalDBShared
sqllocaldb start MSSQLLocalDBShared
```

修改連線字串為 `(localdb)\.\MSSQLLocalDBShared`就可以連線

# IIS 沒有權限使用 LocalDB 共享的個體
``` sql
CREATE LOGIN [IIS APPPOOL\<Your User>] FROM WINDOWS;
EXEC sp_addsrvrolemember N'IIS APPPOOL\<Your User>', sysadmin;
```


