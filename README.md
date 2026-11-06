# 🔥 FortiGate Enterprise College Campus Network

> A hands-on enterprise network lab built in GNS3 using FortiGate firewalls: **VLAN, DHCP & Site-to-Site IPsec VPN Configuration** between the Surat and Ahmedabad campuses.

## 🏗️ Project Architecture

```text
        ┌──────────────┐                                   ┌──────────────┐
        │  IshanISP    │                                   │  HathwayISP  │
        └──────┬───────┘                                   └──────┬───────┘
               │ Port1 (DHCP) 192.168.42.57                       │ Port1 (DHCP) 192.168.42.225
        ┌──────▼───────┐        VPN TUNNEL (IPsec)         ┌──────▼───────┐
        │  SURAT-FGT1  │◄─────────────────────────────────►│ AHEMDABAD-   │
        │   FortiGate  │                                   │    FGT2      │
        └──┬────────┬──┘                                   └──┬────────┬──┘
   Port4   │        │ Port2                          Port2    │        │ Port4
   (MGMT)  │        │ 172.30.20.1/24             172.30.21.1/24        │ (MGMT)
           │        │                                         │        │
     ┌─────▼──┐  ┌──▼───────────┐                 ┌───────────▼──┐  ┌──▼─────┐
     │MGMT-SW │  │  SURAT_SW1   │                 │AHEMDABAD_SW1 │  │MGMT-SW1│
     └────────┘  └──┬───┬────┬──┘                 └──┬───┬────┬──┘  └────────┘
                    │   │    │                       │   │    │
                  PC2  Dean Lecturer              Finance PC4  PC3
              .20.81         .20.82
```

## 🌍 Site Overview

| Site | Firewall | ISP | LAN Network |
|------|----------|-----|-------------|
| Surat | Surat_FGT1 | IshanISP | 172.30.20.0/24 |
| Ahmedabad | Ahemdabad_FGT2 | HathwayISP | 172.30.21.0/24 |

## 🧭 Addressing Table

| Device | Interface | Address |
|--------|-----------|---------|
| Surat_FGT1 | Port1 (WAN, DHCP) | 192.168.42.57 |
| Surat_FGT1 | Port2 (LAN) | 172.30.20.1/24 |
| Surat_FGT1 | Port4 (MGMT, DHCP) | 192.168.230.137 |
| Ahemdabad_FGT2 | Port1 (WAN, DHCP) | 192.168.42.225 |
| Ahemdabad_FGT2 | Port2 (LAN) | 172.30.21.1/24 |
| Ahemdabad_FGT2 | Port4 (MGMT, DHCP) | 192.168.230.138 |
| PC2 (Surat) | NIC | 172.30.20.81/24 |
| Lecturer (Surat) | NIC | 172.30.20.82/24 |
| PC4 (Ahemdabad) | NIC | 172.30.21.81/24 |
| PC3 (Ahemdabad) | NIC | 172.30.21.82/24 |

## 🎯 Project Objectives

- Configure FortiGate interfaces (WAN, LAN, MGMT)
- Configure static routing to the internet
- Configure DNS
- Configure firewall policies with NAT and logging
- Provide DHCP to LAN devices
- Build a Site-to-Site IPsec VPN between Surat and Ahmedabad

---

# 🏢 Surat-FGT1 Configuration

## 🔌 Interface Configuration

```text
config system global
set hostname Surat_FGT1
end

config system interface
edit port1

set mode dhcp
set allowaccess ping ssh https
set role wan
set alias ISHAN_SURAT_WAN
next

edit port2
set mode static
set ip 172.30.20.1/24
set role lan
set alias surat_lan
set allowaccess ping https ssh
set description "CONNECTED TO ISHAN- HEAD OFFICE"
next

edit port4
set mode dhcp
set alias Surat_MGMT

set allowaccess ping https http
end

show system interface
```

> **Note:** To get DHCP IPs on the LAN devices, enable the DHCP server on the Surat_LAN network.

**On the VPCS devices, run these commands to get a DHCP IP:**

```text
dhcp -d
show ip
```

## 🛣️ Static Routes

I created a route between the Surat_LAN network and the Internet on the Surat firewall.

- Destination subnet: `0.0.0.0/0.0.0.0`
- Interface: `ISHAN_Surat_WAN (port1)`
- Gateway Address: Dynamic, which I found to be `192.168.42.1`
- Administrative distance: same as before
- Comments: `Ishan`
- Click **OK**

## 🌐 DNS Configuration

- Select **Specify** DNS servers
- Primary DNS Server: `8.8.8.8`
- Secondary DNS Server: `1.1.1.1`
- Click **Apply**

## 🛡️ Firewall Policy Configuration

- Name: `Surat_Lan to Internet`
- Incoming interface: `surat_lan (port2)`
- Outgoing interface: `ISHAN_SURAT_WAN (port1)`
- Source: `all`
- Destination: `all`
- Schedule: `always` (default)
- Service: `ALL`
- NAT: **enabled**
- Logging Options: enable **Log Allowed Traffic** for **All Sessions**
- Enable **Generate Logs when Session Starts**
- Click **OK**


