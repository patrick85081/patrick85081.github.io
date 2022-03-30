---
title: "PLC 連線設定"
date: 2020-11-08T22:37:59+08:00
draft: false
tags: 
  - PLC
description: "這裡來記錄一下PLC模擬器的設定方式，以及C#連接PLC的程式碼。"
---
這裡來記錄一下PLC模擬器的設定方式，以及C#連接PLC的程式碼。

## Communication 設定教學
### 模擬器連線設定
1. 設定 Station Number  

![](Emulator_Setting_1.png)

2. PLC類型 選擇模擬器 GX Simulator2  

![](Emulator_Setting_2.png)

3. 設定連線名稱  

![](Emulator_Setting_3.png)

4. 完成結果  

![](Emulator_Setting_4.png)

### 透過網路連線設定
1. 選擇 網路以及連線模式

![](Connect_1.png)

2. 設定IP

![](Connect_2.png)

3. 設定模式 以及 CPU類型

![](Connect_3.png)

1. 連線名稱

![](Connect_4.png)

1. 完成結果

![](Connect_5.png)

## GX Work2 啟動模擬器
1. 開啟新專案  

![](GX_Work2_1.png)

2. 開始 模擬  

![](GX_Work2_2.png)

## C# 開發方式
1. 加入參考 `MITSUBISHI ActMulti Control`  

![](CSharp_Develop_1.png)

2. 參考清單  

![](CSharp_Develop_2.png)

3. Coding

``` cs
var easyIf = new ActEasyIF();
easyIf.ActLogicalStationNumber = 1;
if (easyIf.Open() != 0)
    throw new Exception("連線失敗");

// Read Block (D0 ~ D49)
var buffer = new int[50];
easyIf.ReadDeviceBlock("D0", buffer.Length, out buffer[0]);

// Write Block (D0 ~ D49)
easyIf.WriteDeviceBlock("D0", buffer.Length, ref buffer[0]);

// Read Device
int value = 123;
easyIf.GetDevice("D0", out value);

// Write Device
easyIf.SetDevice("D0", value);
```