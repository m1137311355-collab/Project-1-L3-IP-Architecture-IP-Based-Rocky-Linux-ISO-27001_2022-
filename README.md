基於 L3 IP 的網路架構，整合 Rocky Linux 和 ISO 27001:2022

一個企業級三層路由網路架構，整合了 Rocky Linux 基礎架構服務，並根據ISO 27001:2022資訊安全管理系統框架進行了強化。
旨在消除傳統的二層生成樹協定 (STP) 的限制，提高收斂速度，緩解廣播風暴，並強制執行嚴格的合規性控制。

## 架構概述

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

## 主要特性與技術
  * L3 路由存取與 ECMP：完全路由拓樸至存取層繞過STP收斂限制，啟用等價多路徑（ECMP）路由，並將廣播域隔離到本地子網路。.
  * 高可用性（HSRPv2）：使用 HSRP 群組優先權和搶佔技術，在核心交換器和匯聚交換器之間設定網關冗餘。.
  * 網路服務：集中式 DHCP 中繼（ip helper-address）、用於外部出口的連接埠位址轉換 (PAT) 以及使用 NTP 的同步chrony
  * 儲存基礎架構：透過 Stratis 檔案系統自動匯出 NFS 數據stratis-cli，並動態掛載到 Rocky Linux 用戶端。



## ISO 27001:2022 Security Controls Implementation

| Control ID | ISO 27001:2022 Control Name | Technical Implementation |
| :--- | :--- | :--- |
| **A.5.17** | Authentication Information | Configured Linux account password lifecycle management (`chage -M 60`) for mandatory rotation[cite: 1]. |
| **A.8.2** | Privileged Access Rights | Enforced strict access restrictions on root privileges and administrative functions[cite: 1]. |
| **A.8.5** | Secure Authentication | Disabled direct root SSH logins (`PermitRootLogin no`) and restricted access exclusively to SSHv2[cite: 1]. |
| **A.8.9** | Configuration Management | Minimal service footprint enabled via `firewalld`; disabled unused switch features (`cdp run`, `ip http server`)[cite: 1]. |
| **A.8.15** | Logging | Centralized Syslog infrastructure collecting real-time events across network devices and Linux servers into `/var/log/remote/`[cite: 1]. |
| **A.8.17** | Clock Synchronization | NTP system clock alignment across all nodes via internal Linux Chrony Stratum server[cite: 1]. |
| **A.8.20** | Network Security | Exposure reduction by disabling unneeded legacy protocols (`ip source-route`, HTTP) and hardening interface boundaries[cite: 1]. |
| **A.8.24** | Use of Cryptography | Authenticated OSPF routing adjacencies via `HMAC-SHA-256`, encrypted type-7 passwords, and SSH enforcement[cite: 1]. |


## 驗證與命令
### 1. Centralized Log Management (A.8.15)
# Verify remote logs captured from distribution/core routers
cat /var/log/remote/192.168.30.1/2026-08-08.log | grep OSPF

2. OSPF Neighborhood & Authentication (A.8.24)
# Verify client Stratis NFS mount point
df -h | grep stratis
# Output: 192.168.213.14:/srv/stratis_share mounted on /mnt/client_stratis
  

