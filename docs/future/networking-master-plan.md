# Networking Master Plan (PukkancsLak, 2026)

This document is the detailed technical design for the **home network and infrastructure** at PukkancsLak.  
It defines how UniFi, VLANs, WiFi/Thread, and core services are structured to support all home automation subsystems, including heating, cooling, lighting, sensors, and alarms.

For the **heating, cooling, and climate design** that depends on this network, see [`heating-master-plan.md`](heating-master-plan.md). For property layout (floors, outdoor areas) see [property overview](../property-overview.md).

---

## 1. Design Principles

- **Reliability first**: Core switching and routing on UniFi (UDM SE, aggregation switch, Pro/PoE switches) with clean fibre / 10GbE links.
- **Segregation**: Clear separation between Core, Server, IoT, and “no‑internet” (NOT) segments enforced by firewall rules.
- **Local-first**: Home Assistant, MariaDB, Frigate, and other services run on‑prem (Unraid / Raspberry Pi 5), with only minimal, controlled cloud dependencies.
- **Thread and WiFi coexistence**: Thread operates on a fixed channel (e.g. 25) with WiFi channels and AP placement chosen to minimise RF interference.
- **Observability**: Network and key services expose enough metrics and logs (via UniFi and Unraid) to debug issues without guesswork.

---

## 2. Physical Topology

### 2.1 Core Devices

All core devices below (except USW Ultra 60W) are in a **rack cabinet in the Garage**, powered by a **UniFi UPS 2U** in the same cabinet; the UPS also backs all PoE-connected equipment (APs, Aqara M3, Gate Hubs, etc.). Planned solar and battery storage (see [solar-battery-plan.md](solar-battery-plan.md)) will improve whole-house and rack resilience.

- **UDM SE** — main gateway, security appliance, and UniFi controller (Garage rack).
- **UniFi Aggregation Switch (SFP)** — 10G SFP+ core fabric linking the UDM SE, PoE switches, Unraid, and UNVR.
- **USW Pro Max 48 PoE** — primary PoE distribution (Garage rack):
  - Raspberry Pi 5 (Home Assistant host).
  - Aqara M3 hub.
  - Aqara M200 hubs (Guest House IR).
  - Olimex ESP32‑POE‑ISO (OpenTherm gateway).
- **USW XG 10 PoE** — uplink switch for high‑throughput access points (Garage rack; APs in hallway, Living Room, Guest House, front, rear).
- **UNVR** — UniFi Protect NVR (cameras); 10G SFP+ to Aggregation (Garage rack).
- **USW Ultra 60W** — engine room; fed by **2 new CAT6a** runs from core. Powers engine-room devices (e.g. OpenTherm gateway) and keeps cable runs short.
- **Unraid Server (Ryzen 5900X, 100TB)** — runs MariaDB (HA recorder), Frigate, and storage/backup services (Garage rack).

### 2.2 Access Points and Outdoor

| AP / Device | Location | Uplink / Notes |
| :--- | :--- | :--- |
| **U7 Pro XG** | Hallway next to Garage | Existing; PoE from USW XG 10 PoE |
| **U7 Pro XG** | Living Room | **New CAT6a** to USW XG 10 PoE (or Pro Max 48 as designed) |
| **U7 Pro XG** | Guest House | **New CAT6a** to USW XG 10 PoE (or Pro Max 48 as designed) |
| **U7 Outdoor** | Rear garden | Repositioned to improve coverage in outbuilding (Sonos) |
| **UK-ULTRA** | Front of house | New deployment; covers front outdoor area and gate access |

**New cabling required:** 2× CAT6a to engine room (for USW Ultra 60W); 2× CAT6a for Living Room and Guest House U7 Pro XG APs. U7 Outdoor in rear garden will be repositioned to improve coverage in the outbuilding (Sonos speakers).

### 2.3 High-Level Topology Diagram

See also the architecture diagram in [`heating-master-plan.md`](heating-master-plan.md) for how the climate system sits on top of this network.

