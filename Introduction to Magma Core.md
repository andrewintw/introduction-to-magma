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
