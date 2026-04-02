# Routed Access-Layer Campus Network

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet_Tracer-1BA0D7?logo=cisco&logoColor=white)
![Cisco Design](https://img.shields.io/badge/Reference-Cisco_Routed_Access_Guide-0076CE)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

Routed access-layer campus network using L3 switching and multi-area OSPF — replacing traditional L2 access–distribution trunks with routed inter-switch links for simplified convergence and fault isolation. Based on the [Cisco Routed Access Design Guide](https://www.cisco.com/c/en/us/td/docs/solutions/Enterprise/Campus/routed-ex.html).

![Network Topology](RA_PKT.png)

---

## Architecture Overview

| Layer | Components | Role |
|-------|-----------|------|
| **Core** | L3 switches | OSPF Area 0 backbone, inter-area routing |
| **Distribution** | L3 switches | Area border routers, route summarization toward core |
| **Access** | L3 switches | SVIs per VLAN, local routing, DHCP relay, OSPF stub areas |
| **Server** | Infrastructure/App VLANs | HSRP for gateway redundancy (only tier requiring FHRP) |

### Why Routed Access?

| Traditional L2 Access | Routed Access |
|----------------------|---------------|
| STP required between access and distribution | STP only used for host-facing segments |
| Blocked redundant links | All uplinks actively forward traffic |
| HSRP/VRRP needed at every distribution pair | FHRP only needed for server VLANs |
| Trunk management overhead | No inter-switch trunks (L3 point-to-point) |
| Broadcast storms can propagate across tiers | Faults contained to local subnet |
| Extended STP convergence times | Sub-second OSPF convergence |

**Trade-offs:** Higher upfront cost (L3-capable access switches), more complex initial design, and L2 adjacency not available across routed segments.

## Addressing and OSPF Design

| Component | Addressing |
|-----------|-----------|
| Access VLANs | Unique /24 per VLAN (e.g., VLAN 10: 10.10.10.0/24, VLAN 20: 10.10.20.0/24) |
| Server VLANs | VLAN 100 (Infrastructure), VLAN 200 (Applications) — HSRP-protected |
| Loopbacks | Summarization anchors (10.10.0.0/16, 10.20.0.0/16) |
| DHCP | Relay via `ip helper-address` under access SVIs |

**OSPF Area Design:**

| Area | Assignment | Type |
|------|-----------|------|
| 0 | Core-facing interfaces | Backbone |
| 10 | Access-layer networks (site 1) | Stub |
| 20 | Access-layer networks (site 2) | Totally stubby |

Route summarization applied at distribution toward the core to reduce LSDB size and SPF computation overhead. Loopback interfaces configured as passive.

---

## Configuration Summary

**Routing** — OSPF with per-area stub configuration. Inter-switch links are routed (no switchport); trunks retained only for server VLAN distribution where HSRP is required.

**Redundancy** — HSRP on VLAN 100 and 200 with priority and preemption. No FHRP needed at the access layer since the local L3 switch is the default gateway.

**Spanning Tree** — Rapid PVST+ on access switches for host-facing redundancy only. No STP dependency between network tiers.

**SVIs** — Configured on access switches (not distribution), enabling local routing decisions and faster convergence.

## Validation

- Simulated link failures showed fault containment to the affected subnet with no propagation to adjacent areas.
- All uplinks remained active and forwarding during steady-state operation.
- OSPF reconvergence completed within expected timers on simulated topology changes.
- No broadcast storms or STP instability observed during failure scenarios.
