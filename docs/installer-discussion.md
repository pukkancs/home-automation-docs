# Installer Discussion: Planned Changes

Quick reference document for discussing installation requirements and changes with the installer.

---

## Access Control

### Gate Hubs
- **Third Gate Hub** — Install new UniFi Gate Hub dedicated to main house door and garage door control
  - Controls main house front door fail-safe lock
  - Controls outside garage door via garage door opener (relay output)
  - Requires **CAT6a** cabling to door area
  - **Verify:** Relay outputs from Gate Hub can drive garage opener; confirm wiring compatibility

### Main House Front Door
- Convert second lock to **knob-key storage-room-style** mechanism
- Add **1 fail-safe electric lock** paired with converted lock
- First (original) lock remains unchanged for extra security
- Automated access via linked **G6 Entry** (intercom/door station)
- **Open question:** One-handed exit solution without child-accessible exit button

### Garage
- **Outside garage door:** Wire garage door opener to main-house Gate Hub (third Gate Hub)
  - Add position sensors if required for state feedback
- **Garage interior door:** Install **UniFi Access Ultra** + fail-safe electric lock
  - PoE-powered, network-connected standalone controller
  - No Gate Hub required for this door

### Guest House Front Door
- Install **UniFi Door Hub Mini**
  - Requires **CAT6a** cabling to guest house
- Same lock strategy as main house: convert second lock, add fail-safe electric lock
- Automated access via linked **G6 Entry**
- **Open question:** One-handed exit solution

### Cameras
- **4× G6 Turret** cameras — one per gate (Main Car, Garage Car, Pedestrian, Waste)
  - Mount at top of house looking at each gate
- **2× G5 Turret Ultra** cameras — one on each side of house looking towards back garden

### Gate Modifications
- **Main Pedestrian Gate:** Convert to knob-based storage-room-style lock system + UACC-DoorCloser equivalent
- **Waste Gate:** Convert from electric strike + UA-G3 Reader to fixed knobs with key (manual lock only)
  - Remove UA-G3 Reader
  - Retain key access

---

## Heating & Climate

### Boiler Integration
- **Viessmann Vitodens 222-F** — Do NOT connect to VitoConnect/Vitotronic/cloud gateway
  - Leave OpenTherm terminals accessible
  - Install **Viessmann OpenTherm Extension Module** (accessory)
- **OpenTherm Gateway:** Olimex ESP32-POE-ISO + DIYLESS Master Shield
  - PoE-powered, 3000V galvanic isolation
  - Requires **CAT6 PoE** to USW Pro Max 48 PoE switch
  - Wiring: Boiler → OT Extension Module → 2-wire OpenTherm bus → DIYLESS Shield → ESP32-POE-ISO

### Underfloor Heating (UFH)
- **Wiring Centre:** Evaluate Salus wiring centre or equivalent
  - Minimum 3 channels (Guest House, Living Room, Bathroom if independent loop)
- **Aqara W500 thermostats:** Install for UFH zone control
  - Zone 1: Guest House (both rooms, shared)
  - Zone 2: Living Room
  - Zone 3: Bathroom (if independent loop confirmed — **verify before purchase**)
- **Pre-purchase action:** Physically verify at manifold whether bathroom has independent pipe loop

### Radiator Control
- **Aqara W600 TRVs** — Install on radiators throughout property
  - Main house: Living Room, Bathroom, Alex Room, Vicky Room, Playroom, Master Bedroom, Office
  - Thread/Matter protocol for communication

### Sensors
- **Aqara W100 display sensors** — Children's rooms and playroom (visible temperature display)
- **Aqara T1 standalone sensors** — Master Bedroom, Office, Guest House rooms
- **Outdoor temperature sensor** — For weather compensation/heating curve

### Air Conditioning
- **MDV Duct AirCon:** WiFi dongle integration for main house cooling
- **Guest House AirCon:** IR control via Aqara M200 hubs (PoE-powered)
- **Office AirCon:** Smart Gree/LG unit with WiFi integration

---

## Networking

### Core Infrastructure
- **UDM SE** — Main gateway and UniFi controller
- **UniFi Aggregation Switch (SFP)** — 10G SFP+ core fabric
- **USW Pro Max 48 PoE** — Primary PoE distribution
  - Raspberry Pi 5 (Home Assistant host)
  - Aqara M3 hub (Thread Border Router)
  - Aqara M200 hubs (Guest House IR)
  - Olimex ESP32-POE-ISO (OpenTherm gateway)
- **USW XG 10 PoE** — Uplink for U7 Pro XG access points
- **Unraid Server** — MariaDB (HA recorder), Frigate, storage services

### Cabling Requirements
- **CAT6a** to main house front door area (Gate Hub)
- **CAT6a** to guest house (Door Hub Mini)
- **CAT6 PoE** to OpenTherm gateway location
- **PoE** runs for all Aqara hubs, cameras, and network devices
- **10G SFP+** links for core switching infrastructure

### WiFi & Thread
- **U7 Pro XG APs** — WiFi 7 + Thread coverage
  - Ensure strong Thread coverage in all climate-critical rooms
  - Fixed 2.4 GHz WiFi channels (1 or 6) to avoid Thread Channel 25 interference
- **Thread Border Router:** Aqara M3 hub
- **IoT SSID** mapped to IoT VLAN for WiFi-only devices (MDV dongle, Gree/LG AC, Shelly devices)

### VLAN Configuration
- **Core/Management (VLAN 1):** `192.168.10.x` — UDM SE, switches, APs
- **Server (VLAN 6):** `192.168.1.x` — Unraid server, core services
- **IoT (VLAN 3):** `192.168.12.x` — HA host, Aqara hubs, Shelly, OpenTherm gateway, MDV, AC units
- **NOT/No Internet (VLAN 4):** `192.168.13.x` — Devices requiring LAN but no WAN access

---

## Open Items for Discussion

1. **Garage door opener integration:** Confirm relay outputs from Gate Hub can drive garage opener; verify wiring and compatibility
2. **One-handed exit:** Solution for main house and guest house doors without child-accessible exit buttons
3. **Bathroom UFH loop:** Verify if bathroom has independent UFH loop before ordering 3rd W500 thermostat
4. **Heat loss calculation:** Commission PN-EN 12831 calculation to determine correct boiler kW output (24–35 kW expected)
5. **Thread/WiFi interference:** Finalize Thread channel (25) and 2.4 GHz WiFi channel selection to minimize RF interference
6. **Pedestrian gate mechanism:** Confirm exact hardware for knob-based storage-room-style lock + UACC-DoorCloser equivalent
7. **Waste Gate conversion:** Confirm removal of UA-G3 Reader and conversion to fixed knobs with key

---

## Product References

Key products mentioned:
- UniFi Gate Hub, Door Hub Mini, Access Ultra
- G6 Turret, G5 Turret Ultra, G6 Entry cameras
- Aqara W500, W600, W100, T1, M3, M200
- Viessmann Vitodens 222-F + OpenTherm Extension Module
- Olimex ESP32-POE-ISO + DIYLESS OpenTherm Master Shield
