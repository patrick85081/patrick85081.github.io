---
title: "好用工具 Wox"
date: 2022-03-20T13:20:58+08:00
draft: false
description: "Windows 快捷列軟體（Wox），她號稱是 Windows 上的 Alfred ，透過一條小小的搜尋列，可以開啟軟體、搜尋檔案、查詢網頁，並透過安裝外掛還能當作計算機、翻譯器、剪貼簿等來使用。"
tags:
  - Tools
categories:
  - Tools
thumbnail: 'https://images.unsplash.com/photo-1609270293211-726165a8c3bb?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=MnwxMTc3M3wwfDF8c2VhcmNofDIwOXx8aHR0cCUzQSUyRiUyRmFwaS53b3gub25lJTJGbWVkaWElMkZwbHVnaW4lMkZDMzQwNkI1QzIyRjA0OTg0QjAxODNEQUU4OTdDQUIzRiUyRmRlbW8tMmQzNzQwNjYtOWRiNS00NzVkLTg4NDktZWIwMDIzZjdlYWQ0LmdpZnxlbnwwfHx8fDE2NDcxNDQ3MDU&ixlib=rb-1.2.1&q=80&w=2000'
---
Windows 快捷列軟體（Wox），她號稱是 Windows 上的 Alfred ，透過一條小小的搜尋列，可以開啟軟體、搜尋檔案、查詢網頁，並透過安裝外掛還能當作計算機、翻譯器、剪貼簿等來使用。

# 基本功能
## Everything
電腦需要安裝 Everything 軟體，功能是可以直接搜尋電腦裡的檔案，直接開啟。

## 網頁搜尋
```
g => google 搜尋
translate => google 翻譯
youtube => youtube 搜尋
maps => google map 搜尋
image => google image 搜尋
wiki => wiki 搜尋
```

# 好用插件
## Dictionary
> wpm install Dictionary

* 使用畫面

![](./Wox_Dictionary.gif)

* 前置作業
先下載[此檔案](https://github.com/harrynull/WoxDictionary/releases/download/dict/ecdict.7z.zip)，放到C:\Users\{用户名}\AppData\Roaming\Wox\Plugins\Dictionary-{随机字符}\dicts里才可以正常使用！
注意：近义词查找需要手动申请Token并在设置页面里填写才能正常使用！

* 使用方式
```
基本查詢
d <Word>

音標
d <Word>!

中文解釋
d <Word>!t

接近的單字
d <Word>!s

其他變形
d <Word>!e
```
---
### 進制轉換
> wpm install 进制转换
 
![](./Wox_Decimal.png)
 
使用进制关键字+“~”开始。
```
// x代表16进制，d代表10进制，o代表8进制，b代表2进制。
x~f //对16进制的 f 进行转换。
b~1001 //对2进制的 1001 进行转换。
```