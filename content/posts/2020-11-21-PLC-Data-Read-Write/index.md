---
title: "PLC 讀寫的資料轉換"
date: 2022-03-30T20:04:23+08:00
draft: false
tags:
  - PLC
description: "當要跟PLC做資料交換溝通時，常常會碰到一個問題，PLC是16bit的系統，傳遞字串與數字時都需要經過一些處理，高位元與低位元的byte處理，這裡做個紀錄以免忘記。"
---
當要跟PLC做資料交換溝通時，常常會碰到一個問題，PLC是16bit的系統，傳遞字串與數字時都需要經過一些處理，高位元與低位元的byte處理，這裡做個紀錄以免忘記。

## 字串類型
PLC一個Device為16bit，一個字元為8bit，所以可放兩個字元。
### 寫入字串
我們需要將一串文字兩個字元一組轉成int，寫入int陣列進去PLC。
程式的邏輯應該長這樣
:::info
一個int = 高位元文字(char[1]) * 256 + 低位元文字(char[0])
:::

來，我們來看程式碼：
``` cs=
// 輸入的字串
string text = "Patrick";
int[] datas = (
    // 總共有幾組
    from num in Enumerable.Range(0, (text.Length + 1))
    // 每一組的起始 index
    let startIndex = num * 2
    // 低位元
    let low = text.ElementAtOrDefault(startIndex)
    // 高位元
    let heigh = text.ElementAtOrDefault(startIndex + 1)
    select height * 256 + low
)
.ToArray();
```

### 讀入字串
接下來我們需要將int 陣列轉回字串，一個int為兩個字元，程式的虛擬碼長這樣：
:::info
char[0] = 低位元 (char)(int % 256)
char[1] = 高位元 (char)(int / 256)
:::

來，我們來看程式碼
``` cs=
int[] datas;
char[] charArray = (
    from num in datas
    let chars new char[]
    {
        // 低位元
        (char) (num % 256),
        // 高位元
        (char) (num / 256),
    }
    from c in chars
    select c
)
.ToArray();
// 轉換回字串，過濾開頭結尾的 '\0' 的字元
var text = new string(charAray).Trim('\0');
```

## 32bit 數字
c#中的int全名就叫做int32剛好就是32bit，前面說過PLC一個Device只有16bit，所以要把兩個Device轉成int32
### 寫入32bit數字
我們需要拆解int32成4個bytes，兩個bytes一組轉成int16，最後再轉成int32
:::info
輸入數字 (int) => byte[4]
結果
int[0] = byte[0] byte[1]
int[1] = byte[2] byte[3]
:::
``` cs=
int value = -200;
var bytes = BitConverter.GetBytes(value);
var datas = (
    from num in Enumerable.Range(0, 1)
    let index = num * 2
    let int16 = BitConverter.ToInt16(bytes, index)
    let int32 = Convert.ToInt32(int16)
    select int32
)
.ToArray()
```

### 讀取32bit數字
我們從PLC讀出兩個DeviceBlock，得到長度為2的int陣列，我們需要各從這兩個int身上讀取兩個byte組合成一個int
```cs=
int[] datas;
byte[] byteArray = (
    from data in datas
    from @byte in BitConverter.GetBytes(data)
                    .Take(2)
    select @byte
)
.ToArray();

int result = BitConverter.ToInt32(byteArray, 0);
```

## 16bit數字
寫入沒有太大的問題，主要是讀取資料時候，數字又是負數，產生的問題
```
PLC Device  :       FF F0
到達C# int32 : 00 00 FF F0
```
這個時候數字解讀就會有問題
### 讀取16bit數字
``` cs=
int num = -200;
byte[] bytes = BitConverter.GetBytes(num).Take(2).ToArray();
short int16 = BitConverter.ToInt16(bytes, 0);
```