## 🔐 Site-to-Site IPsec VPN

> The VPN is created with the **VPN Wizard** on each FortiGate (**VPN → IPsec Wizard**), using the **Site to Site - FortiGate** template.

### 🏢 Surat-FGT1 VPN Configuration

### 1️⃣ Tunnel Template and Network

| Setting | Value |
|---------|-------|
| Tunnel Template | Site to Site - FortiGate |
| Name | `Surat_VPN` |
| Comments | VPN: Surat_VPN (Created by VPN wizard) |
| IP Version | IPv4 |
| Remote Gateway IP Address | `192.168.42.225` (Ahmedabad WAN) |
| Outgoing Interface | `ISHAN_SURAT_WAN (port1)` |
| Dead Peer Detection | On Demand |
| DPD retry count | `3` |
| DPD retry interval | `20` seconds |
| Forward Error Correction | Egress and Ingress both off |



### 2️⃣ Authentication

| Setting | Value |
|---------|-------|
| Remote Gateway | Static IP Address (`192.168.42.225`) |
| Outgoing Interface | `port1` |
| Method | Pre-shared Key |
| Pre-shared Key | Same key on both firewalls (not stored in this repository) |

### 3️⃣ Phase 2 Selectors

| Name | Local Address | Remote Address |
|------|---------------|----------------|
| `Surat_VPN` | `Surat_VPN_local` | `Surat_VPN_remote` |


# 🏢 Ahemdabad-FGT2 Configuration

## 🔌 Interface Configuration

```text
config system global
set hostname Ahemdabad_FGT2
end

config system interface
edit port1

set mode dhcp
set allowaccess ping ssh https
set role wan
set alias Hathway_AHEMDABAD_WAN
next

edit port2
set mode static
set ip 172.30.20.1/24
set role lan
set alias Ahemadabad_lan
set allowaccess ping https ssh
set description "CONNECTED TO ISHAN- HEAD OFFICE"
next

edit port4
set mode dhcp
set alias Ahemadabad_MGMT

set allowaccess ping https http
end

show system interface
```

> **Note:** To get DHCP IPs on the LAN devices, enable the DHCP server on the Ahemdabad_lan network.

**On the VPCS devices, run these commands to get a DHCP IP:**

```text
dhcp -d
show ip
```

## 🛣️ Static Routes

I created a route between the Ahmedabad_LAN network and the Internet on the Ahmedabad firewall.

- Destination subnet: `0.0.0.0/0.0.0.0`
- Interface: `Hathway_AHEMDABAD_WAN (port1)`
- Gateway Address: Dynamic, which I found to be `192.168.42.1`
- Administrative distance: same as before
- Comments: `Hathway`
- Click **OK**

## 🌐 DNS Configuration

- Select **Specify** DNS servers
- Primary DNS Server: `8.8.8.8`
- Secondary DNS Server: `1.1.1.1`
- Click **Apply**

## 🛡️ Firewall Policy Configuration

- Name: `Ahmedabad_LAN to Internet`
- Incoming interface: `Ahmedabad_LAN (port2)`
- Outgoing interface: `Hathway_AHEMDABAD_WAN (port1)`
- Source: `all`
- Destination: `all`
- Schedule: `always` (default)
- Service: `ALL`
- NAT: **enabled**
- Logging Options: enable **Log Allowed Traffic** for **All Sessions**
- Enable **Generate Logs when Session Starts**
- Click **OK**

---
## 🏢 Ahemdabad-FGT2 VPN Configuration

### 1️⃣ Tunnel Template and Network

| Setting | Value |
|---------|-------|
| Tunnel Template | Site to Site - FortiGate |
| Name | `Ahemdabad_VPN` |
| Comments | VPN: Ahemdabad_VPN (Created by VPN wizard) |
| IP Version | IPv4 |
| Remote Gateway IP Address | `192.168.42.57` (Surat WAN) |
| Outgoing Interface | `Hathway_AHEMDABAD_WAN (port1)` |
| Dead Peer Detection | On Demand |
| DPD retry count | `3` |
| DPD retry interval | `20` seconds |
| Forward Error Correction | Egress and Ingress both off |



### 2️⃣ Authentication

| Setting | Value |
|---------|-------|
| Remote Gateway | Static IP Address (`192.168.42.57`) |
| Outgoing Interface | `port1` |
| Method | Pre-shared Key |
| Pre-shared Key | Same key as Surat_FGT1 (not stored in this repository) |

### 3️⃣ Phase 2 Selectors

| Name | Local Address | Remote Address |
|------|---------------|----------------|
| `Ahemdabad_VPN` | `Ahemdabad_VPN_local` | `Ahemdabad_VPN_remote` |



## 🌐 Networking Technologies

- IPv4 & Subnetting
- DHCP
- DNS
- NAT
- Static Routing
- Firewall Policies
- IPsec VPN
- Logging

## 🧰 Tools Used

- GNS3
- FortiGate VM (FortiOS)
- VPCS
- Cisco Switches (SURAT_SW1, AHEMDABAD_SW1)


