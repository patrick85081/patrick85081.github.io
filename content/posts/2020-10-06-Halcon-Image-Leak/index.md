---
title: "Halcon 影像記憶體洩漏"
date: 2020-10-06T20:25:42+08:00
draft: false
tags: 
  - Halcon
description: "假設因相機或機構問題，相機取到的影像需做簡易影像旋轉和翻轉功能。我們來做幾個簡單的實驗，驗證影像記憶體的佔用情況。"
---

# 問題情境
假設因相機或機構問題，相機取到的影像需做簡易影像旋轉和翻轉功能。我們來做幾個簡單的實驗，驗證影像記憶體的佔用情況。

需求：
1. 彩色黑白轉換
2. 水平翻轉
3. 垂直翻轉
4. 旋轉

![](Origin_Image.png)


# 跑一千次觀察記憶體使用量
![](Run_1000_Image_Process.png)

## 方案一
`HImage`依序做完所有的影像處理。
![](Function_1.png)
![](Function_1_Result.gif)

## 方案二
`HImage` 每做完一個影像處裡，就`Dispose`前一張影像`
![](Function_2.png)
![](Function_2_Result.gif)

# 結論
Halcon 每做完一次影像處理，其實是做資料複製，記憶體中的資料從一張影像會變成兩張影像，做越多次處裡記憶體的佔用也就越大，所以記得每次處理完都需要做影像銷毀。

附上[範例程式碼](https://github.com/patrick85081/HalconImageMemory)，執行前請先安裝`Halcon 12`  
