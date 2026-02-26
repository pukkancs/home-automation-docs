# Current Networking (PukkancsLak)

This document describes the **current, deployed** network and infrastructure. For the target design see the [future plan](../future/future-plan.md) and [networking master plan](../future/networking-master-plan.md). For property layout (floors, outdoor areas) see [property overview](../property-overview.md).

---

## 1. Core Equipment

**Rack cabinet (Garage):** UDM SE, UniFi Aggregation Switch (SFP), USW Pro Max 48 PoE, USW XG 10 PoE, **UNVR**, **Unraid Server**, and **Raspberry Pi 5** (Home Assistant) are installed in a rack cabinet in the Garage. A **UniFi UPS 2U** in the same cabinet powers the rack and all PoE-connected equipment (switches, APs, Aqara M3, Gate Hubs, etc.).

| Device | Role | Uplink | Location |
| :--- | :--- | :--- | :--- |
| **UDM SE** | Main security gateway, UniFi controller | Fiber/Cable WAN | Garage (rack) |
| **UniFi Aggregation Switch (SFP)** | Core 10G SFP fabric | 10G SFP+ to UDM SE | Garage (rack) |
| **USW Pro Max 48 PoE** | PoE distribution | 10G SFP+ to Aggregation | Garage (rack) |
| **USW XG 10 PoE** | AP uplink switch | 10G SFP+ to Aggregation | Garage (rack) |
| **UNVR** | UniFi Protect NVR (cameras) | 10G SFP+ to Aggregation | Garage (rack) |
| **Unraid Server** | Storage, backups, telemetry | 10G SFP+ to Aggregation | Garage (rack) |
| **Raspberry Pi 5** | Home Assistant host | PoE+ from USW Pro Max 48 | Garage (rack) |
| **U7 Pro XG** | WiFi 7 coverage | PoE from USW XG 10 PoE | Hallway next to Garage |
| **U7 Outdoor** | Outdoor WiFi coverage | PoE | Rear garden (covers back garden; coverage weak in outbuilding for Sonos) |
| **Aqara M3 Hub** | Zigbee coordinator, Matter/Thread border router | PoE from USW Pro Max 48 | Garage / Central |

## 2. Connectivity

- The 10G backbone (UDM SE → Aggregation → Pro Max 48, XG 10 PoE, Unraid) is deployed as designed.
- The Aqara M3 Hub is powered over PoE from the USW Pro Max 48 PoE and provides Zigbee and Thread/Matter connectivity for current and future Aqara devices.
- **U7 Outdoor AP** will need repositioning to improve coverage in the rear outbuilding for the 2 Sonos speakers.

For **access control** (gates, doors, Gate Hubs) see [access-control.md](access-control.md).

---

*Update this document as the network changes so it stays an accurate snapshot of what is deployed.*
