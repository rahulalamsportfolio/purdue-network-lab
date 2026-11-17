# Purdue Model Industrial Network – Cisco Packet Tracer Lab

A segmented industrial network built in Cisco Packet Tracer following the **Purdue Model** (ISA-95). The design separates enterprise IT, an industrial DMZ, and an OT zone using VLANs, inter-VLAN routing and static routing between zones.

---

## Network Overview

Three isolated zones with two firewall boundaries:

```
[ IT Zone ]  --FW1--  [ Industrial DMZ ]  --FW2--  [ OT Zone ]
```

| Zone | Purpose | Devices |
|------|---------|---------|
| IT | Office network | PCs, servers, admin hosts |
| DMZ | Shared services | Reporting server, TSN time server |
| OT | Industrial control | PLCs, HMI/SCADA, sensors, motor |

---

## VLAN Design

| VLAN | Name | Subnet | Zone |
|------|------|--------|------|
| 110 | IT Users | 192.168.110.0/24 | IT |
| 120 | IT Servers | 192.168.120.0/24 | IT |
| 130 | IT Admin | 192.168.130.0/24 | IT |
| 200 | DMZ | 192.168.200.0/24 | DMZ |
| 879 | PLC_Control | 10.87.9.0/24 | OT |
| 880 | SCADA | 10.88.0.0/24 | OT |

OT VLANs 879 and 880 are student-specific (derived from student ID).

---

## Routing Design

**IT zone:** Inter-VLAN routing via SVIs on a Cisco 3560 Layer 3 switch (L3-CORE-SWITCH).

**OT zone:** SW-OT is a Layer 2 only switch. FW2 (Cisco 2911 router) handles OT routing using router-on-a-stick with subinterfaces:
- `GigabitEthernet0/1.879` – gateway for VLAN 879 (10.87.9.1)
- `GigabitEthernet0/1.880` – gateway for VLAN 880 (10.88.0.1)

**Cross-zone:** Static routes on FW1 and FW2 handle traffic between IT, DMZ and OT.

---

## Topology

![Full topology](Full%20Toplogy%20View.png)

---

## VLAN Evidence

**IT VLANs on L3-CORE-SWITCH:**

![L3 switch VLANs](VLAN%20database%20on%20L3-CORE-SWITCH.png)

**OT VLANs on SW-OT:**

![SW-OT VLANs](VLAN%20database%20on%20SW-OT.png)

---

## Connectivity Tests

All 7 tests passed with 0% packet loss.

| Test | Result |
|------|--------|
| IT-PC-1 to IT-PC-2 (intra-VLAN 110) | Pass |
| IT-PC-1 to gateway 192.168.110.1 | Pass |
| IT-SERVER to gateway 192.168.120.1 | Pass |
| IT-ADMIN-PC to gateway 192.168.130.1 | Pass |
| PLC-1 to PLC-2 (intra-VLAN 879) | Pass |
| HMI-SCADA to gateway 10.88.0.1 | Pass |
| OT-ENG-PC to HMI-SCADA (intra-VLAN 880) | Pass |

![IT PC ping](IT-PC-1%20PINGING%20IT-PC-2.png)
![PLC ping](PLC-1%20PINGING%20PLC-2.png)

---

## Cross-Zone Traceroute

Path from IT-PC-1 (192.168.110.10) to DMZ Reporting Server (192.168.200.10):

```
192.168.110.10 -> 192.168.110.1 (FW1 inside) -> 172.16.1.2 (FW1 DMZ) -> 192.168.200.10
```

![Traceroute](Traceroute%20IT%20to%20DMZ.png)

---

## OT Independence Test

Disconnected the IT uplink above FW1. OT zone continued operating with no interruption – PLC-to-sensor communications and PLC-to-PLC pings kept working. This demonstrates correct Purdue Model zone isolation.

![Cable disconnected](Cable%20Disconnected%20between%20L3-CORE-SWITCH%20and%20FW1.png)
![OT continuity](OT%20Continuity%20Test.png)

---

## Industrial Process Simulation

PLC-PT programmed in JavaScript (PT Industrial Automation App) to control a motor actuator based on a temperature sensor reading. Two states demonstrated:

**Motor OFF – temperature below threshold:**

![Motor off](PLC%20Motor%20OFF%20State.png)

**Motor ON – threshold condition met:**

![Motor on](PLC%20Motor%20ON%20State.png)

---

## Time-Sensitive Networking (TSN)

| Parameter | Value |
|-----------|-------|
| TSN server IP | 192.168.200.20 (VLAN 200, DMZ) |
| OT time clients | PLC-1, PLC-2 (VLAN 879), HMI-SCADA (VLAN 880) |
| Time sync path | OT device -> FW2 -> DMZ -> TSN server |
| Critical path | PLC-1 to PLC-2, 1ms cycle, 500us latency target |
| Time sync protocol | IEEE 802.1AS (gPTP) |
| Scheduled traffic | IEEE 802.1Qbv |

---

## Tools

- Cisco Packet Tracer 8.x
- Cisco 2960 (Layer 2 switching)
- Cisco 3560-24PS (Layer 3 switching, SVIs)
- Cisco 2911 (routing, router-on-a-stick)
- PLC-PT, temperature sensors, motor actuator (IoT simulation)
