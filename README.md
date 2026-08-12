L3 IP-Based Network Architecture with Rocky Linux & ISO 27001:2022 Integration
An enterprise-grade Layer 3 routed network architecture integrated with Rocky Linux infrastructure services and hardened under the ISO 27001:2022 Information Security Management System framework

Designed to eliminate traditional Layer 2 Spanning Tree Protocol (STP) limits, improve convergence, mitigate broadcast storms, and enforce strict compliance controls

Architecture Overview
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

Key Features & Technologies
  * L3 Routed Access & ECMP: Fully routed topology down to access layers. Bypasses STP convergence limits, enables Equal-Cost Multi-Path (ECMP) routing, and isolates broadcast domains to local subnets
  * High Availability (HSRPv2): Gateway redundancy configured across core and distribution switches using HSRP group prioritization and preemption
  * etwork Services: Centralized DHCP relay (ip helper-address), Port Address Translation (PAT) for external egress, and NTP synchronization using
