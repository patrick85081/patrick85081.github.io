---
title: "SQL 進行 字串串接"
date: 2022-07-15T10:10:34+08:00
draft: false
tags: 
  - SQL
  - SQL Server
description: "在 C# 裡面要做字串串接 string.Join() 非常容易，但是如果要在SQL裡面進行呢？這裡花了一些時間研究，主要使用XML的方式來做。
"
---

在 C# 裡面要做字串串接 `string.Join()` 非常容易，但是如果要在SQL裡面進行呢？這裡花了一些時間研究，主要使用XML的方式來做。

# Case 1: 可以支援到 Sql 2014
* 原始資料

| MAC          | SiteId | NicVendorId | HostName |
| ------------ | ------ | ----------- | -------- |
| 001122334455 | 0      | 3           | Win7     | 
| 001122334455 | 1      | 4           | Win10    | 

``` sql
SELECT 
	DISTINCT M.Mac,
	STUFF( (SELECT ',' + CONVERT(varchar, inner.SiteId) FROM MacAddresses inner WHERE inner.Mac = M.Mac FOR XML PATH('') ), 1, 1, '') as 'SiteId',
	STUFF( (SELECT ',' + CONVERT(varchar, inner.NicVendorId) FROM MacAddresses inner WHERE inner.Mac = M.Mac FOR XML PATH('') ), 1, 1, '') as 'NicVendorId'
	SUBSTRING( (SELECT ',' + inner.HostName FROM MacAddress inner WHERE inner.Mac = M.Mac FOR XML PATH('') ), 2, 999 )
FROM MacAddresses M
```
* 結果

| MAC          | SiteId | NicVendorId | HostName   |
| ------------ | ------ | ----------- | ---------- |
| 001122334455 | 0,1    | 3,4         | Win7,Win10 | 

## STUFF 字串插入 
> STUFF(原始字串, 從第幾個字元插入, 刪除幾個原始字串, 要插入的字串)
  
``` sql
SELECT STUFF( 'abcdefg', 3, 2, '12345')
-- ab12345efg
```

## SUBSTRING 字串截取 
> SUBSTRING(原始字串, 從第幾個字, 截取長度)

``` sql
SELECT SUBSTRING('abcdefg', 2, 2)
-- cd
```

## FOR XML PATH 將表格轉成 XML
原始資料
| MAC          |
| ------------ |
| 001122334455 |
| 112233445566 |

``` sql
SELECT MAC
FROM MacAddress
FOR XML PATH('')
-- <MAC>001122334455</MAC><MAC>112233445566</MAC>

SELECT MAC
FROM MacAddress
FOR XML PATH('Nic')
-- <Nic><MAC>001122334455</MAC><MAC>112233445566</MAC></Nic>

SELECT ',' + MAC
FROM MacAddress
FOR XML PATH('')
-- ,001122334455,112233445566

SELECT ',' + MAC
FROM MacAddress
FOR XML PATH('Nic')
-- <Nic>,001122334455,112233445566</Nic>
```

# Case 2: 可以支援到 Sql 2017
* 原始資料

| SwitchID | MACs              | InterfaceIndexs |
| -------- | ----------------- | --------------- |
| 258      | 00-1F-26-BD-00-80 | 0               |
| 258      | 00-1F-26-BD-00-C0 | 1               |
| 259      | 50-06-04-42-64-00 | 0               |
| 259      | 50-06-04-42-64-40 | 1               |

``` sql
SELECT
	SwitchID,
	STRING_AGG(MAC, ',') as MACs,
	STRING_AGG(InterfaceIndex, ',') as InterfaceIndexs
FROM
	DeviceMacs
GROUP BY
	SwitchID
```

* 結果

| SwitchID | MACs                             | InterfaceIndexs |
| -------- | ----------------------------------- | ------------ |
| 258      | 00-1F-26-BD-00-80,00-1F-26-BD-00-C0 | 0,1          |
| 259      | 50-06-04-42-64-00,50-06-04-42-64-40 | 0,1          |