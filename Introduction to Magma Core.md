# Introduction to Magma Core
　
## Overview

* Magma project -- 一個開源的行動核心網路（mobile network core）
* Magma 支持多種無線電技術，包括 LTE、5G 和 WiFi
* Magma 能快速提供新的網路服務，因此非常適合將行動網路拓展到偏遠、人煙稀少的地區 
	* Magma 支援 2G 到 5G 的任何一代蜂巢式網路（Cellular network），支援 WiFi 和 CBRS（公民寬頻無線電服務）這樣的 unlicensed spectrum
	* Magma 可以與現有的蜂巢式網路聯合，使網路能夠擴展到偏遠地區
* Magma 幫助運營商有效地管理行動網路

目標

* [ ] Magma 的整體架構
* [ ] 它如何融入蜂窩網路架構 -- 尤其是 4G/LTE 和 5G
* [ ] 理解 mobile wireless network 的主要功能
* [ ] 理解每個 Magma components 的功能
	* Access Gateway
	* Federation Gateway
	* Orchestrator


## 01-Cellular Networks An Overview

目標

* [ ] 了解 Radio Access Network (RAN) 和 Mobile Core（有時被稱為 Evolved Packet Core 或　EPC）的功能
* [ ] 了解 CUPS（Control and User Plane Separation） -- User Plane 和 Control Plane 分離的概念


### Standardization Landscape

* Cellular network 的標準由 3GPP 制定
* 儘管其名稱中帶有「3G」，但 3GPP 仍在繼續定義 4G 和 5G 的標準
* 每個標準都對應一個 release，Release 15 被認為是 4G 和 5G 之間的分界點
* 4G LTE 並非真正的 4G，而是朝向 4G 目標前進的標準 -- Long Term Evolution (LTE)
* [ ] Magma 抽像出（abstracts out）了所有　cellular network standards 之間的差異。
	* 重要的是認識這些不同標準之間的差異

### Spectrum Allocation（頻譜分配）

* 與 Wi-Fi 一樣，Cellular network 以無線電頻譜中的特定 bandwidth 傳輸資料
* Wi-Fi 允許任何人使用 2.4 或 5 GHz 的頻道（unlicensed bands）
* 與 Wi-Fi 不同的是，政府將各種頻段的獨家使用權，拍賣並授權給網路服務提供商（service providers）使用
* 服務提供商又將移動接入服務（mobile access service）出售給他們的用戶（subscribers）

對網路運營商來說，複雜的是:

* Cellular network 在世界各地的許可頻段不同
* 網路運營商通常得同時支援新舊技術的標準（ex: 從 2G 到 5G）

* 頻譜分配與理解 Magma 沒有直接關係，但頻譜分配會影響物理層組件，進而對整個系統產生間接影響
* [ ] 確保分配的頻譜得到有效使用也是　cellular network 的設計目標


### Cellular Architecture Overview

![](01-Introduction%20to%20Cellular%20Networking/images/CellularNetwork.png)

(cellular network 的 High-level 架構圖)

 