```text
[Fiber/Cable WAN]
    └── UDM SE (Gateway & Controller) — Garage rack [UniFi UPS 2U]
          └── [10G SFP+] UniFi Aggregation Switch (SFP)
                ├── [10G SFP+] USW Pro Max 48 PoE
                │              ├── [PoE+]  Raspberry Pi 5 (Home Assistant)
                │              ├── [PoE]   Aqara M3 Hub
                │              ├── [PoE]   Aqara M200 x2 (Guest House)
                │              └── [2× CAT6a]  Engine Room → USW Ultra 60W → Olimex ESP32-POE-ISO
                ├── [10G SFP+] USW XG 10 PoE
                │              ├── [PoE]   U7 Pro XG (Hallway)
                │              ├── [CAT6a] U7 Pro XG (Living Room)
                │              ├── [CAT6a] U7 Pro XG (Guest House)
                │              └── [PoE]   U7 Outdoor (Rear garden, repositioned)
                ├── [PoE] UK-ULTRA (Front of house)
                ├── [10G SFP+] UNVR (Protect)
                └── [10G SFP+] Unraid Server (MariaDB, 100TB)
```

---

## 3. VLAN & Subnet Design

| Network | VLAN ID | Subnet | Devices (examples) | WAN Access |
| :--- | :--- | :--- | :--- | :--- |
| **Core / Management** | 1 | `192.168.10.x` | UDM SE, switches, APs | Yes (for updates / controller services) |
| **Server** | 6 | `192.168.1.x` | Unraid server and core services | Yes (restricted) |
| **IoT** | 3 | `192.168.12.x` | HA host, Aqara hubs, Shelly devices, Olimex OT gateway, MDV dongle, Gree/LG AC | Restricted |
| **NOT (No Internet)** | 4 | `192.168.13.x` | Devices that must never reach WAN but need LAN | No |

### 3.1 IoT VLAN Rules (Critical for Heating)

- **Allow**:
  - IoT → Home Assistant host (bidirectional) on required ports (API, MQTT if used, ESPHome, etc.).
  - Home Assistant → Unraid MariaDB (recorder DB).
- **Restrict**:
  - IoT → WAN only for explicitly whitelisted update endpoints (firmware, time, maybe vendor APIs where unavoidable).
- **Block**:
  - Lateral movement from IoT → Core / Server subnets except for narrowly defined service paths (e.g. HA → MariaDB).

This ensures that even if a cloud or IoT device is compromised, it has minimal lateral movement and cannot break the rest of the network or leak data.

---

## 4. WiFi & Thread Configuration

### 4.1 2.4 GHz WiFi

- Use fixed 2.4 GHz channels (e.g. **1 or 6**) across U7 Pro XG APs to:
  - Avoid overlap with **Thread Channel 25** as much as possible.
  - Provide consistent coverage for legacy WiFi IoT devices (Shelly, MDV dongle, etc.).

### 4.2 Thread

- Aqara M3 operates as the **Thread Border Router**.
- W500 thermostats, W600 TRVs, W100 displays, and T1 sensors join the Thread mesh.
- U7 Pro XG AP placement and configuration must ensure:
  - Strong Thread coverage in all climate‑critical rooms (engine room, living room, bedrooms, guest house, office).
  - Minimal RF shadowing from walls/floors; adjust AP placement if necessary.

### 4.3 SSID Strategy

- **User SSID(s)** on Core/standard VLANs for laptops, phones, etc.
- **IoT SSID** mapped to the **IoT VLAN** for:
  - MDV WiFi dongle.
  - Gree/LG office AC unit.
  - Shelly devices (e.g. radiator circuit enable).
  - Any other WiFi‑only IoT devices.
- Guest SSID if needed, with strict internet‑only access and no LAN.

---

## 5. Core Services & Dependencies

### 5.1 Home Assistant

- Runs on **Raspberry Pi 5 with PoE+ and NVMe** or equivalent, located on the **IoT VLAN**.
- Must always be reachable from:
  - Olimex ESP32‑POE‑ISO (OpenTherm).
  - Aqara M3 and M200 hubs.
  - Shelly Plus 1 (radiator circuit).
  - MDV WiFi dongle.
  - Gree/LG office AC unit.

