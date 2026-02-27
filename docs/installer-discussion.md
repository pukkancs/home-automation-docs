# Installer Discussion — PukkancsLak (2026)

Quick reference of planned changes for discussion with installers. See the detailed master plans in `docs/future/` for full specifications.

---

## 1. Networking & Cabling

- **2× CAT6a to engine room** — for USW Ultra 60W switch (powers OpenTherm gateway, keeps cable runs short).
- **2× CAT6a for APs** — Living Room and Guest House U7 Pro XG access points.
- **CAT6a to main house door** — for third Gate Hub (main door + garage door control).
- **CAT6a to guest house** — for Door Hub Mini.
- **U7 Outdoor reposition** — rear garden AP moved to improve coverage in outbuilding.
- **UK-ULTRA at front** — new AP for front outdoor area and gate access.

---

## 2. Heating & Boiler

- **Viessmann Vitodens 222-F** — condensing combi/system boiler. Heat loss calc (PN-EN 12831) required before sizing.
- **No Viessmann cloud** — installer must NOT connect VitoConnect/Vitotronic. Leave OpenTherm terminals accessible for third-party controller.
- **Viessmann OpenTherm Extension Module** — required accessory; connects internally, exposes 2-wire OpenTherm bus.
- **OpenTherm gateway** — Olimex ESP32-POE-ISO + DIYLESS Master Shield in engine room. PoE from garage rack via USW Ultra 60W. Galvanic isolation protects network from boiler electrical noise.

---

## 3. UFH & Wiring Centre

- **Salus wiring centre** — evaluate reuse vs. installer’s recommended replacement. Decision during design.
- **Aqara W500 thermostats** — replace Salus wired stats. Volt-free relay output. 2–3 zones (Guest House, Living Room, Bathroom if loop independent).
- **Bathroom UFH loop** — verify at manifold whether bathroom has independent loop before ordering third W500.
- **Radiator circuit** — rewired through Shelly Plus 1 (DIN rail). HA controls as master enable relay; no individual room stats.

---

## 4. Radiators & TRVs

- **Aqara W600 TRVs** — replace existing TRV heads on all radiators (8 minimum; full survey needed).
- **Wall sensors** — W100 (display) in kids’ rooms/playroom; T1 (standalone) in Master Bedroom, Office, Guest House. Primary room temp for HA logic; W600 built-in sensor not used (reads near radiator).

---

## 5. Air Conditioning

- **MDV duct system** — Midea WiFi dongle for LAN control (no cloud). Serves Living Room, Alex, Vicky, Playroom, Master Bedroom.
- **Guest House** — 2× Aqara M200 hubs (PoE) for IR control of old AC units. No smart connectivity on units.
- **Office** — Gree or LG smart AC; LAN integration. Confirm model for correct HA integration.

---

## 6. Access Control — Gates

- **Controller 1 (Main Car + Pedestrian)** — add camera; change Pedestrian Gate to knob-based storage-room-style lock + door closer.
- **Controller 2 (Garage Car + Waste)** — Waste Gate: convert from electric strike + UA-G3 Reader to fixed knobs with key (manual only). Garage Car Gate unchanged.
- **Key access** — every gate retains mechanical key override.

---

## 7. Access Control — Doors

- **Main house front door** — third Gate Hub; convert second lock to knob-key + fail-safe electric lock. First lock stays for long absences. G6 Entry for intercom/remote access.
- **Garage outside door** — wired to main-house Gate Hub; opener controlled via relay. Verify relay compatibility with installer.
- **Garage interior door** — UniFi Access Ultra + fail-safe electric lock (PoE). Extra security layer.
- **Guest house door** — Door Hub Mini; same lock strategy as main house (second lock → knob-key + fail-safe electric).
- **Open question** — one-handed exit without child-accessible exit button (both main and guest house).

---

## 8. Cameras

- **1× G6 Turret** per gate — Main Car, Garage Car, Pedestrian, Waste (mounted at house top).
- **1× G5 Turret Ultra** each side — front of house, looking towards back. Back garden coverage later.

---

## 9. Solar & Battery (Separate Scope)

- **8 kW peak solar** — grid-tied; placement TBD.
- **20 kWh or high-voltage battery** — self-consumption, backup, peak shaving.
- **Rack integration** — clarify how battery interfaces with garage rack and UniFi UPS.

---

## 10. Blocking Items (Before Ordering)

| Item | Owner |
| :--- | :--- |
| Heat loss calculation (PN-EN 12831) | HVAC engineer |
| Bathroom UFH loop independence | Owner / plumber |
| Guest House boiler topology (shared or separate) | HVAC engineer |
| W600 radiator count (full survey) | Owner |
| Office AC brand/model (Gree vs LG) | Owner |

---

## 11. Verify With Installer

- Salus wiring centre reuse vs. replacement.
- Gate Hub relay outputs → garage opener compatibility.
- Main house door lock + garage opener on same Gate Hub — wiring and feasibility.
- Access Ultra + fail-safe lock on garage interior door.
- Pedestrian gate mechanism (knob + door closer) and hardware.
- Waste Gate conversion (electric strike removal, knob-only).

---

*For full technical details: `heating-master-plan.md`, `networking-master-plan.md`, `access-control-plan.md`, `solar-battery-plan.md`*
