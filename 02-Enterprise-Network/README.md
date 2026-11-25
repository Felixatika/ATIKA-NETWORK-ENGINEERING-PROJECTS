<p align="center">
  <img src="Packet-Tracer/enterprise-network-banner.png"
       alt="ATIKA Network Engineering"
       width="100%">
</p>
# Two-Building Enterprise Network: Cisco Packet Tracer

A redundant, segmented enterprise network for a company with a **Headquarters (Building 1)** and a **Branch Office (Building 2)**, each hosting **HR, Sales and Finance** users. Built and tested in Cisco Packet Tracer.

![Topology](topology/topology.png)

---

## Highlights

| Area | What's implemented |
|---|---|
| **Design** | Three-tier hierarchy (core/edge router → distribution multilayer switches → access switches) |
| **Segmentation** | Separate VLAN per department, plus a management VLAN and an unused native VLAN |
| **Inter-VLAN routing** | SVIs on Cisco 3560 multilayer switches (`ip routing`) |
| **Gateway redundancy** | HSRP: MLS1/MLS3 active, MLS2/MLS4 standby, with preemption |
| **Loop prevention** | Rapid PVST+, root bridge pinned to the active distribution switch |
| **Dynamic routing** | OSPF (single area 0) across both buildings |
| **WAN** | Point-to-point serial link between HQ and Branch routers |
| **IP addressing** | DHCP pools on each edge router, relayed with `ip helper-address` |
| **Security** | SSH v2 only, local AAA user, port security (sticky MAC), BPDU Guard, unused ports shut, encrypted passwords, MOTD banner |

---

## Repository layout

```
.
├── README.md                     ← you are here
├── topology/topology.png         ← network diagram
├── docs/
│   ├── 01-network-design.md      ← requirements, device inventory, naming, cabling
│   ├── 02-addressing-plan.md     ← VLANs, subnets, every interface IP
│   ├── 03-implementation-guide.md← step-by-step build, phase by phase
│   ├── 04-verification-testing.md← show commands, expected results, test matrix
│   └── 05-troubleshooting.md     ← common faults and fixes
├── configs/
│   ├── hq/      ← HQ-R1, HQ-MLS1, HQ-MLS2, HQ-HR-SW, HQ-SALES-SW, HQ-FIN-SW
│   └── branch/  ← BR-R2, BR-MLS3, BR-MLS4, BR-HR-SW, BR-SALES-SW, BR-FIN-SW
├── packet-tracer/                ← the .pkt project file
├── screenshots/                  ← verification evidence
├── linkedin/post.md              ← LinkedIn post + project entry
└── PUBLISHING.md                 ← GitHub + LinkedIn publishing checklist
```

---

## Network at a glance

### Devices (16 network devices, ~42 end hosts)

| Building | Role | Model | Hostname |
|---|---|---|---|
| HQ | Edge router | Cisco 2901 (+HWIC-2T) | `HQ-R1` |
| HQ | Distribution (active) | Cisco 3560-24PS | `HQ-MLS1` |
| HQ | Distribution (standby) | Cisco 3560-24PS | `HQ-MLS2` |
| HQ | Access: HR / Sales / Finance | Cisco 2960-24TT ×3 | `HQ-HR-SW`, `HQ-SALES-SW`, `HQ-FIN-SW` |
| Branch | Edge router | Cisco 2901 (+HWIC-2T) | `BR-R2` |
| Branch | Distribution (active) | Cisco 3560-24PS | `BR-MLS3` |
| Branch | Distribution (standby) | Cisco 3560-24PS | `BR-MLS4` |
| Branch | Access: HR / Sales / Finance | Cisco 2960-24TT ×3 | `BR-HR-SW`, `BR-SALES-SW`, `BR-FIN-SW` |

Each access switch serves one department: about 6 PCs and 1 network printer.

### VLANs and subnets

| Building | VLAN | Name | Subnet | Gateway (HSRP VIP) |
|---|---|---|---|---|
| HQ | 10 | HR | 192.168.10.0/24 | 192.168.10.1 |
| HQ | 20 | SALES | 192.168.20.0/24 | 192.168.20.1 |
| HQ | 30 | FINANCE | 192.168.30.0/24 | 192.168.30.1 |
| HQ | 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 |
| Branch | 40 | HR | 192.168.40.0/24 | 192.168.40.1 |
| Branch | 50 | SALES | 192.168.50.0/24 | 192.168.50.1 |
| Branch | 60 | FINANCE | 192.168.60.0/24 | 192.168.60.1 |
| Branch | 99 | MANAGEMENT | 192.168.199.0/24 | 192.168.199.1 |
| Both | 999 | NATIVE_UNUSED | n/a (trunk native / parking VLAN) | n/a |

The full interface-by-interface plan is in [docs/02-addressing-plan.md](docs/02-addressing-plan.md).

---

## How to run it

1. Install **Cisco Packet Tracer 8.x** (free with a Cisco Networking Academy account).
2. Open `packet-tracer/enterprise-network.pkt`.
3. Wait about 60 seconds for STP, HSRP and OSPF to converge (or click **Fast Forward Time**).
4. From any PC: **Desktop → Command Prompt** → `ipconfig` should show a DHCP address. Then `ping` a PC in the other building.

To rebuild from scratch, follow [docs/03-implementation-guide.md](docs/03-implementation-guide.md) and paste the files in [`configs/`](configs/).

---

## Verification summary

| Test | Result |
|---|---|
| PCs receive DHCP addresses in the correct VLAN | ✅ |
| Inter-VLAN ping inside each building | ✅ |
| HQ ↔ Branch ping across the WAN | ✅ |
| OSPF neighbors FULL on all links | ✅ |
| HSRP failover: shut active MLS, pings continue | ✅ |
| SSH to every switch/router; Telnet refused | ✅ |
| Port security shuts a port when an unknown MAC appears | ✅ |

Commands and screenshots are in [docs/04-verification-testing.md](docs/04-verification-testing.md).

---

## What I learned

- How to design for **redundancy at every layer**: dual uplinks, HSRP gateways and STP root placement that matches the HSRP active router.
- How **DHCP relay** lets one central server address many VLANs.
- How OSPF passive interfaces stop routing updates from going out to user VLANs.
- Why the native VLAN should be an unused VLAN, and why unused ports should be shut down.

## Possible next steps

- Add an ISP router with **NAT/PAT** for internet access
- **ACLs** that limit Finance access to HR/Sales
- Redundant WAN link (second serial or a GRE/IPsec tunnel)
- **EtherChannel** between the distribution switches
- Centralized **AAA (RADIUS/TACACS+)**, Syslog and NTP servers
- Wireless access for each department (WLC + LAPs)

---

## Credits

Built by following [this tutorial](https://youtu.be/RvPR2BeqwLw) and then documented and extended by **[Your Name]**.

## License

MIT. See [LICENSE](LICENSE).
