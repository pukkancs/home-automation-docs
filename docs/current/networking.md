# Current Networking (PukkancsLak)

*Last updated: 2026-02-26*

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
| **U7 Outdoor** | Outdoor WiFi coverage | PoE | Rear garden (covers back garden; coverage weak in Grill House for Sonos) |
| **Aqara M3 Hub** | Zigbee coordinator, Matter/Thread border router | PoE from USW Pro Max 48 | Garage / Central |

## 2. Connectivity

- The 10G backbone (UDM SE → Aggregation → Pro Max 48, XG 10 PoE, Unraid) is deployed as designed.
- The Aqara M3 Hub is powered over PoE from the USW Pro Max 48 PoE and provides Zigbee and Thread/Matter connectivity for current and future Aqara devices.
- **U7 Outdoor AP** will need repositioning to improve coverage in the Grill House for Sonos speakers.

### 2.1 Sonos Equipment (Wireless)

All Sonos equipment runs over WiFi (wireless) per [UniFi best practices for Sonos](https://help.ui.com/hc/en-us/articles/18930473041047-Best-Practices-for-Sonos-Devices).

| Location | Equipment |
| :--- | :--- |
| Living Room (Entertainment Area) | Arc, Sub 3, 2× Era 500 |
| Master Bedroom | 2× SYMFONISK Gen 1 |
| Grill House | 2× Play 3 |

## 3. Cabling & Distribution

### 3.1 Room Drops (Office / Main House Upper Floors)

Main-house room drops below terminate in the **office built-in wardrobe**. Distribution from the office to rooms is **CAT5e**. Guest House drops will be **replaced by direct runs** from High Attic during refurbishment (see [networking master plan](../future/networking-master-plan.md)).

| Room | Points | Notes |
| :--- | :--- | :--- |
| Office | 6 | |
| Vicky Room | 2 | |
| Alex Room | 2 | |
| Playroom | 4 | |
| Guest Bedroom | 2 | Legacy; superseded by direct run |
| Guest Living Room | 8 | Legacy; superseded by direct run |
| **Main house total** | **14** | Office, Vicky, Alex, Playroom |
| **Guest House (legacy)** | **10** | Will be replaced |

**Trunk (Rack → High Attic → Office):** CAT6a. **Distribution (office → main-house rooms):** CAT5e.

**Current uplink from high attic to office:** Only **2× CAT6a** run from the high attic to the office.

- **Wire 1:** Office PC — 5 Gb/s direct link.
- **Wire 2:** USW-Ultra — provides connectivity for Office Mac Mini, PS5, HP Laser Printer.
- **LG C4 48" OLED** (office): WiFi.

Most room drops in the office wardrobe are **not yet connected** due to the 2-cable uplink limit.

### 3.2 High Attic Wiring Centre

- **24× CAT6a** cables between the garage rack and the high attic.
- **Currently in use:**
  - 2 → Office
  - 1 → 5th floor hallway U7 Pro XG
  - 1 → M3 Hub
  - 1 → UniFi PoE Chime
  - 1 → Playroom G6 Turret
- **~18 cables spare** for future runs (Living Room, Floor 3, Guest House direct from rack, engine room, door access, etc.).

### 3.3 Floor 3 — Living Area, Dining Area, Kitchen

Floor 3 has three sections: **Living Area** (couch, entertainment), **Dining Area** (tables, chairs), and **Kitchen**. The UniFi Connect Display will go on the wall between Dining and Kitchen; the Living Room entertainment area needs Ethernet for Shield TV, PS5, LG CX 65" OLED.

**Current gaps:** No PoE at Connect Display location; no PoE for Living Room ceiling (U7 Pro XG); no Ethernet at entertainment area. See [networking master plan](../future/networking-master-plan.md) for planned runs.

---

For **access control** (gates, doors, Gate Hubs) see [access-control.md](access-control.md).

---

*Update this document as the network changes so it stays an accurate snapshot of what is deployed.*
