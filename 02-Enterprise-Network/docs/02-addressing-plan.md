# 02: IP Addressing Plan

## 1. Addressing conventions (every user /24)

| Range | Used for |
|---|---|
| `.1` | HSRP virtual IP (the default gateway hosts use) |
| `.2` | Active distribution switch SVI (MLS1 / MLS3) |
| `.3` | Standby distribution switch SVI (MLS2 / MLS4) |
| `.4`–`.9` | Reserved |
| `.10` | Department network printer (static) |
| `.11`–`.254` | DHCP pool for PCs |

`.1`–`.10` are excluded from DHCP on the routers.

## 2. VLAN table

| VLAN | Name | HQ subnet | Branch VLAN | Branch subnet |
|---|---|---|---|---|
| 10 / 40 | HR | 192.168.10.0/24 | 40 | 192.168.40.0/24 |
| 20 / 50 | SALES | 192.168.20.0/24 | 50 | 192.168.50.0/24 |
| 30 / 60 | FINANCE | 192.168.30.0/24 | 60 | 192.168.60.0/24 |
| 99 | MANAGEMENT | 192.168.99.0/24 | 99 | 192.168.199.0/24 |
| 999 | NATIVE_UNUSED | n/a | 999 | n/a |

## 3. Point-to-point links

| Link | Network | Side A | IP | Side B | IP |
|---|---|---|---|---|---|
| HQ WAN | 10.10.10.0/30 | HQ-R1 Se0/0/0 (DCE) | 10.10.10.1 | BR-R2 Se0/0/0 | 10.10.10.2 |
| HQ uplink 1 | 10.0.0.0/30 | HQ-MLS1 G0/2 | 10.0.0.1 | HQ-R1 G0/1 | 10.0.0.2 |
| HQ uplink 2 | 10.0.0.4/30 | HQ-MLS2 G0/1 | 10.0.0.5 | HQ-R1 G0/0 | 10.0.0.6 |
| BR uplink 1 | 10.0.1.0/30 | BR-MLS3 G0/1 | 10.0.1.1 | BR-R2 G0/1 | 10.0.1.2 |
| BR uplink 2 | 10.0.1.4/30 | BR-MLS4 G0/1 | 10.0.1.5 | BR-R2 G0/0 | 10.0.1.6 |

## 4. Device interface table

### Routers

| Device | Interface | IP / Mask | Description | OSPF router-ID |
|---|---|---|---|---|
| HQ-R1 | G0/1 | 10.0.0.2 /30 | to HQ-MLS1 | 3.3.3.3 |
| HQ-R1 | G0/0 | 10.0.0.6 /30 | to HQ-MLS2 | |
| HQ-R1 | Se0/0/0 | 10.10.10.1 /30 | WAN to BR-R2 (clock rate 64000) | |
| BR-R2 | G0/1 | 10.0.1.2 /30 | to BR-MLS3 | 4.4.4.4 |
| BR-R2 | G0/0 | 10.0.1.6 /30 | to BR-MLS4 | |
| BR-R2 | Se0/0/0 | 10.10.10.2 /30 | WAN to HQ-R1 | |

### Distribution (multilayer) switches

| Device | Interface | IP | HSRP group / VIP | HSRP priority | Helper |
|---|---|---|---|---|---|
| HQ-MLS1 (RID 1.1.1.1) | VLAN 10 | 192.168.10.2 | 10 / 192.168.10.1 | 110 (active) | 10.0.0.2 |
| | VLAN 20 | 192.168.20.2 | 20 / 192.168.20.1 | 110 | 10.0.0.2 |
| | VLAN 30 | 192.168.30.2 | 30 / 192.168.30.1 | 110 | 10.0.0.2 |
| | VLAN 99 | 192.168.99.2 | 99 / 192.168.99.1 | 110 | n/a |
| | G0/2 | 10.0.0.1 /30 | n/a | n/a | n/a |
| HQ-MLS2 (RID 2.2.2.2) | VLAN 10 | 192.168.10.3 | 10 / 192.168.10.1 | 100 (standby) | 10.0.0.6 |
| | VLAN 20 | 192.168.20.3 | 20 / 192.168.20.1 | 100 | 10.0.0.6 |
| | VLAN 30 | 192.168.30.3 | 30 / 192.168.30.1 | 100 | 10.0.0.6 |
| | VLAN 99 | 192.168.99.3 | 99 / 192.168.99.1 | 100 | n/a |
| | G0/1 | 10.0.0.5 /30 | n/a | n/a | n/a |
| BR-MLS3 (RID 5.5.5.5) | VLAN 40 | 192.168.40.2 | 40 / 192.168.40.1 | 110 (active) | 10.0.1.2 |
| | VLAN 50 | 192.168.50.2 | 50 / 192.168.50.1 | 110 | 10.0.1.2 |
| | VLAN 60 | 192.168.60.2 | 60 / 192.168.60.1 | 110 | 10.0.1.2 |
| | VLAN 99 | 192.168.199.2 | 99 / 192.168.199.1 | 110 | n/a |
| | G0/1 | 10.0.1.1 /30 | n/a | n/a | n/a |
| BR-MLS4 (RID 6.6.6.6) | VLAN 40 | 192.168.40.3 | 40 / 192.168.40.1 | 100 (standby) | 10.0.1.6 |
| | VLAN 50 | 192.168.50.3 | 50 / 192.168.50.1 | 100 | 10.0.1.6 |
| | VLAN 60 | 192.168.60.3 | 60 / 192.168.60.1 | 100 | 10.0.1.6 |
| | VLAN 99 | 192.168.199.3 | 99 / 192.168.199.1 | 100 | n/a |
| | G0/1 | 10.0.1.5 /30 | n/a | n/a | n/a |

### Access switches (management only)

| Device | VLAN 99 IP | Default gateway | User VLAN on Fa0/3-24 |
|---|---|---|---|
| HQ-HR-SW | 192.168.99.11 | 192.168.99.1 | 10 |
| HQ-SALES-SW | 192.168.99.12 | 192.168.99.1 | 20 |
| HQ-FIN-SW | 192.168.99.13 | 192.168.99.1 | 30 |
| BR-HR-SW | 192.168.199.11 | 192.168.199.1 | 40 |
| BR-SALES-SW | 192.168.199.12 | 192.168.199.1 | 50 |
| BR-FIN-SW | 192.168.199.13 | 192.168.199.1 | 60 |

### End devices

| Device | Addressing | Gateway | DNS |
|---|---|---|---|
| All PCs | DHCP (`.11` onward) | HSRP VIP `.1` (from DHCP) | 8.8.8.8 (from DHCP) |
| HQ HR printer | 192.168.10.10 /24 static | 192.168.10.1 | n/a |
| HQ Sales printer | 192.168.20.10 /24 static | 192.168.20.1 | n/a |
| HQ Finance printer | 192.168.30.10 /24 static | 192.168.30.1 | n/a |
| Branch HR printer | 192.168.40.10 /24 static | 192.168.40.1 | n/a |
| Branch Sales printer | 192.168.50.10 /24 static | 192.168.50.1 | n/a |
| Branch Finance printer | 192.168.60.10 /24 static | 192.168.60.1 | n/a |

## 5. Credentials (lab only)

| Item | Value |
|---|---|
| Console password | `cisco` |
| Enable secret | `class` |
| SSH user | `admin` / `Admin123` (privilege 15) |

> These are throwaway lab credentials that are fine to publish. Never put real passwords in a public repo.

