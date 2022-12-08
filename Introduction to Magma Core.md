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

* [ ] 了解 Radio Access Network (RAN) 和 Mobile Core（有時被稱為 Evolved Packet Core 或　EPC）的主要功能
	* RAN 由相互連接的基站組成
	* 這些基站負責處理 UE 無線電信號的發送和接收
* [ ] 了解 CUPS（Control and User Plane Separation） -- User Plane 和 Control Plane 分離的概念
	* 用戶平面（UP）負責移動數據和語音
	* 控制平面（CP）處理身份驗證、安全、使用監控和移動管理等功能


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
* 確保分配的頻譜得到有效使用也是　cellular network 的設計目標


### Cellular Architecture Overview

![](01-Introduction%20to%20Cellular%20Networking/images/CellularNetwork.png)

(cellular network 的 High-level 架構圖)

* cellular network 由兩個主要子系統組成:
	* Radio Access Network (RAN)
	* Mobile Core（在 4G 中稱為 Evolved Packet Core 或 EPC）
* User Equipment (UE) 這個術語，是指連接到網路的設備，包括行動電話和其他設備，例如具有 cellular interface 的路由器
* RAN 管理無線電頻譜，確保它滿足每個用戶對 QoS 的要求。它對應於一組分散式的 base stations，通過 backhaul network 連接 Mobile Core

Mobile Core 不應看成一種設備，而是一群功能的綑綁。可應付多種目的：

* 為數據和語音服務提供 Internet (IP) 連接
* 確保上述的連接滿足 QoS 的要求
* 跟踪用戶的移動性，以確保服務不間斷
* 跟踪用戶（subscriber）使用情況，以進行計費和收費


### Control and User Plane Separation (CUPS) 控制和用戶平面分離

![](01-Introduction%20to%20Cellular%20Networking/images/Control_and_User_Plane_separation_in_the_RAN_and_Mobile_Core.png)

(RAN 和 Mobile Core 中的 CUPS 示意)

* RAN 由一組連接到 Mobile Core 的 base stations 組成。
* base station 連接到用戶設備 (UE) -- 手機或其他 cellular 設備

* Mobile Core 被劃分為「Control Plane 控制平面」和「User Plane 用戶平面」
	* 類似於 Internet 上所認為的 control/data plane split
* 3GPP 使用 CUPS 這個術語 -- Control and User Plane Separation -- 來表示這個概念

重點

* User Plane 承載數據或語音信息 -- 事實上就是用戶會接觸、或產生的東西
* Control Plane 負責識別用戶、確定他們有權獲得哪些服務，以及追踪使用情況以進行計費等功能

與控制平面的概念密切相關的是信令（signalling）。Signalling messages 在控制平面設備和 UE 之間交換以建立 user path channel

