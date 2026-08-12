基於 L3 IP 的網路架構，整合 Rocky Linux 和 ISO 27001:2022

一個企業級三層路由網路架構，整合了 Rocky Linux 基礎架構服務，並根據ISO 27001:2022資訊安全管理系統框架進行了強化。
旨在消除傳統的二層生成樹協定 (STP) 的限制，提高收斂速度，緩解廣播風暴，並強制執行嚴格的合規性控制。

架構概述

                      +-------------------+
                      |   Cloud / WAN     |
                      +---------+---------+
                                | 192.168.213.0/24
                       +--------+--------+
                       |    R-1 / R-2    | (Core Routers / HSRP / PAT)
                       +--------+--------+
                                | Port-Channels (OSPF Area 0)
                       +--------+--------+
                       |    D-1 / D-2    | (Distribution Switches)
                       +--------+--------+
                                | Port-Channels (OSPF Area 0)
                       +--------+--------+
                       |    A-1 / A-2    | (Access Switches / L3 Routed)
                       +----+-------+----+
                            |       |
               +------------+       +------------+
               |                                 |
       +-------+-------+                 +-------+-------+
       | Linux-1 / L-3 |                 | Linux-2 / L-4 |
       | 192.168.10.0/24                 | 192.168.20.0/24
       +---------------+                 +---------------+

  * 環境/平台： Cisco IOS、GNS3、Rocky Linux
  * 核心設計：採用乙太網路通道（連接埠通道）中繼的 L3 路由存取架構，執行 OSPF 區域 0

主要特性與技術
  * L3 路由存取與 ECMP：完全路由拓樸至存取層繞過STP收斂限制，啟用等價多路徑（ECMP）路由，並將廣播域隔離到本地子網路。.
  * 高可用性（HSRPv2）：使用 HSRP 群組優先權和搶佔技術，在核心交換器和匯聚交換器之間設定網關冗餘。.
  * 網路服務：集中式 DHCP 中繼（ip helper-address）、用於外部出口的連接埠位址轉換 (PAT) 以及使用 NTP 的同步chrony
  * 儲存基礎架構：透過 Stratis 檔案系統自動匯出 NFS 數據stratis-cli，並動態掛載到 Rocky Linux 用戶端。


ISO 27001:2022 安全控制實施
控制 IDISO 27001:2022 控制名稱技術實施A.5.17身份驗證訊息已配置 Linux 帳戶密碼生命週期管理（chage -M 60）以強制輪換。  A.8.2特權存取權限對根權限和管理功能實施了嚴格的存取限制。  A.8.5安全認證已停用直接 root 使用者 SSH 登入（PermitRootLogin no），並將存取權限限制為僅限 SSHv2。。  A.8.9配置管理透過以下方式啟用最小服務佔用空間firewalld；停用未使用的交換器功能（cdp run，ip http server）。  A.8.15日誌記錄集中式 Syslog 基礎設施收集來自網路設備和 Linux 伺服器的即時事件。/var/log/remote/。  A.8.17時鐘同步透過內部 Linux Chrony Stratum 伺服器對所有節點進行 NTP 系統時鐘校準。  A.8.20網路安全透過停用不必要的舊版協定（ip source-route例如 HTTP）和加強介面邊界來降低風險暴露。。  A.8.24密碼學的應用透過驗證的 OSPF 路由鄰接關係HMAC-SHA-256、加密的 7 型密碼和 SSH 強制執行。


驗證與命令
1. 集中式日誌管理 (A.8.15)
  # Verify remote logs captured from distribution/core routers
  cat /var/log/remote/192.168.30.1/2026-08-08.log | grep OSPF
2. OSPF 鄰域和認證 (A.8.24)
  D-2# show ip ospf neighbor
  Neighbor ID     Pri   State           Dead Time   Address         Interface
  192.168.213.12    1   FULL/DR         00:00:38    192.168.30.81   Port-channel10   
3. Linux Stratis 檔案系統自動掛載
  # Verify client Stratis NFS mount point
  df -h | grep stratis
  # Output: 192.168.213.14:/srv/stratis_share mounted on /mnt/client_stratis
4. 



