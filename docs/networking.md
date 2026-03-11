# Networking (PukkancsLak)

*Last updated: 2026-02-26*

Network and infrastructure for the home automation system. For property layout see [property-overview.md](property-overview.md). For heating and climate (which depend on this network) see [heating.md](heating.md). For access control see [access-control.md](access-control.md).

---

## Current State

### 1. Core Equipment

**Rack cabinet (Garage):** UDM SE, UniFi Aggregation Switch (SFP), USW Pro Max 48 PoE, USW XG 10 PoE, **UNVR**, **Unraid Server**, and **Raspberry Pi 5** (Home Assistant) are installed in a rack cabinet in the Garage. A **UniFi UPS 2U** in the same cabinet powers the rack and all PoE-connected equipment (switches, APs, Aqara M3, Gate Hubs, etc.).

| Device | Role | Uplink | Location |
| :--- | :--- | :--- | :--- |
| **UDM SE** | Main security gateway, UniFi controller | Fiber/Cable WAN | Garage (rack) |
| **UniFi Aggregation Switch (SFP)** | 10G SFP fabric | 10G SFP+ to UDM SE | Garage (rack) |
| **USW Pro Max 48 PoE** | PoE distribution | 10G SFP+ to Aggregation | Garage (rack) |
| **USW XG 10 PoE** | AP uplink switch | 10G SFP+ to Aggregation | Garage (rack) |
| **UNVR** | UniFi Protect NVR (cameras) | 10G SFP+ to Aggregation | Garage (rack) |
| **Unraid Server** | Storage, backups, telemetry | 10G SFP+ to Aggregation | Garage (rack) |
| **Raspberry Pi 5** | Home Assistant host | PoE+ from USW Pro Max 48 | Garage (rack) |
| **U7 Pro XG** | WiFi 7 coverage | PoE from USW XG 10 PoE | Hallway next to Garage |
| **U7 Outdoor** | Outdoor WiFi coverage | PoE | Rear garden (coverage weak in Grill House for Sonos) |
| **Aqara M3 Hub** | Zigbee coordinator, Matter/Thread border router | PoE from USW Pro Max 48 | Garage / Central |

### 2. Connectivity & Sonos

- The 10G backbone (UDM SE → Aggregation → Pro Max 48, XG 10 PoE, Unraid) is deployed as designed.
- The Aqara M3 Hub provides Zigbee and Thread/Matter connectivity.
- **U7 Outdoor AP** will need repositioning to improve coverage in the Grill House for Sonos speakers.

**Sonos equipment (wireless):** All run over WiFi per [UniFi best practices for Sonos](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices).

| Location | Equipment |
| :--- | :--- |
| Living Room (Entertainment Area) | Arc, Sub 3, 2× Era 500 |
| Master Bedroom | 2× SYMFONISK Gen 1 |
| Grill House | 2× Play 3 |

### 3. Cabling & Distribution

**Room drops:** Main-house drops terminate in the **office built-in wardrobe**. Distribution from office to rooms is **CAT5e**. Guest House drops currently terminate in the office wardrobe via the Attic path. Whether to keep this or switch to direct runs from the rack is **to be decided with the installer** during refurbishment planning.

| Room | Points | Notes |
| :--- | :--- | :--- |
| Office | 6 | |
| Vicky Room | 2 | |
| Alex Room | 2 | |
| Playroom | 4 | |
| Guest Bedroom | 2 | To be decided with installer: keep or replace with direct run |
| Guest Living Room | 8 | To be decided with installer: keep or replace with direct run |

**Trunk (Rack → High Attic → Office):** CAT6a. **Distribution (office → main-house rooms):** CAT5e. Only **2× CAT6a** currently run from High Attic to office (Office PC, USW-Ultra for Mac Mini, PS5, printer). LG C4 48" OLED (office): WiFi.

**High Attic:** 24× CAT6a between garage rack and High Attic. In use: 2→Office, 1→5th floor hallway U7 Pro XG, 1→M3 Hub, 1→UniFi PoE Chime, 1→Playroom G6 Turret. ~18 spare.

**Floor 3 gaps:** No PoE at Connect Display location; no PoE for Living Room ceiling (U7 Pro XG); no Ethernet at entertainment area.

---

## Planned / Target

### 1. Design Principles

- **Reliability first:** Core switching and routing on UniFi with clean fibre / 10GbE links.
- **Segregation:** Core, Server, IoT, and NOT (no-internet) segments with firewall rules.
- **Local-first:** HA on Pi 5, MariaDB on Unraid; minimal cloud dependency.
- **Thread and WiFi coexistence:** Thread on fixed channel (25); WiFi channels chosen to minimise RF interference.
- **Observability:** Enough metrics and logs to debug without guesswork.

