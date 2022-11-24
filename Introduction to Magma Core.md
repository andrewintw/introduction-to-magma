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


===


### Federation Gateway (FEG)

### Network Management System (NMS)

![](02-Introduction%20to%20Magma%20Architecture/images/Magma-arch@2x.png)