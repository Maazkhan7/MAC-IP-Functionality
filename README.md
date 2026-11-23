<div align="center">

# 🔀 MAC & IP Functionality
### How a Frame Travels Across a Router — ARP, MAC Table & Layer 2 Rewriting

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Topic](https://img.shields.io/badge/Topic-ARP%20%7C%20MAC%20%7C%20IP-blue?style=for-the-badge)
![Layer](https://img.shields.io/badge/OSI-Layer%202%20%2B%20Layer%203-orange?style=for-the-badge)
![Mode](https://img.shields.io/badge/Mode-Simulation-green?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20Fundamentals-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

### 🔑 IP stays the same end-to-end. MAC changes at every hop.

![MAC and IP Functionality](images/MAC-IP.png)

🔵 **Blue = STEP-1 (Request)** &nbsp;&nbsp;|&nbsp;&nbsp; 🔴 **Red = STEP-2 (Reply)**

</div>

---

## 📖 Table of Contents

1. [Overview](#-overview)
2. [Lab Objectives](#-lab-objectives)
3. [Topology](#-topology)
4. [Addressing Table](#-addressing-table)
5. [Router Configuration](#-router-configuration)
6. [Theory Refresher](#-theory-refresher)
7. [STEP-1: Request (Blue)](#-step-1--request-blue)
8. [STEP-2: Reply (Red)](#-step-2--reply-red)
9. [Complete Frame Summary](#-complete-frame-summary)
10. [How ARP Fits In](#-how-arp-fits-in)
11. [MAC Address Table Behaviour](#-mac-address-table-behaviour)
12. [Is MPLS Involved?](#-is-mpls-involved)
13. [Verification Commands](#-verification-commands)
14. [Troubleshooting Notes](#-troubleshooting-notes)
15. [Key Takeaways](#-key-takeaways)
16. [Author](#-author)

---

## 🎯 Overview

In **Packet Tracer Simulation Mode** I followed an ICMP ping from **PC0 (LAN 1)** to **PC2 (LAN 2)** through **Router0** and inspected the PDU of every frame.

The lab proves the most important idea in IP networking:

> **The IP header (Layer 3) never changes from source to destination, but the Ethernet header (Layer 2) is rewritten by the router at every hop.**

This lab extends the earlier ARP lab (same-subnet communication) to **inter-network communication**.

**PDU field legend (as used in the diagram):**

| Short | Meaning |
|---|---|
| **SIP** | Source IP |
| **DIP** | Destination IP |
| **SM** | Source MAC |
| **DM** | Destination MAC |

---

## ✅ Lab Objectives

- [x] Understand why a host sends traffic to its **default gateway MAC** for remote networks
- [x] Observe **SIP / DIP** staying constant
- [x] Observe **SM / DM** changing at the router
- [x] Understand how the router uses its **routing table** and **ARP**
- [x] Understand how a switch builds its **MAC address table**
- [x] Confirm that MPLS (Layer 2.5) is **not** involved

---

## 🗺️ Topology

```
   192.168.1.0/24                                      200.100.100.0/24

 PC0 ─────┐                                                   ┌───── PC2
 .2       │                                                   │      .2
          ├── Switch0 ───── [ Router0 (2901) ] ───── Switch1 ┤
 PC1 ─────┘    2950-24      .1 │          │ .1       2950-24   │
 .3                           LAN-1      LAN-2                 └───── PC3
                                                                      .3
```

| Segment | Network | Gateway |
|---|---|---|
| LAN 1 (Switch0) | `192.168.1.0/24` | `192.168.1.1` |
| LAN 2 (Switch1) | `200.100.100.0/24` | `200.100.100.1` |

---

## 📋 Addressing Table

| Device | Interface | IP Address | Subnet Mask | MAC Address |
|---|---|---|---|---|
| PC0 | NIC | 192.168.1.2 | 255.255.255.0 | `0005.5EA8.D53D` |
| PC1 | NIC | 192.168.1.3 | 255.255.255.0 | `0004.9A33.87CC` |
| Router0 | LAN-1 side | 192.168.1.1 | 255.255.255.0 | `0060.3E9E.2801` |
| Router0 | LAN-2 side | 200.100.100.1 | 255.255.255.0 | `0060.3E9E.2802` |
| PC2 | NIC | 200.100.100.2 | 255.255.255.0 | `000C.CF52.A56A` |
| PC3 | NIC | 200.100.100.3 | 255.255.255.0 | `0002.4AC2.2299` |
| Switch0 | — | — | — | `00E0.F93E.17C2` |
| Switch1 | — | — | — | `00D0.FF8D.2E18` |

---

## ⚙️ Router Configuration

```
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip address 200.100.100.1 255.255.255.0
Router(config-if)# no shutdown
```

> Both networks are **directly connected**, so no static route or routing protocol is required.

---

## 📚 Theory Refresher

| Concept | What it does |
|---|---|
| **IP address (L3)** | Logical, end-to-end. Does **not** change on the way. |
| **MAC address (L2)** | Physical, hop-by-hop. Rewritten at every Layer 3 device. |
| **ARP** | Resolves *IP → MAC* inside one broadcast domain. |
| **Default gateway** | Router interface used when the destination is in another network. |
| **Switch** | Forwards by the **MAC table**; never looks at IP. |
| **Router** | Looks up the **routing table**, picks the out-interface, rewrites the L2 header. |

---

## 🔵 STEP-1 — Request (Blue)

**PC0 (192.168.1.2) → PC2 (200.100.100.2)**

### Frame 1️⃣ — PC0 → Router0

PC0 sees that `200.100.100.2` is **not** in its own subnet, so it sends the frame to its **default gateway** (it ARPs for `192.168.1.1`, not for PC2).

| Field | Value | Who |
|---|---|---|
| **SIP** | `192.168.1.2` | PC0 |
| **DIP** | `200.100.100.2` | PC2 (final target) |
| **SM** | `0005.5EA8.D53D` | PC0 |
| **DM** | `0060.3E9E.2801` | Router0 LAN-1 interface |

📝 **Annotation:** DIP is PC2 but DM is the **router**. This is the Layer 2 vs Layer 3 difference.

### Frame 2️⃣ — Router0 → PC2

Router0 checks its **routing table**, finds `200.100.100.0/24` directly connected, and selects the LAN-2 interface. If PC2's MAC is not in its ARP cache, it sends an **ARP Request (broadcast `FFFF.FFFF.FFFF`)** and PC2 answers with a **unicast ARP Reply**. Then the router **rewrites the L2 header**.

| Field | Value | Who |
|---|---|---|
| **SIP** | `192.168.1.2` | unchanged ✅ |
| **DIP** | `200.100.100.2` | unchanged ✅ |
| **SM** | `0060.3E9E.2802` | Router0 LAN-2 interface (**new**) |
| **DM** | `000C.CF52.A56A` | PC2 (**new**) |

---

## 🔴 STEP-2 — Reply (Red)

**PC2 (200.100.100.2) → PC0 (192.168.1.2)**

### Frame 3️⃣ — PC2 → Router0

PC0 is in another network, so PC2 sends the reply to **its own** default gateway.

| Field | Value | Who |
|---|---|---|
| **SIP** | `200.100.100.2` | PC2 |
| **DIP** | `192.168.1.2` | PC0 |
| **SM** | `000C.CF52.A56A` | PC2 |
| **DM** | `0060.3E9E.2802` | Router0 LAN-2 interface |

### Frame 4️⃣ — Router0 → PC0

The router rewrites the L2 header again, this time for LAN 1.

| Field | Value | Who |
|---|---|---|
| **SIP** | `200.100.100.2` | unchanged ✅ |
| **DIP** | `192.168.1.2` | unchanged ✅ |
| **SM** | `0060.3E9E.2801` | Router0 LAN-1 interface |
| **DM** | `0005.5EA8.D53D` | PC0 |

---

## 📊 Complete Frame Summary

| Step | # | Direction | SIP | DIP | SM | DM |
|:-:|:-:|---|---|---|---|---|
| 🔵 1 | 1 | PC0 → Router0 | 192.168.1.2 | 200.100.100.2 | `0005.5EA8.D53D` | `0060.3E9E.2801` |
| 🔵 1 | 2 | Router0 → PC2 | 192.168.1.2 | 200.100.100.2 | `0060.3E9E.2802` | `000C.CF52.A56A` |
| 🔴 2 | 3 | PC2 → Router0 | 200.100.100.2 | 192.168.1.2 | `000C.CF52.A56A` | `0060.3E9E.2802` |
| 🔴 2 | 4 | Router0 → PC0 | 200.100.100.2 | 192.168.1.2 | `0060.3E9E.2801` | `0005.5EA8.D53D` |

```
 L3 (IP):   192.168.1.2 ═════════════════════════════════► 200.100.100.2    (never changes)

 L2 (MAC):  D53D ──► 2801 │ router │ 2802 ──► A56A                          (rewritten at router)
```

---

## 🔍 How ARP Fits In

| Who ARPs | For which IP | Why |
|---|---|---|
| PC0 | `192.168.1.1` (gateway) | Destination is remote, so it needs the **gateway's** MAC |
| Router0 | `200.100.100.2` (PC2) | PC2 is directly connected, so it needs PC2's MAC |
| PC2 | `200.100.100.1` (gateway) | Reply goes to the gateway |
| Router0 | `192.168.1.2` (PC0) | Needs PC0's MAC for final delivery (usually cached already) |

> ⚠️ A router does **not** check "does this IP exist". It checks its **routing table** for the destination **network**, then uses ARP only to find the next-hop MAC.

---

## 🧠 MAC Address Table Behaviour

A switch **learns from the source MAC** and **forwards by the destination MAC**.

| Case | Switch action |
|---|---|
| **Known unicast** | Forward only to the matching port |
| **Unknown unicast** | **Flood** to all ports except the incoming one |
| **Broadcast** (`FFFF.FFFF.FFFF`) | Always flood (e.g. ARP Request) |

Example on **Switch0**:

1. Table is empty at start.
2. PC0 ARP broadcast → Switch0 learns `0005.5EA8.D53D → PC0 port`, floods the frame.
3. Router replies (unicast) → Switch0 learns `0060.3E9E.2801 → router port`.
4. Later PC0 → Router traffic goes out a single port, no flooding.

> ⏱️ Default aging time: **300 seconds**
> 💡 Switch0 only ever sees the **router's** MAC for remote hosts, never PC2's MAC, because the router rewrites the L2 header.

---

## ❓ Is MPLS Involved?

**No.** This lab is plain Ethernet switching plus IP routing.

| | This lab | MPLS |
|---|---|---|
| Header between L2 and L3 | none | **MPLS label (shim)**, hence "Layer 2.5" |
| Forwarding decision | IP lookup at the router | Label lookup |
| Needs | nothing extra | MPLS / LDP enabled |

The PDUs contain only Ethernet and IP headers, and there is a single router, so no label is used.

---

## ✅ Verification Commands

> 📸 Add your own outputs / screenshots under each command.

**PCs:**
```
C:\> ipconfig /all
C:\> arp -a
C:\> ping 200.100.100.2
```

**Switches:**
```
Switch# show mac address-table
Switch# clear mac address-table dynamic
```

**Router:**
```
Router# show ip interface brief
Router# show ip route
Router# show arp
```

| Check | Expected result |
|---|---|
| `show ip interface brief` | Both interfaces **up/up** with correct IPs |
| `show ip route` | `C` routes for 192.168.1.0/24 and 200.100.100.0/24 |
| `show arp` (router) | Entries for PC0 and PC2 after the ping |
| `show mac address-table` (Switch0) | PC0, PC1 and router-port MACs, type DYNAMIC |
| `ping` PC0 → PC2 | **Success** |

---

## 🛠️ Troubleshooting Notes

| Problem | Likely cause | Fix |
|---|---|---|
| Ping **Failed** | Wrong router IP or wrong PC gateway | Router LAN-1 IP must be `192.168.1.1` and match the PC gateway |
| First ping drops | ARP still resolving | Ping again |
| Interface down | Not enabled | `no shutdown` |

---

## 🎓 Key Takeaways

1. **IP is end-to-end, MAC is hop-by-hop.**
2. For a remote destination, the host uses the **gateway's MAC** as DM.
3. The router uses the **routing table** to pick the out-interface, then **ARP** for the next MAC.
4. The router **rewrites the Layer 2 header** on every hop; Layer 3 is untouched (except TTL).
5. A switch **learns from source MAC**, **forwards by destination MAC**, and floods unknown unicast and broadcast.
6. **MPLS (Layer 2.5)** is not part of normal routing.

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

⭐ If this lab helped you, give the repo a star!
