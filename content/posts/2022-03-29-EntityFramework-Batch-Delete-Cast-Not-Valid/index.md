---
title: "SQL 批次刪除，資料轉型失敗問題"
date: 2022-03-30T13:12:45+08:00
draft: false
tags:
  - SQL
  - EntityFramework
description: "最近碰到一個有去的錯誤，當透過EntityFrameworkPlus 做資料庫資料批次刪除，結果卻回傳InvalidCastException，但是卻只有這個DB才有此問題，這問題真叫人苦惱。
"
---
# 原由
最近碰到一個有去的錯誤，當透過`EntityFrameworkPlus` 做資料庫資料批次刪除，結果卻回傳`InvalidCastException`，但是卻只有這個DB才有此問題，這問題真叫人苦惱。

原程式很單純，大致上如下：
``` cs
dataContext.Employees
  .Where(e => e.Year > 30)
  .Delete();
```

![](InvalidCastException.png)

# 結果
結果發現因之前有資料庫除錯上需求，在該資料表加入 `Delete Trigger`，應該要 Insert 刪除記錄到另一張表，卻寫成資料 Select。

陰錯陽差之下在 `Entity Framework Plus` 下達刪除命令後，會去抓資料庫回傳的影響資料筆數，卻抓到 Trigger 產生出來的資料，造成使用**該資料**的第一個Row第一個Column資料，轉型成 **影響筆數** （Int32） 才出的錯誤，結案。
