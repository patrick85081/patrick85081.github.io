---
title: "Halcon 常用的Function"
date: 2020-11-18T22:18:52+08:00
draft: false
tags: 
  - Halcon
description: "這裡記錄一下，常用到的一些Halcon影像處理方法"
---

# 影像分割

## 固定二值化

``` csharp
// P3, P4 => 二值化閥值區間
threshold(GrayImage, DarkArea, 0, 128)
```

## 直方圖自動二值化

``` csharp
// P3 => Sigma 高斯運算 平滑算子
auto_threshold(GrayImage, DarkArea, 8.0)
```

>  P.S. 可用 `gray_histo` or `gen_region_histo` 查看直方圖

## 自動全局二值化

> 利用直方圖像素分佈，例如**最大類間方插法**或**平滑直方圖法**

``` csharp
// P3 => 二值化的方法 
// max_separability 直方圖中最大的可分性分割
// smooth_histo 平滑直方圖
// P4 => 取 亮部 light 還是 暗部 dark
// P5 => 輸出 自動二值化使用的閥值
binary_threshold(GrayImage, DarkArea, 'max_separability', 'dark', UsedThreshold)
```

## 局部閥值分割法

> 適用於無法用單一灰階分割情況，如背景複雜，亮暗不均

步驟

1. 套用平滑濾波器
2. 使用`dyn_threshold` 比較 原始圖像 與 套用平滑濾波器 後的影像差異，將差異大於設定值的點找出來

``` csharp
// P3 => 輸出的閥值區域
// P4 => Offset 值，比較後大於該值將被提取出來
// P5 => 哪個區域
// light 原圖 >= 平滑後 + Offset
// dark 原圖 <= 平滑後 - Offset
// equal (平滑後 + Offset) < 原圖 < (平滑後 - Offset)
// not_equal (平滑後 + Offset) >= 原圖 Or 原圖 >= (平滑後 - Offset)
dyn_threshold(Image, ImageMean, RegionDynThresh, 4, 'not_equal')
```

## Var_Threshold
``` csharp
// P3, P4 => Mask 長寬
// P5 => 標準差因子
// P6 => 絕對閥值
// P7 => dark, light, equal, not_equal
var_threshold(Image, Region, 15, 15, 0.2, 35, 'darkk')
```

## Char_Threshold
``` csharp
char_threshold(Imge, Image, Characters, 6, 95, Threshold)
```

## Dual_Threshold
``` csharp
dual_threshold(Imge, RegionCrossings, MinSize, MinGray, Threshold)
```

# 區域生長法
## RegiongRowing
> 將灰階相近的像素合併
``` csharp
// P3, P4 => 矩形區域長寬 (奇數)
// P5 => 灰階差的分割標準
// P6 => 輸出區域的最小像素
regiongrowing(Image, Regions, 1, 1, 3.0, 100)
```

---
# Bayer RG8 to Halcon

## Halcon CfaToRgb
> 將單通道彩色，轉換成三通道
``` cs
// cfa_to_rgb — Convert a single-channel color filter array image into an RGB image.
// CFAType: 'bayer_bg', 'bayer_gb', 'bayer_gr', 'bayer_rg'
// Interpolation: 'bilinear', 'bilinear_dir', 'bilinear_enhanced'
static void HOperatorSet.CfaToRgb(HObject CFAImage, out HObject RGBImage, HTuple CFAType, HTuple interpolation)
```

[Basler Grab result to Halcon](https://www.baslerweb.com/en/sales-support/knowledge-base/frequently-asked-questions/how-can-i-convert-a-pylon-grabresult-into-an-mvtec-halcon-image-buffer/103744/)

[图像Bayer格式介绍以及Bayer插值原理CFA](http://blog.chinaaet.com/justlxy/p/5100052454)


[图像bayer格式介绍以及bayer插值原理](https://zhuanlan.zhihu.com/p/72581663)