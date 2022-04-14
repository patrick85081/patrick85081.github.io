---
title: "Windows Port Forwarding"
date: 2022-04-14T13:47:43+08:00
draft: false
tags: 
  - Network
  - Port Forwarding
description: "今天來研究Windows平台如何做到 Port Forwarding的功能，將所有的封包進行轉發作業。"
---

今天來研究Windows平台如何做到 Port Forwarding的功能，將所有的封包進行轉發作業。

# 使用 Windows 內建的 Port Proxy 功能
只支援TCP協定
``` cmd
// 增加 Port Forwarding
netsh interface portproxy add v4tov4 listenport=4001 listenaddress=192.168.36.100 connectport=4001 connectaddress=192.168.36.75

// 刪除 Port Forwarding
netsh interface portproxy delete v4tov4 listenport=4001 listenaddress=192.168.36.100

// 清除所有的 Port Forwarding
netsh interface portproxy reset

// 顯示目前的 Forwarding 設定
netsh interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:
Address         Port        Address         Port
--------------- ----------  --------------- ----------
192.168.36.100  4001        192.168.36.90   4001
```

---
# 使用 Windows Server NAT Server 的 Port Mapping
首先先開啟 NAT 伺服器，使用 TCP/UDP Port對應，讓外部使用者可以存取內部伺服器。

## 安裝 NAT 伺服器
### 伺服器腳色安裝  
伺服器 》 管理 》 新增角色及功能  

![](Windows_Server_Enable_Nat_01.jpg)

選擇 **遠端存取**  
![](Windows_Server_Enable_Nat_02.jpg)

選擇 **RAS連線管理員系統管理組建(CMAK)**  
![](Windows_Server_Enable_Nat_03.jpg)

![](Windows_Server_Enable_Nat_04.jpg)

選擇 **路由**  
![](Windows_Server_Enable_Nat_05.jpg)

### 路由及遠端存取 設定  

![](Windows_Server_Enable_Nat_06.jpg)

伺服器管理員 > 工具 > 點選 路由及遠端存取  
![](Windows_Server_Enable_Nat_07.jpg)

選 **網路位址轉譯 (NAT)**  
![](Windows_Server_Enable_Nat_10.jpg)

選擇 網路卡  
![](Windows_Server_Enable_Nat_11.jpg)

![](Windows_Server_Enable_Nat_12.jpg)

### 配置 NAT 設定  
![](Windows_Server_Enable_Nat_13.jpg)

![](Windows_Server_Enable_Nat_14.jpg)

![](Windows_Server_Enable_Nat_15.jpg)


## Routing Nat Port Mapping
### 使用 Command Line 設定
``` cmd
// 增加 Routing Nat Port Mapping
netsh routing ip nat add portmapping Ethernet0 udp 0.0.0.0 4000 192.168.36.75 4000
// 刪除 Routing Nat Port Mapping
netsh routing ip nat delete portmapping Ethernet0 udp 0.0.0.0 4000

// 顯示目前 Routing Nat Port Mapping
netsh routing ip nat show interface Ethernet0


NAT  Ethernet0 設定
---------------------------
模式              : 位址和連接埠轉譯


NAT 靜態連接埠對應設定
-------------------------------------
通訊協定          : UDP
公用位址          : 0.0.0.0
公用連接埠        : 4000
私人位址          : 192.168.36.90
私人連接埠        : 4000
```

### 使用 UI 設定
![](UI_Port_Mapping_01.png)

新增 Port Mapping  
![](UI_Port_Mapping_02.png)

設定相關對應資料  
![](UI_Port_Mapping_03.png)


## 實際測試結果
測試環境
* 192.168.36.88  
  向 192.168.36.100 發送 UDP封包
* 192.168.36.100  
  使用 NAT 轉發 UDP 4000 給 192.168.36.90
* 192.168.36.90  
  接收 UDP 封包
  
![](./UDP_Port_Forwarding_Result.png)


---
# 參考資料
* 通訊埠轉發
  - [在 Windows 上配置埠轉發](http://woshub.com/port-forwarding-in-windows/)
  - [Windows 上 UDP 流的简单端口转发](https://cxywk.com/q/Cd3KMBps)

* NAT 伺服器
  - [Windows Server 2012筆記(六) NAT伺服器設定](https://blog.pmail.idv.tw/?p=5165)