### 5.2 Database & Storage

- **MariaDB on Unraid** hosts the HA recorder database.
- Recorder config in HA points to MariaDB, not local SQLite, to:
  - Avoid wearing out Pi NVMe storage.
  - Centralise history and telemetry on the Server VLAN.

### 5.3 DNS, NTP, and Time

- UDM SE provides DNS and NTP services for all VLANs.
- IoT devices and HA should use local NTP/DNS wherever possible to:
  - Reduce WAN dependency for basic operation.
  - Keep log timestamps consistent across devices and services.

### 5.4 Key Consumers: Heating & Climate

The **Heating & Climate Master Plan** ([`heating-master-plan.md`](heating-master-plan.md)) depends on:

- Stable LAN connectivity between:
  - HA and the OpenTherm gateway.
  - HA and Aqara hubs (M3, M200).
  - HA and MDV WiFi dongle, Gree/LG office AC.
- Proper VLAN isolation and firewalling such that:
  - A WAN outage does **not** break heating or automation logic.
  - Local automations across IoT and Server VLANs continue to function.

---

## 6. Security & Maintenance

- **Least privilege**: Restrict firewall rules to only what is needed for automation flows.
- **Firmware policy**:
  - Schedule regular but controlled firmware upgrades for UniFi, Aqara, Shelly, and ESPHome devices.
  - Test critical upgrades (especially for OT/PoE hardware) outside heating season where possible.
- **Backups**:
  - Regular backups of:
    - UniFi network configuration.
    - Home Assistant configuration and MariaDB.
    - Critical automation blueprints for the heating system.

---

## 7. Open Items (Networking)

- Confirm final VLAN IDs and subnets in UniFi and ensure they match this document.
- Validate Thread channel and 2.4 GHz WiFi configuration to minimise interference.
- **Resolved:** Home Assistant runs on **Raspberry Pi 5** (IoT VLAN). Unraid is for data backup, heavy visualization (Grafana), and other server workloads.
- Define which external endpoints (if any) IoT devices are allowed to reach for firmware and time.

For **access control** (gates, doors, Gate Hubs, cameras) see [access-control-plan.md](access-control-plan.md).

---

## 9. Product References

Official product pages and documentation:

| Product | Store / manufacturer |
| :--- | :--- |
| UDM SE | [store.ui.com](https://store.ui.com/us/en/products/udm-se) |
| UniFi Aggregation Switch (SFP) | [store.ui.com](https://store.ui.com/us/en/products/usw-aggregation) |
| USW Pro Max 48 PoE | [store.ui.com](https://store.ui.com/us/en/products/usw-pro-max-48-poe) |
| USW XG 10 PoE | [store.ui.com](https://store.ui.com/us/en/products/usw-pro-xg-10-poe) |
| USW Ultra 60W | [store.ui.com](https://store.ui.com/us/en/products/usw-ultra-60w) |
| UNVR | [store.ui.com](https://store.ui.com/us/en/products/unvr) |
| U7 Pro XG | [store.ui.com](https://store.ui.com/us/en/products/u7-pro-xg) |
| U7 Outdoor | [store.ui.com](https://store.ui.com/us/en/products/u7-outdoor) |
| UK-ULTRA | [store.ui.com](https://store.ui.com/us/en/products/uk-ultra) |
| UniFi UPS 2U | [store.ui.com](https://store.ui.com/us/en/products/ups-2u-us) |
| Tech specs (all UniFi) | [techspecs.ui.com](https://techspecs.ui.com/) |
| Aqara Hub M3 | [aqara.com](https://us.aqara.com/products/aqara-smart-hub-m3) |

---

*This document is intended to evolve alongside [`heating-master-plan.md`](heating-master-plan.md) and other subsystem plans (lighting, sensors, alarms). Update both when changes affect cross‑cutting concerns like VLANs, RF planning, or core services.*