**NOTE**: [signalling](https://en.wikipedia.org/wiki/Signaling_(telecommunications)) 是電信領域的詞彙，指為了使通信網中各種設備協調運作，在設備之間傳遞的有關控制信息。例如 DTMF 就是 PSTN 裡的一種 signalling


### Radio Access Network (RAN) Overview 無線電存取網路概述

![](01-Introduction%20to%20Cellular%20Networking/images/A_RAN_is_implemented_by_a_set_of_interconnected_base_stations.png)

* 一個 RAN 是由一組相互連接的基站（base stations）實現的
* 基站包括天線，它處理所有特定於無線電的通信功能

基站（Base station）命名術語:

* 4G 中被命名為 eNodeB（或 eNB） -- 是 evolved Node B 的縮寫
* 5G 中，被稱為 gNB（g 代表 "next Generation"）

### RAN Functions

最佳了解 RAN 的方法就是是去看單個基站如何對新 UE 的到來做出反應。

* 首先，subscriber 的 UE 開機時，或是從另一個基站切換過來時，基站會替它建立 wireless channel
* 接著，基站在 UE 和 Mobile Core 之間建立 Control Plane 連接，並轉發兩者之間的 signaling。此 signaling 會啟用 UE 身份驗證、註冊和移動追踪
* 最後，對於「每個 active 的 UE」，基站建立一個或多個 tunnels 到 Mobile Core 的 User Plane component。

![](01-Introduction%20to%20Cellular%20Networking/images/Tunnels_for_the_User_Plane_from_Base_Station_to_Mobile_Core.png)

(Tunnels for the User Plane from Base Station to Mobile Core)


* 基站能夠在 Mobile Core 和 UE 之間轉 control 和 user plane 的封包
* 基站之間會有連線，因此可處理 UE 與相鄰基站的切換 -- 一個 UE 可能同時被多個基站服務　 

![](01-Introduction%20to%20Cellular%20Networking/images/A_UE_may_be_served_by_two_base_stations_at_one_time.png)


### Mobile Core Overview

![](01-Introduction%20to%20Cellular%20Networking/images/Major_functional_blocks_of_the_Mobile_Core.png)

(4G Mobile Core 的主要 functional blocks)

* 上圖顯示了 4G 移動核心的主要功能塊，也稱為 Evolved Packet Core (EPC)
* 在 5G 中，Mobile Core 稱為 NG-Core，代表 Next Generation
* 從 4G 到 5G，這些 functional blocks 在 3GPP 架構下可能會以不同的名稱表示，但主要功能都是一致的
* Mobile Core 的功能包括　user plane components（SGW 和 PGW）和 control plane components（HSS、MME、PCRF）


### Mobile Core Main Components (4G)

4G Mobile Core，也稱為 Evolved Packet Core (EPC)，由五個主要 components 組成，前三個在控制平面 (CP) 中運行，後兩個在用戶平面 (UP) 中運行：

* CP（control plane）
	* MME（Mobility Management Entity）
		* "Mobility" 顧名思義。追蹤和管理 UE 在 RAN 中的移動。包含在 UE 不活動時進行記錄
	* HSS（Home Subscriber Server）
		* 一個用來儲存用戶（Subscriber）相關資訊的資料庫
	* PCRF（Policy & Charging Rules Function）
		* 記錄用戶流量的計費功能
* UP（user plane）
	* SGW（Serving Gateway）
		* 將 IP 封包轉發到 RAN，或從 RAN 接收 IP 封包
		* 將承載服務的 Mobile Core 錨定（Anchors）到 UE，因此涉及從一個基站到另一個基站的切換（記得會有多個基站連線到 Mobile Core 吧）
	* PGW (Packet Gateway)
		* 本質上是一個 IP 路由器，將 Mobile Core 連接到外部 Internet
		* 支持額外的存取控制功能，例如流量和計費

NOTE: 儘管在邏輯上這些 components 都是個體，但在實務上，（往 RAN 方向的）SGW 和（往 Internet 方向的）PGW 通常組合在同一個設備中，稱為 S/PGW


### Mobile Core: Authentication 身份驗證

從上面的架構圖來看，基站進入 Mobile Core 之後遇到的第一個 component 就是 MME 和 SGW，所以可以想像得到關於身份驗證應該會透過這兩個單元組件進行...

![](01-Introduction%20to%20Cellular%20Networking/images/Sequence_of_steps_to_establish_secure_channels_for_Core_and_User_planes__1_.png)

(為核心和用戶平面建立 secure channels 的步驟順序)

下面的例子也說明了 mobile core 的一些關鍵功能:

* UE 安裝了電信營商提供的 SIM 卡，SIM 卡使得用戶能有一個可識別的唯一性，並且記錄了與基站通訊時所需的無線電參數（例如，frequency band）
* SIM 卡還包括 UE 用來驗證自己的 secret key

1. 未驗證的 UE 與附近的基站建立臨時的連線
2. 基站將請求轉發給 Core-CP，Core-CP 啟動與 UE 的身份驗證協議
	* 在 4G 中，執行此步驟的控制平面組件稱為 Mobility Management Entity (MME)
	* 此步驟向彼此驗證 UE 和 Mobile Core
3. 為了能提供 UE 服務，Core-CP 將所需的參數通知其他組件
	a. 指示 Core-UP 初始化用戶平面（例如，為 UE 分配 IP 地址）
    b. 指示基站建立與 UE 的 encrypted channel
    c. 在上述的 encrypted channel 中，向 UE 提供對稱密鑰（symmetric key）
4. 最後，UE 可以透過 encrypted channel 與 mobile core 的用戶平面通訊
	* 在 4G 中，UE 與之通訊的用戶平面組件是 SGW（Serving Gateway）。

步驟 1 到 3 都是一種 signaling 形式：在可以建立用戶平面連接之前發生的步驟。

NOTE:（不負責心得）我自己的感覺是，這種分離可能來自於電信網路的發展史，一開始是 circuit switched network，在這種網路下傳送的都是基礎的電子訊號。進入 3G 以後為了 packet switched network，需要做些融合。概念上有點像是把用戶平面的東西往 packet switched 方向轉移，而控制平面仍維持在 circuit switched 時代的思維。（這講法可能不是很精確 XD）


### Mobile Core: Mobility 移動性

![](01-Introduction%20to%20Cellular%20Networking/images/Sequence_of_steps_to_establish_secure_channels_for_Core_and_User_planes__1_.png)

（UE 的移動性是透過重新執行一個或多個身份驗證來實現的）

* 當 UE 在 RAN 周圍移動時，它會進入多個基站的訊號範圍內
* 根據訊號的測量品質，基站與基站之間會直接相互通訊以做出切換的決定
* 一旦基站做出切換決定，就會將決定傳達給 Mobile Core，然後重新觸發步驟 (3) 的流程
	* 配發 IP
	* 建加密通道
	* 提供 key
* 最後，在 UE 和  Mobile Core 之間建立了新的用戶平面連接（ex: 對行動通訊用戶來說就是：可以上網啦）
* Mobile Core 的用戶平面，會在 UE 切換基站的過渡期間 buffers data，避免丟包和重傳

換句話說，cellular network 在 Mobility 時，會保持 UE session　-- 當然是指同一家的 Mobile Core serves（ex: 都是中華電信）

* 若是在 Mobile Core 之間移動（ex: 中華電信的範圍移到遠傳的範圍）
	* 對基站而言可以看做是 UE 充新上電
	* UE 被分配了一個新的 IP 地址
	* 沒有 buffers data

剛剛說，cellular network 會盡可能地保持 UE session 來做到 Mobility。但長期處於 inactive 的 UE 遺失 session，當 UE 再次 active 時會建立新的 session 並分配新的 IP 地址。


## 02-Magma Architecture Overview

Magma的主要組成部分：

* [ ] Access Gateway（AGW）：為 Mobile Core 實現了 user plane 和 mobility 的管理功能
* [ ] Orchestrator（orc8r）：它為 Mobile Core 提供 control 和 configuration 的中控中心，並與許多其他 components 連接
* [ ] Federation Gateway（FGW）：允許 Magma 擴展「現有移動網絡」
* [ ] Network Management System（NMS）：它提供了一個 GUI，用於 provisioning 和操作基於 Magma 的網絡

Magma 在幾個重要方面與 3GPP 的傳統實現方式不同，特別是在實現無線接入（radio access）技術**獨立性**的方式上。

目標

* [ ] 熟悉 Magma 的主要 building blocks
* [ ] 了解 Magma 的關鍵設計原則
* [ ] 討論 Magma 架構在提高 reliability、scale、operations 和 heterogeneity 方面的主要優勢



### Architectural Principles

Magma 從現代 cloud data center 的架構原則中借鏡，並將它們套用於 mobile core 的架構中。概括如下：

* 控制平面（control plane）和數據平面（data plane）分離
	* 控制平面元素的故障不應導致數據平面失敗
* 將數據平面狀態推到邊緣（edge）
	* 例如，每個 UE 的狀態不應出現在網路的核心中，而應僅存在於靠近邊緣的設備中（例如，在靠近 UE 的 AGW 中）
	* [ ] 這使得系統更具可擴展性和容錯性
* 中心化儲存 configuration state，在邊緣存儲 runtime state
	* 這與軟體定義網絡 (SDN) 中採用的方法相同
* Desired State Configuration（DSC）模式
	* APIs 允許用戶配置他們的預期狀態，而控制平面負責確保實現該狀態
* 將 core network 與 radio access network 的細節隔離開來
	* 3GPP 針對不同世代的無線電在 mobile core 中做出了不同的設計選擇
	* Magma 對所有無線電接入技術採用「"common（通用）" core design」，包括公民寬頻無線電服務 (CBRS) 和 WiFi 等非 3GPP 技術
* 軟件控制的數據平面
	* 可通過獨立於硬件的 interfaces 管理數據平面
* 在各 components 之間使用標準的分散式技術進行通信
	* 例如 Magma 的主要組件使用 gRPC 相互通信
	* gRPC 是一種開源的 RPC 框架
* 預計個別組件會失效
	* 一個組件的故障會影響盡可能少的用戶（即 small fault domains）並且不會影響其他組件
	* 「組件失效是正常的，並且必須在設計中處理」的假設是 cloud native applications 的典型思維，但在傳統電信網路的架構中並不常見

NOTE:

* 儘管 3GPP 具有控制和用戶平面分離 (CUPS) 的概念，但控制平面元素通常會持有一些用戶平面（數據平面）狀態
* （Magma 所說的）控制平面和數據平面的適當分離，能使系統具有更好的 upgradability（可升級性）與更健壯的架構。


### Magma Architecture Overview

![](02-Introduction%20to%20Magma%20Architecture/images/Magma_High-Level_Architecture.png)

(Magma High-Level 架構圖)

* 「Orchestrator 是一種"雲服務"」，提供了一種簡單、一致且安全的方式來配置和監控無線網絡
	* Orchestrator 可以託管在公共/私有雲上
	* 透過 Orc8r 平台獲取的 metrics（權值？）允許您透過 Magma web UI 查看無線用戶的分析和流量
	* Orchestrator 實現了中心化的 management plane，以及 Magma control plane 的各個方面
* Access Gateway (AGW) 實現 Evolved Packet Core (EPC) 的 runtime 特性
	* 內容就是 SGW、PGW 和 MME 等所做的事情
		* NOTE: 記得上面說 "在邊緣存儲 runtime state" 和 "UE 的狀態推到邊緣 -- AWG" 嗎？
	* 「Runtime」指的是那些由於 UE 開機、使用配額消耗或移動事件等事件而頻繁變化的功能 -- 不就是 4G 中提到的 MME、SGW 所做的事情？
	* 一個 Orchestrator 通常管控多個 AGW，一個 AGW 通常支援多個基站
* Federation Gateway (FGW)
	* 視為連結現有 mobile core 的標準接口
	* 透過 gRPC 與 Orchestrator 通訊
	* 使得 Magma 系統能夠連接並擴展現有的 mobile core network
		* 例如利用現有的用戶數據、計費系統和政策

NOTE: 這張圖沒有顯示 Magma 的第四個組件 NMS（network management system）


### Management, Control, and Data Planes

3GPP 中規範了 Control Plane 和 User Plane 的概念。但在 Magma 的系統中使用不同的術語：management, control, 以及 data planes

![](02-Introduction%20to%20Magma%20Architecture/images/Management__Control__and_Data_Planes.png)

(Management, Control, and Data Planes 示意圖)

* 在 Magma 的架構下，Data Planes ​分佈在一組 AGW（以及非必要的一個或多個 Federation Gateways）中
	* 它涉及 data packets 的轉發和策略的執行
	* 概念上接近 3GPP 所說的 User Plane -- 可以想成是用戶會產生的資料
* control plane 在邏輯上是中心化的，它決定如何對 data plane 進行更新，以確保實際呈現 desired state
	* 回想剛剛說的 -- "中心化儲存 configuration state"
	* 回想剛剛說的 -- "DSC 模式下，允許用戶配置他們的預期狀態，而控制平面負責確保實現該狀態"
* control plane 會從 data planes 收集資訊。例如，了解 UE 的新位置
	* 這稱為 discovered state
	* 當 discovered state 和 desired state 之間存在差異時，control plane 有責任決定如何解決該差異
* management plane 為 configuration 和 querying 提供單一進入點（其實就是 APIs 啦）
	* 當 configuration info 透過「northbound API」提供給 management plane 時，更新 desired state

這樣的「desired state model」是 cloud native systems 中的常見模式

* control plane 的主要工作是: 在面對 configuration changes、components failures 或其他事件（例如新 UE 的入網）時，不斷協調系統的 actual state 與 desired state

這是架構上的邏輯視角，而非物理視角

* 圖中的 control plane 在邏輯上是中心化的，但實務上可能在可用性、故障隔離和其他考量下，會以分層和分散式的方式部署
	* control plane 的某些部分在 Orchestrator 中集中運行，而其他部分則分散到 AGW -- 記得之前說過 "一個 Orchestrator 通常管控多個 AGW" 嗎？
	* 運行在 orc8r 中的控制平面部分稱為 central control plane
	* 運行在 AGW 中的部分控制平面稱為 local control plane

3GPP 的控制平面與 Magma 控制平面不同

* 3GPP 控制平面可以理解為一種 signaling 功能，用來建立 User Plane channel
* 3GPP 並沒有真正集中式的 state model，因此不會對應到上述的 desired state model

不負責小感想

3GPP 的思維起點並非架構導向的，比較像是流程導向，他像是在思考 -- 為了讓用戶上網（User Plane），我該進行那些控制行為（Control Plane）？而那些控制行為要傳送什麼 signaling？

Magma 則是從 state model 的角度思考，我希望系統成為什麼樣子（desired state）？目前是什麼樣子（discovered state）？要怎麼實現它（Realized State）？然後依據 state 職責劃分出 Management, Control, 以及 Data Planes。


### The Orchestrator (a.k.a orc8r)

* 在剛剛描述的模型中，orc8r 實現了 management plane -- 透過 REST APIs 讓 Magma 能提供 configuration interface
* orc8r 還實現了 Magma control plane 的中心化思維 -- 根據 configuration info 收集（discovered）各種（state），然後將指令 push 到 AGWs
* orc8r 還整合了其他系統（一些開源套件），以提供在電信領域常見的 FCAPS（fault, configuration, accounting, performance, security | 故障、配置、記帳、性能、安全）管理功能

![](02-Introduction%20to%20Magma%20Architecture/images/Orchestrator_and_its_main_interfaces.png)

(Orchestrator 及其主要界面)

* orc8r 以雲服務來實現
	* 如上圖紫色框所示，服務可以有多個 instance
	* 具備容錯性、Failover
	* 並且可以通過額外的 instance 進行橫向擴展 -- 這些擴充的 instances 在邏輯上仍會有個中心化的 control plane 和 API 進入點
* Orchestrator 可連接到其他服務（如右側所示）
* 一個 Orchestrator 連接到多個 Magma Access Gateways
* gRPC 用於將 status info 從 AGW 傳送到 orc8r，並將 configuration info 推送到 AGW

Orchestrator 可以連接到傳統電信提供的 Operations Support 和 Billing Support (OSS/BSS)系統，也可以單獨運作一個小型的 OSS/BSS -- 小型的運營商可能會需要這樣的東西

Orchestrator 實現前面提到的「desired state model」:

* 用戶輸入 desired state（例如，某個 subscriber 應該可以存取具有特定流量的網路）
* Orchestrator 通過將更新推送到適當的 gateways 來確保實現(realized) desired state
* 有一個 loop，持續協調 realized state 與 desired state

理解 Orchestrator 和 AGW 之間的分離，有助於區分 configuration state 和 runtime state

* 記得前面說過: 中心化儲存 configuration state，在邊緣存儲 runtime state
* configuration state 由運營商提供，例如，新增一組與新用戶相關的策略 -- orc8r 處理
* runtime state 由真實運作的行為產生，例如 UE 上電，或從一個基站到另一個基站的移動性切換 -- AGW 處理（它在邊緣 -- 靠近用戶）


### Access Gateway (AGW)

* Access Gateway (AGW) 提供 EPC 的 runtime functionality
	* 這些功能包括（4G 下的）
		* User Plane functions: SGW、PGW
		* Control Plane function: MME。
* 一個 AWG 通常有幾個基站，每個 Orchestrator 有多個 AGW -- 可以根據需要添加額外的 AGW 擴展網絡

![](02-Introduction%20to%20Magma%20Architecture/images/High_level_view_of_Access_Gateway.png)

* AGW 向 Orchestrator 報告 runtime state
* Orchestrator 將配置命令推送給 AGW，例如
	* AGW 負責生成加密密鑰（runtime state），以便 UE 可以安全地與 mobile core 通訊
	* Orchestrator 負責配置允許 UE 連接到網絡的策略（config state）

### Federation Gateway (FEG)

![](02-Introduction%20to%20Magma%20Architecture/images/Magma_High-Level_Architecture.png)

* FEG 透過「標準 3GPP 介面」連接「現有 MNO components」
* FEG 讓移動網絡運營商 (MNO, Mobile Network Operator) 的核心網絡能銜接 Magma 
* 它充當 Magma AGW 和運營商網絡之間的 proxy
* 這讓身份驗證、data plans、策略執行和計費等核心功能，能在現有 MNO 網絡和 Magma 網路之間保持統一


### Network Management System (NMS)

![](02-Introduction%20to%20Magma%20Architecture/images/Magma-arch@2x.png)

* Magma NMS 是一個能夠進行 provisioning 和設定 Magma 網路 的 GUI (web)介面
* 「It is implemented on top of the REST API exposed by the orchestrator」
	* NMS 是一個 GUI，但真正操作的是 orchestrator 提通的 REST APIs
* NMS GUI 夠建立和管理:
	* Organizations
	* Networks
	* Access Gateways 和基站（eNodeNs, GNodeBs）
	* Subscribers
	* Policies and APNs (Access Point Names)

* NMS 是 Magma 系統的 optional component，因為可以從其他系統（例如運營商現有的 NMS）直接與 Magma 的 RESTful API 通訊
* NMS 還提供 dashboards 來監控 infrastructure 和系統的健康狀態


## 03-The Orchestrator

* Orchestrator 是 configuration input 和 monitoring Magma 部署狀態的中心點
* 傳統的 3GPP 部署需要管理各個網路元件，而 Orchestrator 提供了 network-wide 的 config 能力和 status 視角

目標

* [ ] 了解 Orchestrator component 的主要功能
* [ ] 了解 Orchestrator 如何與 Magma 的其餘部分通訊
* [ ] 了解 Orchestrator 如何與各種外部 component 溝通，以支持 logging 和 metrics gathering 等功能（稱為 FCAPS 的功能——故障、配置、記帳、效能、安全性）


### Orchestrator Overview

Orchestrator 及其主要介面

![](03-The%20Orchestrator/images/Orchestrator_and_its_main_interfaces.png)

* Orc8r 整合了一系列開源套件的功能，如 ElasticSearch Kibana、Fluentd 等
* Orc8r 使用 gRPC 與 AGW 和 FEG 通訊
* 它 "exposes" 了一個用於管理和配置的「northbound REST API」
* Orc8r 是個在公有或私有雲上實現的 cloud service
* Orc8r 透過 gRPC 與 AGW "雙向" 通訊
	* Orchestrator 將 configuration info 發送到 gateways，然後 gateways 回報 status, metrics, events...等等
	* 上面說的 “gateways” 可以是多個 AGW 或 通常只有一兩個的 FEG


### Orchestrator Functions

Orchestrator 支援的功能：

* Network entity configuration (networks, access gateways, federation gateways, subscribers, policies, etc.)
* Metrics querying（透過整合到 Orc8r 的 Prometheus 和 Grafana 套件功能）
* Event and log 聚合（透過整合到 Orc8r 的 Fluentd 和 Elasticsearch Kibana 套件功能）
* Device state reporting（metrics 和 status）


所有的功能都是中心化的

* 無論部署了多少 radio access points（eNodeBs）和 Access Gateways (AGWs)，都可以透過 Orchestrator 提供的　API 完成配置和監控
* 相較於多數 3GPP 的實作方式，必須對每個 component 進行配置，而 Magma 將此類配置全集中在 Orchestrator
	* 無論是增加新的 AGW 還是新的用戶，都可以透過一個 API 完成
	* Orchestrator 將設備的管理抽象化

所有 Orchestrator 功能都透過訪問 REST API 實現

* 管理者可以使用 NMS 訪問（因為 NMS 使用 Orchestrator 開放的 APIs）
* 也可以讓營運商現有的 dashboards 或系統整合近來


### FCAPS (Fault, Configuration, Accounting, Performance, Security)

* FCAPS 是電信行業的術語，用來泛指與「管理網路」相關的事情
* 重要的是 -- 發生異常時診斷問題；即將出現效能影響時發出警報


### Orchestrator Architecture

* Orchestrator 具備 modular and extensible
* modularity 允許運營商僅部署所需的服務
	* 例如，可以安裝內含或不含 metrics module 的 Orchestrator
	* metrics, alerts, logging 等功能，都是由透過整合開源套件到 orchestrator 提供的

在設計上，Core Orchestrator 的原則是與特定技術的細節無關，所以核心的服務都是 "domain-agnostic" 的 -- 這裡指的應該就是 FCAPS 那些事情。但透過 "modularity" 模塊化的特性，系統仍可以為核心提供 "domain-specific" 的支援 -- 例如對 LTE 的支援。

* Orchestrator 的模塊化架構是使用 service mesh (服務網格) 方法實現的
	* 每個 module 都被實現為一個 microservice (微服務)，可以獨立於其他 module 進行橫向擴展


### Configuration Example

![](03-The%20Orchestrator/images/Example_of_configuration_of_new_AGW_by_the_Orchestrator.png)

假設運營商想要新增一台支援 LTE 功能的 AGW:

* 透過呼叫 Orchestrator 的 REST API 啟動 -- 該 API 呼叫可由 NMS 或另一個系統產生
* 因為這請求是專屬於 LTE 的特定功能，因此 API 請求將被導到 LTE 模組處理
* 更新 "desired state" 以反映「有一台新 AGW 要註冊」的事實，並生成並傳遞一組 configuration info 並給 AGW 以實現 state


### State Checkpointing

* Orchestrator 接收 configuration inputs，並且扮演 desired state 的 repository。
* Orchestrator 還得接收並儲存旗下的 AGWs 回傳的 "runtime state" -- ex: active UEs 當前的狀態
* orchestrator 中的 “state service” 維護這些資訊，以便 orchestrator 的其他元件可以利用 checkpointed state 執行各種任務
	* 例如: 根據 UE 的當前位置將 message forward 到適當的 AGW

由於 state 都保存在中心化的 repository，Orchestrator 得以支援以下功能：

* 源自 federated network 的請求可以根據 UE 位置正確 forward 到適當的 AGW
* 如果 UE 從一個 AGW 移動到另一個 AGW，其相關狀態可以轉移到新的 AGW 以支持 mobility (移動性)
* 可以利用這個 checkpointed state 來提高 AGW 的可用性 (availability)


### Metrics and Monitoring

* Metrics 由 gateway components 生成，並回報給 Orchestrator 進行存儲和分析
* Magma 整合 Prometheus -- 將 Metrics 存儲在 time-series database
* Magma 還整合了 Grafana -- 將 Prometheus 收集到的資料進行可視化和分析

![](https://ralph.blog.imixs.com/wp-content/uploads/2019/01/imixs-cloud-grafana-768x449.png)


```
$ docker ps --format '{{.Names}}'
nms-magmalte-1
orc8r-nginx-1
orc8r-controller-1
orc8r-postgres_test-1
orc8r-alertmanager-1
orc8r-prometheus-1
orc8r-prometheus-configurer-1
orc8r-prometheus-cache-1
orc8r-maria-1
orc8r-user-grafana-1
orc8r-postgres-1
elasticsearch
orc8r-alertmanager-configurer-1
```


### Events and Logging

Orchestrator 可追蹤事件:

* Subscriber session 追蹤
* Subscriber 資料使用和 bearer creation
* Gateway 連接狀態
* eNodeB 狀態追踪
* New gateway configurations pushed

Orchestrator 使用 Fluentd 來存儲事件追踪的資料（就是指 log 吧）。Elasticsearch Kibana 則用於支援資料查詢


### Security

* Orchestrator 使用 TLS（Transport Layer Security）來保護通訊
* 多數情況下，連線都是相互驗證的，即 client 和 server 都使用 certificates 向另一方驗證自己的身份

例外情況:

* 一旦通過身份驗證，client certificate 將轉換為一組 "trusted metadata"
* 從此，各個服務可以通過查詢此 metadata 來做出有關 access 的決策 -- 在 Orchestrator 中由 accessd 服務處理

對於 "northbound API"，請求必須包含已 provisioned 的 client certificate -- 可用　Orchestrator CLI 產生此證書


## 04-The Access Gateway

* Access Gateway (AGW) 也是模組化設計的

目標

* [ ] 了解 Access Gateway component 的主要功能。
* [ ] 了解 AGW 如何與 Magma 的其餘部分通訊
* [ ] 了解 AGW 與傳統 3GPP mobile core 在實作上的差異


### AGW Architecture

![](04-The%20Access%20Gateway/images/Major_Functional_Blocks_of_the_Access_Gateway.png)

(AGW 的主要 Functional Blocks)

* 模組化設計允許將 "radio-technology-specific" 的細節限制在幾個特定模組中
* 透過維護 "small fault domains" 來提高 availability 和 upgradeability 的目標
* 每個模組都可以獨立升級和重啟

如前所述，每個 AGW 都連到一個 Orchestrator。Orc8r 實踐了 centralized control plane

* AGW 包含與 UEs 相關聯的所有 runtime state
* 多個 eNodeB 可能連接到單個 AGW
	* 5G 情況下是 gNodeB 
* AGW 可以支援多種無線技術，包括 5G/4G/LTE，以及 WiFi 和 CBRS（公民寬帶無線電服務）


* Orchestrator 和 AGW 之間的通訊使用 gRPC
* AGW 內部的模組間通信也使用 gRPC
* eNodeB（或其他基站類型）的介面是由 3GPP 規範的


* AGW 中的所有 "stateful services" 將其記憶體中的狀態寫入 Redis 的 key value 存儲
* 在啟動時，服務從狀態存儲中讀取它們的 runtime state，並使用 GRPC 介面將必要影響傳遞到其他服務


### Device Management

設備管理需要兩個主要元件：

* AGW 本身的管理
* 管理 RAN 設備，例如 eNodeBs

AGW 的管理獨立於無線電技術並且需要：

* 確保 AGW 的所有元件服務都正常啟用且健康
* 收集並向 Orchestrator 報告其他服務的 metrics
* 與 Orchestrator 通信以 bootstrap AGWs


* 設備管理中有一個 technology-specific 的模塊，它知道如何與特定類型的 RAN 設備（例如 4G/LTE 中的 eNodeB）進行通信
* 可以根據需求增加模組，以支援各種無線電接入技術，無需更改 AGW 系統的其餘部分

這突出了 Magma 設計與傳統 3GPP 實施之間的兩個主要區別：

* 可以從一個介面（Orchestrator）集中管理一組 eNodeBs（或其他基站）
* implementation 獨立於無線電接入技術（4G/LTE/5G/WiFi）。只有 AGW 中的特定模塊了解無線電技術，並知道如何使用適當的通訊協定與基站進行通訊


### Access Control and Management

* AGW 的訪問控制元件會在 UE active 時執行身份驗證
* 並建立從 UE 到 mobile core 的 data plane 元件（4G/LTE 中的 SGW 和 PGW）的 secure data path
* 必須訪問 subscriber database 才能執行像是驗證用戶是否有權訪問網路、獲取用於驗證的 key


### Subscriber Management

* 用戶管理功能有效地提供了用戶資訊的 local store
* 相當於 3GPP 標準中的 Home Subscriber Service (HSS)
* 如果 Magma 與現有的 MNO network 連接，則無需使用 local subscriber management
* 當 AGW 在沒有聯合(MNO)的情況下運行時，subscriber management 存儲由 Orchestrator 提供的 subscriber profiles


### Session and Policy Management

* 當 UE "attaches" 到 mobile core 時，需要建立一組 session policies
	* 例如，用戶可能有權獲得特定的 data rate 或 QoS
* 為實踐這些策略，Orchestrator 將 policy rules 提供給 AGW 中的 policy database
* 為了執行策略，session management 元件向 data plane 提供資訊，以便 QoS、速限等設定可以適當地套用到 data plane 中並執行


### Data Plane (Packet Processing Pipeline)

* Magma 的 data plane 是使用可程式化的 forwarding model 實現的 -- based on "Open vSwitch (OVS)"
* 可以在 data traffic 通過 AWG 時對其執行一系列策略 -- "programmable"
* OVS 根據 session 和 policy management 元件提供的 forwarding rules 強制執行
* packet processing pipeline 甚至可以使用深度封包檢測 (DPI, deep packet inspection) 在各個協議層 (L2-L7) 執行規則


### Telemetry and Logging

AGW 可以收集事件並將其傳遞給 Orchestrator 以進行後續分析，包括：

* 與 access gateway 功能相關的事件（例如服務重啟）
* 與 subscriber sessions 相關的事件（例如 session 開始/結束）
* 與 MME 功能相關的事件（例如 subscriber attach/detach）

還有一種追踪功能，用於捕獲無線接入點和 AGW 之間流動的 control packets，以進行故障排除

該服務透過 Orchestrator 的 API 呼叫啟用，然後 Orchestrator 將請求傳給 AGW 中的 tracing module -- 中心化控制


### Optional Services

包括使用 ping 監控連接到 Magma 服務的 CPE（客戶端設備）設備的活躍度，並將此報告回給 Orchestrator


## 05-The Federation Gateway

* FEG 是 Magma 與 “標準” 3GPP 世界之間的介面

目標

* [ ] 了解 FEG component 的主要功能
* [ ] 了解 FEG 如何與 Magma 的其餘部分互動
* [ ] 了解 federation 的 use cases

FEG 滿足了擴展現有 MNO 網路（例如，增加覆蓋範圍或到達偏遠地區）的目標，同時保持用戶、政策和計費資料的一致性


### Federation Gateway Overview

![](05-The%20Federation%20Gateway/images/Magma_High-Level_Architecture.png)

* 簡單地說，FEG 是個 proxy：一邊是使用 3GPP 協議（SGs, S6a, 等等），另一邊使用 gRPC。

FEG 和 AGW 之間:

* 兩者都在 Orchestrator 的控制下運行
* FEG 處理與運營商核心的 control plane -- FEG 不承載 User Plane traffic
* AGW 處理與 RAN 的 control plane
* 通常架構中最多有兩個 FEG（for availability）就足以滿足 control plane 的流量負載需求
* 相比之下，需要數十個或更多 AGW 才能支持大規模網絡


### Communication among AGWs and FEGs

* 請注意，Orchestrator 和 FEG 之間以及 Orchestrator 和 AGW 之間存在 gRPC 通訊路線，但 AGW 和 FEG 之間沒有直接連接
	* 因為通常有多個 AGW 和 FEG
	* 而 Orchestrator 得維護必要的 global mapping state，以確定哪個 AGW 或 FEG 應該是通訊目標
* 這也最大限度地減少了進入運營商核心網絡的連接點數量 -- 只有 FEG 需要與 federated mobile core 直接連接
* 每當需要從 AGW 向 FEG 發送訊息，或從 AGW 向 FEG 發送時，訊息都會發送到 Orchestrator 中的中繼服務


### High Availability

* 通常在 active/standby 配置中會有一對 FEG 以提供 high availability (可用性)
* 每個網關向 Orchestrator 報告其 health status
* Health 由 real-time system 和 service metrics 組成
	* 例如: 服務可用性、請求失敗百分比和 CPU 利用率
* Orchestrator 維護所有 Federation Gateway 健康狀況，負責設置哪個 Gateway 處於 active 狀態，並將流量 routing 到該 Gateway
* FEG 不存儲持久狀態，standby gateway 可以在故障轉移期間快速提升為 active 狀態


### Federation Gateway Main Functions

FEG 支持 Magma 與 federated network 中的通訊，包含:

* 使用 S6a interface 的 Home Subscriber Server (HSS)
* 使用 Gx interface 的 Policy & Charging Rules Function (PCRF)
* 使用 Gy interface 的 Online Charging System (OCS)
* 使用 SGs interface 的 Mobile Switching Center/Visitor Location Register (MSC/VLR)

以下部分將更詳細地探討其中的每一個

這些 components 中的每一個，都有一個標準的 3GPP protocol 用於該 component 和 FEG 之間的通訊

![](https://i0.wp.com/yatebts.com/wp-content/uploads/2018/11/LTE_classic_architecture.png)

NOTE: 這個章節要講的是 FEG，要使用 FEG 表示要連接現有的 mobile network 服務，所以下面的 HSS, PCRF..等等都位於 FEG 後面的 Operator Core

### Home Subscriber Server (HSS)

* Home Subscriber Server 保存有關用戶的資訊，MME 會存取它以對 UE 進行身份驗證和授權
* MME 和 HSS 之間的標準 3GPP 介面是 S6a
* 當 UE 嘗試在 Magma 網路中與 AGW 關聯時（ex: 想透過它入網），AGW 的 MME 組件將需要向 federated network 中的 HSS 發出請求
	* 這些請求以 gRPC 發送到 Orchestrator，然後中繼到 FEG
	* FEG 形成必要的 S6a protocol messages 來查詢 HSS
	* response 由 FEG 接收，然後再以 gRPC 回給到 Orchestrator，再中繼回 AGW

雖然看起來很繞，但它確保：

* federated network 的精確細節與 AGW 和 Orchestrator 隔離 -- 只有 FEG 需要詳細了解 federated network 中需實現的 3GPP protocols 和 procedures
* 使用更多 AGW 或更多 FEG 擴展系統，並不會給 AGW 和 FEG 帶來額外負擔


### Policy & Charging Rules Function (PCRF)

* 在 "federated" 部署的情況下，PCRF 可以由 federated network 提供
* 在這種情況下，FEG 透過 3GPP 定義的 Gx 介面，對 PCRF 提供標準介面

與上一節一樣，FEG 需透過 Orchestrator 中繼與 AGW 通訊


### Online Charging System (OCS)

* Online Charging System 是 3GPP 架構的一個元件
* 它追踪用戶的使用情況以進行計費
* 它是 “在線” 的，因為它可以 real time 查詢
	* 用戶是否仍有足夠的信用來繼續發送數據。
* 透過 3GPP 定義的 Gy 介面
* 一樣需透過 Orchestrator 中繼與 AGW 通訊


### Mobile Switching Center/Visitor Location Register (MSC/VLR)

* Visitor Location Register 類似於 HSS，但它儲存漫游到網路的 subscribers info
* VLR 中存了用戶在 home network 中檢索到的訊息
* FEG 充當 Magma 和 VLR 之間的 proxy


## 06-The Network Management System

* 幾乎 Magma 中的每個功能都是透過 Orchestrator 的 REST API 存取的
* NMS 提供了一個位於該 API 之上的 GUI -- 管理者可以與 Magma 互動，無需開發自己的系統來使用 API

目標

* [ ] 了解 NMS 的主要功能。
* [ ] 了解 NMS 如何與 Magma 的其餘部分互動

NMS 提供了一個基本的 OSS/BSS 來操作 Magma 網絡。它支援電信術語說的 FCAPS（故障、配置、計費、性能、安全）


### Rationale for the Network Management System

NOTE: Orchestrator 可以單獨運作一個小型的 Operations Support 和 Billing Support (OSS/BSS)

對於已建立的行動網絡運營商，他們可能需要連到 Magma 的自帶的 OSS/BSS，這可以透過以下組合來實現：

* 使用 Federation Gateway 將現有網絡與 Magma 聯合(Federating)；和
* 使用 Orchestrator 公開的 REST API 將 OSS/BSS 與 Magma 整合

NMS 提供了一個架構在 Orchestrator REST API 之上的 GUI


### Network Management System Overview

Magma NMS 提供了一個 GUI，它能夠管理：

* Organizations
* Networks
* Gateways 和 eNodeBs
* Subscribers
* Policies
* APNs（Access Point Names）

其他特性:

* 提供所有 AGWs 及其 top-level status 的 dashboard
* KPIs（關鍵績效指標）和特定 AGW 的狀態
* 所有用戶及其 top-level status 的 dashboard
* 單個用戶的 KPIs 和服務狀態
* dashboard 上顯示的升級/修復的警報
* 故障修復程序，包括遠程重啟 gateways 的能力
* 遠程觸發 gateways 進行軟體升級的能力
* A way to configure and download call traces


### Federated Operations

* 當 Magma 網絡與現有 mobile network 聯合 (federated) 時，在 federated network 中的配置在 NMS 中是可見的。
	* ex: 可以在現有網路中創建用　subscriber profile，然後在 Magma NMS 中查看

### Dashboards Overview

NMS 的主畫面如下

![](06-The%20Network%20Management%20System/images/Main_Screen_of_NMS.png)

top-level dashboard 提供 summary info：

* alerts 和 events 發生的時間和頻率
* 顯示 “Critical”、“Major”、“Minor” 和 “Misc” alerts
* 所有 gateways 和 eNodeBs 的 KPI 指標
* “健康” 和 “不健康” gateways 的數量
* eNodeBs 的數量，以及當前傳輸的狀態
* Network-wide event table

### Equipment Dashboard

設備儀表板涵蓋了 Magma 部署的基礎設施組件，即接入網關和 eNodeB。對於 AGW 和 eNodeB，儀表板顯示狀態（例如健康狀況）並提供配置和升級相關設備的選項。


### Equipment Dashboard: The Gateway Page

具體來說，網關頁面允許操作員查看：

* “簽入事件”作為每個網關的健康和連接性的可見指示。
* 網關特定的 KPI，包括對特定目的地的 ping 的最大、最小和平均延遲以及健康網關的百分比。
* 每個網關的詳細信息，例如名稱、硬件 ID 等。
* 連接到每個網關的 eNodeB 的數量。
* 連接到特定網關的訂閱者列表。
* 網關記錄的事件的可搜索表和事件頻率的圖形表示。
* 可搜索的日誌消息表和日誌消息頻率的圖形表示。
* 警報表。

下圖顯示了事件頁面上設備儀表板的示例：

![](06-The%20Network%20Management%20System/images/gateway_detail2.png)

網關頁面還允許操作員配置 AGW、添加或刪除 AGW 以及升級其軟件。

### Equipment Dashboard: eNodeB Page

eNodeB 頁面允許操作員查看：

* 通過一組 eNodeB 的流量的總吞吐量。
* eNodeB 的摘要健康狀況。
* 每個 eNodeB 的詳細信息，例如其名稱、硬件 ID、硬件配置等。

還可以添加、編輯和刪除 eNodeB。


### Network Dashboard

在移動核心的上下文中，“網絡”可以定義為 RAN 和 EPC 資源的集合，用於向一組用戶提供網絡服務。
因此，網絡具有一組屬性，例如名稱、MCC/MNC 元組（移動國家代碼/移動網絡代碼）和一組相關資源，
例如 RAN 中分配的帶寬和無線電信道。這些對於所使用的特定無線電接入技術來說是非常特定的。
網絡儀表板允許操作員查看一組已配置的網絡並添加新網絡或編輯現有網絡的屬性。


### Subscriber Dashboard

訂閱者儀表板提供所有訂閱者的可搜索列表、每個訂閱者的一些高級詳細信息以及深入了解單個訂閱者的能力。都支持訂閱者的增刪改查。概覽頁面（取自Magma 文檔）如下所示：

![](06-The%20Network%20Management%20System/images/subscriber_overview1.png)

當操作員深入了解單個訂戶時，他們會看到該訂戶狀態的概覽。這包括訂閱者最近的數據使用情況，如下所示（從Magma 文檔中檢索）：

![](06-The%20Network%20Management%20System/images/stacked_barchart.png)

儀表板還允許操作員查詢與訂閱者相關的事件，例如會話何時開始和終止。

訂閱者儀表板還可用於創建、編輯和刪除訂閱者記錄，以及每個訂閱者的所有相關參數。這些包括：

* 訂戶名稱
* IMSI（訂閱者的唯一標識符）
* 訂閱者的密鑰（Auth Key）
* 訂戶的數據計劃
* 訂閱者的 APN（接入點名稱 - 訂閱者連接的數據網絡）。

如前所述，在聯合環境中，用戶信息來自運營商現有的移動網絡，NMS 僅允許查看而不是創建或編輯該信息。


### Traffic Dashboard

* “traffic” 這個名字有點令人困惑；更準確地應該稱為“Policies Dashboard”。
* 此儀表板用於控制 traffic 的策略（例如通過速限管制 “noisy neighbors”）以及根據服務需求配置 APNs（Access Point Names）。
* 該儀表板允許操作員查看配置的策略和 APN，編輯、建立和刪除它們。

移動網路 context 中的策略包括：QoS 配置文件和 filtering rules（例如，阻止到某些 IP 的流量）。


### Metrics

正如第 3 章所討論的，Magma 被廣泛使用，因此幾乎可以從系統的每個部分收集指標。Magma 網關和 Orchestrator 生成指標以提供對網關、基站、用戶、可靠性、吞吐量等的可見性。這些指標定期推送到 Prometheus 提供的時間序列數據庫中。指標可以通過多種方式查詢和展示，網管為此提供了儀表板。

用於探索和查看指標的儀表板由 Grafana 提供，Grafana 是一個功能齊全的分析平台，允許用戶創建自定義儀表板。Magma 帶有一些內置儀表板，如下圖所示，它顯示了 eNodeB 狀態、連接的用戶以及 AGW 的附加指標。


![](https://github.com/andrewintw/introduction-to-magma/raw/main/06-The%20Network%20Management%20System/images/grafana_variables.png)

Magma 支持的全套指標將隨著時間的推移而增加；當前集記錄在此處。有關 Grafana 的完整文檔以及如何自定義它，請參閱Grafana 文檔。


### Alerts

警報為操作員提供了一種方法，可以通知操作員網絡中需要注意的情況，例如性能問題故障。Magma 提供一組內置警報、一種自定義警報的方法以及將警報定向到其他系統（例如 Slack）的工具，以便可以傳遞適當的通知並採取措施解決問題。NMS 支持這些操作。

預定義的警報包括：

* 網關上的高 CPU 使用率
* UE 的不成功連接嘗試
* 授權或認證失敗
* 引導網關時的異常
* 服務重新啟動。

可以使用上面討論的指標配置自定義警報，並將它們與某些觸發器進行比較，例如可用內存低於閾值、磁盤利用率高於閾值等。


### Call Tracing

調用跟踪是 Magma 提供的一種重要的故障排除工具，可以從 NMS 或 Orchestrator API 啟動。調用跟踪在 AGW 的特定接口上啟動數據包捕獲；然後可以使用 Wireshark 等工具下載和分析捕獲的數據包。通過檢查捕獲的消息交換，可以診斷一系列可能影響連接和使用網絡的訂戶的問題。下圖（取自 Magma 文檔）顯示了 Wireshark 中顯示的跟踪示例。

![](06-The%20Network%20Management%20System/images/example_of_a_trace_displayed_in_Wireshark.png)

Wireshark 中顯示的跟踪示例


~ END ~