### 2. Physical Topology

**Core devices** (Garage rack): UDM SE, Aggregation Switch, USW Pro Max 48 PoE, USW XG 10 PoE, UNVR, Unraid, Pi 5 (HA), Aqara M3, Aqara M200×2 (Guest House IR), Olimex ESP32-POE-ISO (OpenTherm). **USW Ultra 60W** in engine room fed by 2× new CAT6a from rack.

**Access points:**

| AP / Device | Location | Notes |
| :--- | :--- | :--- |
| U7 Pro XG | Hallway | Existing |
| U7 Pro XG | Living Room ceiling | New CAT6a (PoE) |
| U7 Pro XG | Guest Living Room | Direct run from rack (1→2) |
| UniFi Connect Display | Wall between Dining and Kitchen | New CAT6a (PoE) |
| U7 Outdoor | Rear garden | Reposition for Grill House |
| UK-ULTRA | Front of house | New deployment |

**New cabling:** 2× CAT6a to engine room; 1× to Living Room ceiling; 2× (1 PoE) to Dining/Kitchen wall; 2× to Living Room entertainment; Guest House cabling (see Section 3 below); door access runs (see [access-control.md](access-control.md)). Solar/battery (see [solar-battery.md](solar-battery.md)) will improve rack resilience.

### 3. Cable Routing & Guest House

**Principle:** All cables in-wall or in conduit. No visible cables after refurbishment. **Guest House routing is a key decision to make with the installer** — see options below.

**Floor layout** (see [property-overview.md](property-overview.md)): Split-level; vertical runs (e.g. 1→3) typically easier than lateral (e.g. 1→2). Floors 6–7 are empty — cable runs straightforward.

| Path | Notes |
| :--- | :--- |
| Rack → High Attic | 24× CAT6a existing |
| Rack → Guest House | **Option to discuss with installer:** direct run (1→2) — shorter path than via Attic |
| High Attic → Office | Feeds office wardrobe |
| High Attic → Floor 3 | Living Room AP, Connect Display, entertainment |
| High Attic → Floor 5 | Hallway U7 Pro XG, Playroom camera, etc. |
| Rack → Engine Room | 2× for USW Ultra 60W |

**Guest House — option A (direct from rack):** Run 11× CAT6a from Rack to Guest House (1→2): 1× U7 Pro XG (ceiling), 1× M200 (Living Room), 1× M200 (Bedroom), 1× Door Hub Mini, 2× entertainment, 2× couch, 2× bedroom. Would supersede existing office-wardrobe drops. **Option B:** Keep/repurpose the existing Attic→Office→Guest House path. Compare cost, cable length, and installer recommendation.

### 4. VLAN & Subnet Design

| Network | VLAN ID | Subnet | WAN |
| :--- | :--- | :--- | :--- |
| Core / Management | 1 | 192.168.10.x | Yes |
| Server | 6 | 192.168.1.x | Yes (restricted) |
| IoT | 3 | 192.168.12.x | Restricted |
| NOT | 4 | 192.168.13.x | No |

**IoT rules:** Allow HA↔IoT; HA→MariaDB. Restrict IoT→WAN to whitelisted endpoints. Block lateral movement.

### 5. WiFi & Thread

- **2.4 GHz:** Fixed channels 1 or 6 to avoid Thread Channel 25.
- **Thread:** Aqara M3 as border router; W500, W600, T1 join mesh.
- **SSID:** User SSID(s); IoT SSID mapped to IoT VLAN; Guest SSID if needed.

### 6. Core Services

- **Home Assistant:** Pi 5 on IoT VLAN. Must reach OpenTherm gateway, Aqara hubs, Shelly, MDV WF-60A1 WiFi module, Gree/LG AC.
- **MariaDB:** On Unraid (Server VLAN); HA recorder.
- **DNS/NTP:** UDM SE.

### 7. Product References

| Product | Store |
| :--- | :--- |
| UDM SE, Aggregation, USW Pro Max 48, XG 10, USW Ultra 60W | [store.ui.com](https://store.ui.com) |
| UNVR, U7 Pro XG, U7 Outdoor, UK-ULTRA | [store.ui.com](https://store.ui.com) |
| UniFi UPS 2U | [store.ui.com](https://store.ui.com) |
| Aqara Hub M3 | [aqara.com](https://us.aqara.com/products/aqara-smart-hub-m3) |
