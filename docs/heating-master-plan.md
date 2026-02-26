# Project Documentation: PukkancsLak Smart Climate System (2026)

This document is the comprehensive technical design guide for the smart climate control system at PukkancsLak. It supersedes earlier drafts and incorporates all architectural decisions made through design review. The system integrates a **Viessmann Vitodens 222-F** boiler into a **UniFi 10GbE backbone**, using **Home Assistant** as the sole logic engine, with an emphasis on **local-first operation**, **OpenTherm modulation**, and **ecosystem consolidation** around the Aqara Matter/Thread/Zigbee stack.

For the underlying **network and infrastructure design** this system depends on, see:

- [`networking-master-plan.md`](networking-master-plan.md)

---

## Table of Contents

1. [System Philosophy](#1-system-philosophy)
2. [Physical Layout & Zone Map](#2-physical-layout--zone-map)
3. [Boiler & OpenTherm Strategy](#3-boiler--opentherm-strategy)
4. [Heating & Cooling Architecture](#4-heating--cooling-architecture)
5. [UFH Manifold & Wiring Centre](#5-ufh-manifold--wiring-centre)
6. [Radiator Control (W600 TRVs)](#6-radiator-control-w600-trvs)
7. [MDV Duct AirCon Integration](#7-mdv-duct-aircon-integration)
8. [Guest House AirCon (IR Control)](#8-guest-house-aircon-ir-control)
9. [Office AirCon (Gree/LG)](#9-office-aircon-greelg)
10. [Sensor Strategy](#10-sensor-strategy)
11. [Schedule & Automation Logic](#11-schedule--automation-logic)
12. [Network Architecture](#12-network-architecture)
13. [Control & Display Hierarchy](#13-control--display-hierarchy)
14. [Hardware Bill of Materials](#14-hardware-bill-of-materials)
15. [Risks & Critical Maintenance Notes](#15-risks--critical-maintenance-notes)
16. [Open Items & Pre-Purchase Decisions](#16-open-items--pre-purchase-decisions)
17. [System Architecture Diagram](#17-system-architecture-diagram)

---

## 1. System Philosophy

### Core Principles

| Principle | Implementation |
| :--- | :--- |
| **Local-first** | All control logic runs in Home Assistant on-premises. No internet dependency for heating, cooling, or scheduling. |
| **Ecosystem consolidation** | Primary RF protocol is Aqara Matter/Thread/Zigbee. WiFi (Shelly, ESPHome) is used only where mains switching is required. No new ecosystems added unless unavoidable. |
| **OpenTherm modulation** | Boiler runs "slow and low" — burner modulates rather than bang-bang cycling. Maximises condensing efficiency, prevents short-cycling, extends boiler life. |
| **Safety interlocks** | HA automations enforce hard rules: UFH and cooling never fight each other; MDV cannot heat a room the UFH is already heating. |
| **Graceful degradation** | If HA is unreachable, thermostats (W500, W600) fall back to their last setpoint. Boiler defaults to a safe flow temperature. The house remains warm. |

### Control Hierarchy (Who Wins)

```
Home Assistant (highest authority)
  └── Physical wall controls (W500, W600, W100) — manual override
        └── Boiler safety limits (lowest level, hardware-enforced)
```

---

## 2. Physical Layout & Zone Map

### Main House

| Room | Radiators | UFH | Cooling | Wall Sensor | Zone Type |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Living Room | — | W500 (UFH stat) | MDV Duct AirCon | W500 (built-in) | Hybrid — UFH primary heat, MDV cooling |
| Bathroom | 2x W600 | W500 (if loop independent, TBC) | — | W500 (built-in) | Hybrid UFH + Rad |
| Alex (child bedroom) | 1x W600 | — | MDV Duct AirCon (shared) | W100 (display sensor) | Hybrid Rad + MDV |
| Vicky (child bedroom) | 1x W600 | — | MDV Duct AirCon (shared) | W100 (display sensor) | Hybrid Rad + MDV |
| Playroom | 1x W600 | — | MDV Duct AirCon (shared) | W100 (display sensor) | Hybrid Rad + MDV |
| Master Bedroom | 2x W600 | — | MDV Duct AirCon (shared) | T1 (standalone sensor) | Hybrid Rad + MDV |
| Office | 1x W600 | — | Smart AirCon (Gree/LG) | T1 (standalone sensor) | Hybrid Rad + AirCon |

> **Sensor logic:** W100 display sensors are used in children's rooms and playroom where visibility of current temperature is useful. T1 standalone sensors are used in Master Bedroom and Office where a display is not needed. W500 and W600 have built-in sensors but wall sensors provide a more representative room-average reading. W600 sensors read close to the radiator and should not be used as the sole room temperature source for HA demand logic — always use the wall sensor as the primary temp source for HA automations.

### Guest House

| Room | Radiators | UFH | Cooling | Wall Sensor | Zone Type |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Guest Room 1 | — | UFH (shared zone, W500 #1) | Old AirCon (IR via M200) | T1 (standalone sensor) | Hybrid UFH + IR AC |
| Guest Room 2 | — | UFH (shared zone, W500 #1) | Old AirCon (IR via M200) | T1 (standalone sensor) | Hybrid UFH + IR AC |

> **Guest House UFH:** Both guest rooms share a single UFH zone controlled by W500 #1. Individual room temperature feedback comes from the T1 sensors — HA uses these to decide whether to trigger the IR AirCon in each room independently, even though the UFH zone covers both rooms together.

### UFH Zones Summary

| Zone | Location | Thermostat | Wiring Centre Channel |
| :--- | :--- | :--- | :--- |
| Zone 1 | Guest House (both rooms) | Aqara W500 #1 | Channel 1 |
| Zone 2 | Living Room | Aqara W500 #2 | Channel 2 |
| Zone 3 | Bathroom (if independent loop confirmed) | Aqara W500 #3 | Channel 3 |

> **Pre-purchase action required:** Physically verify at the manifold cabinet whether the bathroom has an independent pipe loop before ordering the 3rd W500. If the bathroom shares the Living Room loop, Zone 2 covers both and no 3rd W500 is needed.

---

## 3. Boiler & OpenTherm Strategy

### 3.1 Boiler: Viessmann Vitodens 222-F

The **Vitodens 222-F** is a condensing combi/system hybrid with an integrated stainless steel DHW cylinder. It is the correct model for this installation.

**Key specifications relevant to this design:**
- Native **OpenTherm interface** (requires Viessmann OpenTherm Extension Module — see below)
- Minimum modulation: **~1.9 kW** — enables true "slow and low" operation
- Inox-Radial stainless heat exchanger — long service life
- Widely serviced across Poland (dense Viessmann dealer network)

> **Critical pre-purchase action:** A **heat loss calculation (PN-EN 12831)** must be completed before selecting the kW output variant. For a property of this size (Main House + Guest House), expect 24–35 kW, but do not guess — an oversized boiler will short-cycle even with OpenTherm modulation. Commission a Polish HVAC engineer to perform the calc.

**Installer instruction (mandatory):**
> "Do NOT connect the boiler to VitoConnect, Vitotronic, or any Viessmann cloud gateway. Leave the OpenTherm terminals accessible. I will be connecting a third-party OpenTherm controller."

### 3.2 OpenTherm Extension Module

The Vitodens 222-F does not expose OpenTherm natively on standard terminals — it requires the **Viessmann OpenTherm Extension Module** (accessory, ordered separately). This connects internally via ribbon cable and exposes a 2-wire OpenTherm bus terminal.

### 3.3 OpenTherm Gateway: Olimex ESP32-POE-ISO + DIYLESS Master Shield

This is the bridge between the OpenTherm bus and Home Assistant.

| Component | Role |
| :--- | :--- |
| **Olimex ESP32-POE-ISO** | ESP32 microcontroller, **PoE-powered** (no wall PSU), **3000V galvanic isolation** protecting the UniFi switch from boiler electrical noise |
| **DIYLESS OpenTherm Master Shield** | Translates ESP32 GPIO to OpenTherm bus signalling |
| **ESPHome firmware** | Runs on the ESP32; exposes all OpenTherm data points to HA natively |

**Why this over a standalone OTGW:** Galvanic isolation is critical — without it, boiler electrical transients can damage PoE switch hardware. The Olimex ISO model provides 3000V isolation. ESPHome integration is also more maintainable long-term than a separate HACS OTGW integration.

**Wiring:**
```
Viessmann 222-F
  └── [Internal ribbon cable]
        └── Viessmann OT Extension Module
              └── [2-wire OpenTherm bus / twin-core 18/2 bell wire]
                    └── DIYLESS Master Shield
                          └── Olimex ESP32-POE-ISO
                                └── [Cat6 PoE] → USW Pro Max 48 PoE → Home Assistant
```

### 3.4 Boiler Control Strategy: "Slow and Low"

Home Assistant controls the boiler by setting the **flow temperature setpoint** via OpenTherm, rather than switching it on/off. This has two components:

**A. Weather Compensation (Heating Curve)**
- An outdoor Aqara temperature sensor feeds HA with outdoor temp
- HA calculates the required flow temperature using a heating curve:
  - Cold day (−10°C outdoor) → higher flow temp (e.g., 55°C)
  - Mild day (+10°C outdoor) → lower flow temp (e.g., 35°C)
  - The exact curve is tuned during commissioning based on the property's heat loss characteristics
- This alone reduces gas consumption significantly versus fixed flow temp

**B. Load Compensation (TRV Demand Feedback)**
- Aqara W600 TRVs report their **valve opening percentage** back to HA via Thread/Matter
- HA aggregates demand across all W600s: if average opening is low (rooms nearly at setpoint), HA reduces the flow temp setpoint sent to the boiler
- If W600s are wide open (rooms cold, high demand), HA raises the flow temp setpoint
- This prevents the boiler from overheating rooms that are already warm

**Combined effect:** The boiler modulates its burner output precisely to what is needed, running at low flame for long periods rather than cycling on/off at full power. Condensing efficiency is maximised, boiler wear is minimised.

**ESPHome OpenTherm data points exposed to HA:**
- `ch_setpoint` — flow temperature setpoint (HA writes this)
- `boiler_temperature` — actual flow temperature (read)
- `relative_modulation_level` — burner modulation % (read, for monitoring)
- `flame_status` — burner on/off (read)
- `fault_indication` — boiler fault flag (read, triggers HA alert)
- `outside_temperature` — if boiler has outdoor sensor fitted (read)
- `dhw_setpoint` — domestic hot water setpoint (HA can manage this too)

---

## 4. Heating & Cooling Architecture

### 4.1 Zone Schedules

| Zone | Target Temp | Schedule | Notes |
| :--- | :--- | :--- | :--- |
| Living Room | **21°C** | 24/7 constant | UFH thermal mass — no setback. Floor slab must stay warm. |
| Bathroom | **21°C base / 23°C boost** | Base 24/7; boost 7–9am & 7–9pm | Rad boost during morning/evening routines. UFH maintains tile warmth. |
| Alex (bedroom) | **21°C / 18°C** | 21°C: 7pm–8am; 18°C: 8am–7pm | Setback during school hours |
| Vicky (bedroom) | **21°C / 18°C** | 21°C: 7pm–8am; 18°C: 8am–7pm | Setback during school hours |
| Playroom | **21°C / 18°C** | 21°C: 2pm–9pm; 18°C: outside these hours | Active only during occupied window |
| Master Bedroom | **21°C / 18°C** | 21°C: 7pm–8am; 18°C: 8am–7pm | Setback during day |
| Office | **21°C** | Occupancy-based (manual or presence) | Gree/LG unit, independent |
| Guest House (Occupied) | **Follows main house** | Same schedules and targets as main house | Triggered via `guest_house_mode = Occupied` |
| Guest House (Unoccupied) | **18°C constant** | No schedule; constant setback; AC inhibited | Humidity management active daily |
| Guest House (Frost Protection) | **7°C minimum** | Absolute minimum; no AC for temp; boiler only | Humidity management active; long absences |

### 4.2 Operating Modes

Two independent `input_select` entities control the system. They are orthogonal — season mode affects *how* systems heat/cool, house modes affect *whether* they run at all and at what target.

**Season Mode** — whole property, controls direction of heating/cooling:
```
input_select.season_mode:
  options:
    - Heating   # Boiler + UFH + rads active; MDV cooling inhibited
    - Cooling   # UFH off (floor protection only); MDV cooling active; boiler inhibited
    - Neutral   # Neither unless room is >2°C from setpoint
```

**Main House Mode** — controls main house comfort level:
```
input_select.main_house_mode:
  options:
    - Normal           # All schedules and setbacks run as designed
    - Frost Protection # All zones → 7°C minimum; all schedules suspended;
                       # MDV inhibited; boiler fires only to prevent freezing
```

**Guest House Mode** — controls guest house independently:
```
input_select.guest_house_mode:
  options:
    - Occupied         # Follows same schedules/targets as main house
    - Unoccupied       # 18°C constant; AC inhibited; humidity management active
    - Frost Protection # 7°C minimum; AC inhibited; humidity management active
```

Mode transitions:
- **Season mode:** Automatic via 24h outdoor temp average with 48h guard + hysteresis; manual override always available
- **House modes:** Manual only — set via Shelly Wall Display, UniFi Connect panel, or HA Companion App

---

## 5. UFH Manifold & Wiring Centre

### 5.1 Existing Salus Wiring Centre — To Be Evaluated

The existing **Salus wiring centre** currently:

- Receives volt-free call-for-heat signals from zone thermostats.
- Drives 230V manifold actuators (motorised valves on UFH pipe loops).
- Signals the boiler to fire when any zone is calling.

The plan is to **reuse** this wiring centre with Aqara W500 thermostats, which also provide volt-free contacts. However, this must be **validated with the installer**, and the pros and cons of replacing it with the installer’s recommended wiring solution must be evaluated before finalising:

- **Reuse Salus:**
  - Pro: minimal rewiring, lower cost, known working behaviour.
  - Con: may limit future zone expansion or diagnostics compared to a modern wiring centre.
- **Replace per installer recommendation:**
  - Pro: supported, documented wiring, possibly better integration with new boiler/manifold layout.
  - Con: higher cost and more invasive work.

> **Decision:** **To be taken with the heating installer during design/commissioning.** Until then, this document assumes reuse of the Salus wiring centre, but this is explicitly subject to change.

### 5.2 Zone Channel Mapping

| Wiring Centre Channel | Old Thermostat | New Thermostat | Zone |
| :--- | :--- | :--- | :--- |
| Channel 1 | Salus wired stat | **Aqara W500 #1** | Guest House UFH |
| Channel 2 | Salus wired stat | **Aqara W500 #2** | Living Room UFH |
| Channel 3 | Salus wired stat (if present) | **Aqara W500 #3** (if loop confirmed) | Bathroom UFH |

### 5.3 Radiator Circuit Channel — Modified

Previously the radiator zone was driven by a single wireless Salus thermostat covering all radiators. With W600 TRVs installed on every radiator, individual room thermostats are no longer needed. The radiator circuit channel on the wiring centre is rewired as follows:

- The channel output is wired through a **Shelly Plus 1** (DIN rail, IoT VLAN WiFi)
- HA controls this Shelly as a **"Radiator Circuit Enable"** relay
- When any W600 reports demand (valve opening), HA closes the relay → wiring centre fires the radiator zone → boiler gets a call for heat on the rad circuit
- When no W600 has demand (summer mode, or all rooms at setpoint), HA opens the relay → radiator circuit is fully inhibited

This gives a clean master shutoff for the entire radiator circuit without touching wiring centre internals.

### 5.4 Aqara W500 — Key Properties

- Zigbee/Matter wall thermostat
- Volt-free relay output for UFH control
- Displays current and target temperature
- Pairs directly with Aqara M3 Hub
- Supports HA climate entity natively
- Physical rotary control for manual override

---

## 6. Radiator Control (W600 TRVs)

### 6.1 Aqara W600 — Key Properties

- Thread/Matter radiator TRV head
- Reports **valve opening percentage** back to HA — this is the critical data point for load compensation
- Replaces existing thermostatic TRV heads on each radiator
- Pairs with Aqara M3 Hub (Matter/Thread border router)
- Supports HA climate entity natively

### 6.2 Radiator Inventory

| Location | Quantity | Wall Sensor | Notes |
| :--- | :--- | :--- | :--- |
| Bathroom | 2x W600 | W500 (built-in) | Boost to 23°C during comfort windows |
| Alex (bedroom) | 1x W600 | W100 | Setback schedule |
| Vicky (bedroom) | 1x W600 | W100 | Setback schedule |
| Playroom | 1x W600 | W100 | Active 2pm–9pm schedule |
| Master Bedroom | 2x W600 | T1 | Setback schedule; MDV also serves this room |
| Office | 1x W600 | T1 | Occupancy-based schedule; Gree/LG also serves this room |

**Total W600 count: 8 minimum.** Confirm during site survey — if any room has additional radiators not listed, add one W600 per head.

> **Important:** W600 built-in sensors read temperature near the radiator, which is not representative of room temperature. HA automations must use the dedicated wall sensor (W100 or T1) as the primary temperature source for each room. The W600 sensor is useful only for valve position feedback and load compensation logic.

### 6.3 W600 Demand Aggregation Logic (HA)

```
Every 5 minutes:
  rad_demand_pct = average(W600_valve_opening across all active W600s)

  IF rad_demand_pct > 60%:
    → Raise boiler flow temp setpoint by 2°C (up to max curve value)
  ELIF rad_demand_pct < 20%:
    → Lower boiler flow temp setpoint by 2°C (down to minimum ~35°C)
  ELSE:
    → Hold current setpoint

  IF rad_demand_pct > 0%:
    → Shelly Plus 1 (radiator circuit enable) = ON
  ELSE:
    → Shelly Plus 1 = OFF
```

---

## 7. MDV Duct AirCon Integration

### 7.1 Scope

The MDV shared duct system serves **5 rooms**:
- Living Room
- Alex (bedroom)
- Vicky (bedroom)
- Playroom
- Master Bedroom

There is **no individual room regulation** — the MDV system is shared on/off with mode control (heat/cool/fan). Individual room comfort is achieved by combining MDV with the room's primary heat source (UFH or rads) and using Aqara temp sensors to decide when MDV should run.

### 7.2 Integration Path: midea-local (LAN, no cloud)

MDV is the commercial HVAC brand of **Midea**. The MDV units accept a compatible WiFi dongle. Once fitted:

1. The dongle joins the **IoT VLAN** (192.168.12.x)
2. The **`midea-local`** Home Assistant integration (HACS) discovers the unit on LAN
3. No Midea cloud account or app required — fully local
4. HA sees a single `climate` entity for the MDV system

### 7.3 MDV/UFH Interlock — Living Room

The Living Room sits in **both** the UFH zone (W500 #2) and the MDV zone. A mandatory HA automation enforces the interlock:

```
HEATING SEASON:
  IF Living Room UFH is actively heating (W500 #2 calling for heat):
    → MDV is restricted to FAN ONLY mode (air circulation, no heating, no cooling)
    → Rationale: cooling while heating wastes energy and fights the slab;
      heating-on-heating risks overheating the Living Room
  ELSE (UFH slab is at temp, no active call for heat):
    → MDV heating is PERMITTED if any MDV room is below its target temp
    → MDV cooling remains INHIBITED (still heating season — never cool while boiler is running)

COOLING SEASON:
  → UFH is OFF (W500 #2 setpoint = floor protection minimum, ~15°C — slab won't heat)
  → MDV heating mode is INHIBITED
  → MDV cooling runs freely for all 5 rooms based on demand
  → No conflict (UFH will not fire in cooling season)

NEUTRAL SEASON:
  → Neither heating nor cooling runs unless a room is >2°C from setpoint
  → If heating is needed: boiler + UFH/rads only; MDV heating not used in neutral season
  → If cooling is needed: MDV cooling only; UFH remains off
```

### 7.4 MDV Demand Logic

Since all 5 MDV rooms share one system, HA uses the **coldest room vs. target** as the demand signal for heating and the **hottest room vs. target** for cooling.

**Dead-band thresholds (asymmetric by design):**

| Mode | Turn ON when | Turn OFF when | Rationale |
| :--- | :--- | :--- | :--- |
| Heating | any room < target − **1.0°C** | all rooms ≥ target − 0.2°C | Tight — heating is the primary comfort function, don't allow rooms to drift cold |
| Cooling | any room > target + **3.0°C** | all rooms ≤ target + 0.5°C | Relaxed — slight warmth is tolerable; cooling is expensive and noisy |

**Mode-swap grace period — no direct heating↔cooling transitions:**

The MDV unit must never switch directly from heating to cooling or vice versa. A compressor reversal without a rest period causes mechanical wear and may trip the unit's protection circuit. HA enforces a mandatory grace period:

```
input_datetime.mdv_last_mode_change  # timestamp of last heat/cool mode switch

RULE: MDV cannot change between HEAT and COOL modes
      unless (now - mdv_last_mode_change) > 30 minutes

Transition path when mode needs to change:
  HEAT → [FAN ONLY for 30 min grace] → COOL
  COOL → [FAN ONLY for 30 min grace] → HEAT
  FAN ONLY does not reset the grace timer — only HEAT↔COOL transitions do
```

**Full demand logic:**

```
# Evaluate every 5 minutes

mdv_heating_demand =
  any MDV room where current_temp < (target_temp - 1.0°C)
  AND season_mode = HEATING
  AND Living Room UFH NOT actively calling for heat
  AND (now - mdv_last_mode_change) > 30 min  [if last mode was COOL]

mdv_cooling_demand =
  any MDV room where current_temp > (target_temp + 3.0°C)
  AND season_mode = COOLING
  AND (now - mdv_last_mode_change) > 30 min  [if last mode was HEAT]

IF mdv_heating_demand AND current_mdv_mode != COOL:
  → MDV ON, mode = HEAT
  → Record mdv_last_mode_change if mode changed from COOL

ELIF mdv_cooling_demand AND current_mdv_mode != HEAT:
  → MDV ON, mode = COOL
  → Record mdv_last_mode_change if mode changed from HEAT

ELIF mdv_heating_demand AND current_mdv_mode = COOL:
  → MDV mode = FAN ONLY  (grace period — compressor rest, no heating yet)

ELIF mdv_cooling_demand AND current_mdv_mode = HEAT:
  → MDV mode = FAN ONLY  (grace period — compressor rest, no cooling yet)

ELSE (no demand):
  → MDV OFF
```

---

## 8. Guest House AirCon (IR Control)

### 8.1 Hardware: Aqara M200 Hub (PoE)

The old AirCon units in Guest House Room 1 and Room 2 do not have smart connectivity. They are controlled via **IR blasters** using the **Aqara M200 Hub**, which has a built-in IR transmitter.

- **2x Aqara M200 Hubs** — one per Guest House room, PoE-powered
- Joins Aqara Zigbee/Matter ecosystem — no new ecosystem introduced
- HA sends IR commands via the M200 to replicate remote control actions
- Pairs with the Aqara M3 Hub

### 8.2 Limitations & Mitigation

IR control has no state feedback — HA cannot confirm the AC unit turned on or what mode it's in. Mitigation:

- HA maintains a **virtual state** for each AC (tracks last command sent)
- Aqara room temp sensors provide indirect feedback — if room temp moves toward setpoint after an IR ON command, assume it worked
- **Interlock:** HA automation prevents IR AC and UFH from running simultaneously in the same Guest House room

### 8.3 Guest House Occupancy Mode

The Guest House operates on a dedicated `input_select.guest_house_mode` with three states:

| Mode | UFH Setpoint | AC | Humidity Management | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Occupied** | Follows main house schedule (21°C / setbacks) | Available, same logic as main house | Standard | Guests treated as full occupants — same comfort targets and season mode apply |
| **Unoccupied** | 18°C constant setback | Inhibited (temperature) | Active (see below) | House is empty but maintained |
| **Frost Protection** | 7°C minimum — pipes safe | Inhibited (temperature) | Active (see below) | Long absence in winter — absolute minimum energy use |

```
input_select.guest_house_mode:
  options: [Occupied, Unoccupied, Frost Protection]
```

**Occupied mode detail:** When set to Occupied, the Guest House UFH zone follows the same season mode and schedule logic as the main house. The old AirCon units (IR via M200) are available for heating and cooling on the same demand thresholds. The Guest House is treated as a full zone of the property.

**Toggle locations:** Shelly Wall Display in Guest Room 2, UniFi Connect panel, HA Companion App.

---

## 9. Office AirCon (Gree/LG)

The Office has a smart AirCon unit (Gree or LG — confirm model). This is integrated via the **Gree integration** (built into Home Assistant core — no HACS required).

- Integration operates **LAN-local** — no cloud dependency
- Unit must be on the **IoT VLAN** (192.168.12.x)
- HA sees a `climate` entity for the Office AC
- Schedule: 21°C when office is occupied; setback to 18°C otherwise
- Occupancy can be presence-based (HA companion app on phone) or time-scheduled

> **Action:** Confirm whether unit is Gree or LG. If LG, use the **LG ThinQ integration** (available in HA, also supports LAN-local for compatible models).

---

## 10. Sensor Strategy

### 10.1 Temperature Sensors — Aqara TVOC T1 (Zigbee)

One sensor per room for room temperature feedback to HA. These drive the demand logic for UFH, radiators, MDV, and IR AC.

| Location | Wall Sensor | Primary HA Temp Source | Notes |
| :--- | :--- | :--- | :--- |
| Living Room | W500 #2 (built-in) | W500 built-in | UFH demand + MDV interlock logic |
| Bathroom | W500 #3 (built-in, if loop independent) | W500 built-in | UFH + rad boost logic |
| Alex (bedroom) | W100 (display sensor) | W100 | Rad demand + MDV demand; display useful for kids |
| Vicky (bedroom) | W100 (display sensor) | W100 | Rad demand + MDV demand; display useful for kids |
| Playroom | W100 (display sensor) | W100 | Rad demand + MDV demand; display for quick check |
| Master Bedroom | T1 (standalone) | T1 | Rad demand + MDV demand; W600 built-in NOT used as primary |
| Office | T1 (standalone) | T1 | Rad + Gree/LG demand; discreet, no display needed |
| Guest House Room 1 | T1 (standalone) | T1 | UFH indirect feedback + IR AC trigger |
| Guest House Room 2 | T1 (standalone) | T1 | UFH indirect feedback + IR AC trigger |
| Outdoor | T1 (outdoor rated) | T1 outdoor | Weather compensation heating curve input |

> **Critical:** W600 TRV built-in sensors must never be used as the primary room temperature source in HA automations. They read near the radiator and will be several degrees higher than actual room temperature when the rad is firing. Always use the dedicated wall sensor (W100 or T1) as `sensor.room_temperature` in all automations and climate entities.

> **Humidity:** The Aqara T1, W100, and W500 all report **temperature and humidity**. No additional humidity sensors are needed. The Guest House T1 sensors in Room 1 and Room 2 provide the humidity readings used by the Unoccupied and Frost Protection humidity management automations (Automation 11). Ensure HA exposes `sensor.guest_room_1_humidity` and `sensor.guest_room_2_humidity` from the T1 devices.

### 10.2 W100 Display Sensor

The **Aqara W100** combines a temperature/humidity sensor with a small e-ink display. Use in rooms where visibility of current temp is useful but a full thermostat is not needed (Playroom, Alex, Vicky). Stays in the Aqara ecosystem.

---

## 11. Schedule & Automation Logic

### 11.1 HA Entities Overview

```yaml
input_select:
  season_mode:
    options: [Heating, Cooling, Neutral]
    # Controls the whole-property heating/cooling direction
    # Transitions guarded by 48h stability + hysteresis (see Automation 1)

  guest_house_mode:
    options: [Occupied, Unoccupied, Frost Protection]
    # Controls Guest House operating mode independently of main house

  main_house_mode:
    options: [Normal, Frost Protection]
    # Normal = schedules run as designed
    # Frost Protection = all zones drop to minimum safe temp (7°C)

input_boolean:
  rad_circuit_enable: # Mirrors Shelly Plus 1 state (radiator circuit master relay)

input_number:
  boiler_flow_setpoint:     # Current OT setpoint written to boiler (35–60°C)
  outdoor_temp_avg_24h:     # Rolling 24h average of outdoor sensor
  frost_temp_main:          # Frost protection minimum for main house (default: 7°C)
  frost_temp_guest:         # Frost protection minimum for guest house (default: 7°C)
  humidity_threshold_high:  # Humidity % above which dehumidify cycle triggers (default: 70%)
  humidity_run_minutes:     # Duration of AC dehumidify cycle in minutes (default: 30)

input_datetime:
  mdv_last_mode_change:       # Timestamp of last MDV heat↔cool mode transition
  guest_ac_r1_last_dehumid:   # Last dehumidify run timestamp — Guest Room 1
  guest_ac_r2_last_dehumid:   # Last dehumidify run timestamp — Guest Room 2

input_text:
  mdv_last_mode:         # Tracks last active MDV mode: HEAT / COOL / FAN / OFF
                         # Needed because midea-local may not reliably report
                         # previous mode after an OFF state
```

### 11.2 Key Automations

**Automation 1: Season Mode Auto-Transition**
```
Trigger: outdoor_temp_avg_24h changes (evaluated on 24h rolling average — dampens
         day-to-day fluctuation)

Guard: season_mode must have been stable for ≥ 48 hours before any auto-transition
       (prevents oscillation on mild days straddling a threshold)
       Manual override always permitted immediately via UI

Thresholds use hysteresis — different values for entering vs. leaving a mode:

  Entering Heating:  avg < 10°C  (conservative entry — definitely cold)
  Leaving Heating:   avg > 14°C  (generous exit — don't leave heating too early)

  Entering Cooling:  avg > 20°C  (conservative entry — definitely hot)
  Leaving Cooling:   avg < 16°C  (generous exit — don't leave cooling too early)

  Neutral zone sits between 14°C and 20°C

Transition rules:
  Heating → Cooling:  MUST pass through Neutral (cannot jump directly)
  Cooling → Heating:  MUST pass through Neutral (cannot jump directly)
  Neutral → either:   Permitted when threshold met AND 48h stability guard passed

Note: season_mode is independent of main_house_mode and guest_house_mode.
  A house in Frost Protection still has a season_mode (determines whether
  the boiler or AC would be used if frost temp is breached).
  When season_mode changes, set input_datetime.mdv_last_mode_change = now
  to force MDV through its 30-min grace period.
```

**Automation 2: Boiler Flow Temp (Weather Compensation)**
```
Trigger: outdoor temperature sensor updates (every 15 min)
Action:
  flow_temp = heating_curve(outdoor_temp)
  # Example curve: flow = 70 - (2.0 * outdoor_temp), clamped 35–60°C
  Write flow_temp to OT setpoint via ESPHome
```

**Automation 3: Load Compensation (TRV Feedback)**
```
Trigger: any W600 valve_position changes
Action:
  avg_opening = mean(all W600 valve positions)
  Adjust flow_temp setpoint ±2°C per cycle (see Section 6.3)
```

**Automation 4: Radiator Circuit Enable**
```
Trigger: any W600 valve_position changes
Action:
  IF any W600 valve_position > 5%:
    → Shelly Plus 1 (rad circuit) = ON
  ELSE:
    → Shelly Plus 1 (rad circuit) = OFF
```

**Automation 5: MDV/UFH Living Room Interlock**
```
Trigger: W500 #2 (Living Room) call_for_heat state changes
         OR season_mode changes

Action:
  IF season_mode = Heating AND call_for_heat = True:
    → MDV mode = FAN ONLY (no heating, no cooling — circulation only)
  ELIF season_mode = Heating AND call_for_heat = False:
    → MDV heating PERMITTED (UFH slab satisfied, MDV can top up other rooms)
    → MDV cooling still INHIBITED (never cool during heating season)
  ELIF season_mode = Cooling:
    → MDV cooling PERMITTED freely
    → MDV heating INHIBITED
    → UFH setpoint = 15°C floor protection (will not actively heat)
  ELIF season_mode = Neutral:
    → MDV heating and cooling both INHIBITED unless room >2°C from setpoint
```

**Automation 6: Bathroom Rad Boost**
```
Trigger: Time (7:00am, 7:00pm)
Action:
  → Set W600 bathroom x2 setpoint to 23°C for 2 hours
  → After 2 hours: revert to 21°C
```

**Automation 7: Boiler Fault Alert**
```
Trigger: ESPHome fault_indication = True
Action:
  → HA notification to mobile (all adults)
  → Log to Unraid/MariaDB
  → Reduce flow setpoint to frost-safe minimum (35°C) — keeps house from freezing
     even if the fault is non-critical
```

**Automation 8: Main House Frost Protection**
```
Trigger: main_house_mode changes to "Frost Protection"
         OR (main_house_mode = Frost Protection AND any room temp < frost_temp_main)

Action when entering Frost Protection:
  → All W500 UFH setpoints → frost_temp_main (default 7°C)
  → All W600 TRV setpoints → frost_temp_main
  → MDV: INHIBITED (neither heating nor cooling — frost protection via wet system only)
  → Boiler flow temp: minimum curve value (35°C) — runs only if zone calls
  → All schedules and setbacks suspended
  → HA notification: "Main house set to Frost Protection"

Action when leaving Frost Protection (back to Normal):
  → Restore all setpoints to scheduled values
  → Re-enable MDV and schedule logic
  → HA notification: "Main house returned to Normal mode"

Note: Frost Protection does NOT override the boiler's own anti-freeze function.
The boiler has a hardware anti-freeze cycle that fires independently at ~3°C flow temp.
The HA frost protection layer acts earlier (7°C room temp) to prevent the boiler
anti-freeze from ever needing to activate.
```

**Automation 9: Guest House Frost Protection**
```
Trigger: guest_house_mode changes to "Frost Protection"
         OR (guest_house_mode = Frost Protection AND
             any guest room temp < frost_temp_guest)

Action when entering Frost Protection:
  → Guest House W500 #1 setpoint → frost_temp_guest (default 7°C)
  → Guest House IR AC: INHIBITED for temperature control
  → Humidity management: ACTIVE (see Automation 11)
  → HA notification: "Guest house set to Frost Protection"

Action when leaving:
  → Restore to Unoccupied mode setpoint (18°C) or Occupied if toggled
  → Re-enable IR AC if appropriate
```

**Automation 10: Guest House Unoccupied Mode**
```
Trigger: guest_house_mode changes to "Unoccupied"

Action:
  → Guest House W500 #1 setpoint → 18°C (constant, no schedule)
  → Guest House IR AC: INHIBITED for temperature control
  → Humidity management: ACTIVE (see Automation 11)
  → HA notification: "Guest house set to Unoccupied"
```

**Automation 11: Humidity Management (Unoccupied & Frost Protection)**
```
Applies to: Guest House rooms when guest_house_mode = Unoccupied OR Frost Protection
            Can be extended to main house rooms if main_house_mode = Frost Protection

Trigger: Time (once daily, 10:00am — chosen to avoid cold start in early morning)

Condition check per room:
  IF room_humidity > humidity_threshold_high (default 70%):
    AND last_dehumidify_run > 23 hours ago (prevents multiple runs per day)
    AND season_mode != Heating (do not run AC dehumidify while boiler is heating —
        condensation management is less critical when the house is warm)

  Action:
    → Send IR command: AC ON, mode = DRY (dehumidify), fan = AUTO
    → Wait humidity_run_minutes (default 30 min)
    → Send IR command: AC OFF
    → Record timestamp to guest_ac_rX_last_dehumid

  ELIF room_humidity > 80% (emergency threshold — override season restriction):
    → Run dehumidify cycle regardless of season_mode
    → HA notification: "High humidity alert — Guest Room X dehumidifying"

Rationale:
  Empty rooms in cold/damp climates accumulate humidity rapidly — mould risk
  is significant in Polish winters in unheated spaces. Running AC in DRY mode
  for 30 min/day draws moisture out of the air without meaningfully cooling the
  room or fighting the UFH. The 70% threshold is conservative; mould risk
  begins around 75–80% sustained humidity.

Limitations (IR control):
  AC DRY mode is sent as an IR command — no feedback on whether the unit
  executed it. If the Aqara T1 sensor shows humidity dropping after the run,
  assume success. If not, HA can log a warning for manual inspection.
```

### 11.3 Presence & Occupancy

- **Children's rooms:** Time-schedule based (school hours)
- **Playroom:** Time-schedule based (2pm–9pm active)
- **Office:** HA companion app presence detection OR time-schedule
- **Guest House:** `input_select.guest_house_mode` — Occupied / Unoccupied / Frost Protection
- **Master Bedroom:** Time-schedule (setback 8am–7pm)
- **Main House:** `input_select.main_house_mode` — Normal / Frost Protection

---

## 12. Network Architecture

### 12.1 Core Hardware

| Device | Role | Uplink | Location |
| :--- | :--- | :--- | :--- |
| **UDM SE** | Main security gateway, controller | Fiber/Cable WAN | Garage |
| **UniFi Aggregation Switch (SFP)** | Core 10G SFP fabric — interconnects UDM SE, USW Pro, Unraid | 10G SFP+ to UDM SE | Garage |
| **USW Pro Max 48 PoE** | PoE distribution — powers all PoE endpoints | 10G SFP+ to Aggregation | Garage |
| **Unraid Server (Ryzen 5900X, 100TB)** | MariaDB, backups, telemetry, Frigate | 10G SFP+ to Aggregation | Garage |
| **USW XG 10 PoE** | AP uplink switch — powers U7 Pro XG APs via PoE | 10G SFP+ to Aggregation | Distributed / Garage |
| **U7 Pro XG APs** | WiFi 7 coverage + Thread mesh (Ch.25) | PoE from USW XG 10 PoE | Distributed |
| **Raspberry Pi 5 (PoE+, NVMe SSD)** | Home Assistant host | PoE+ from USW Pro Max 48 | Garage rack |
| **Aqara M3 Hub** | Zigbee coordinator + Matter/Thread border router | PoE from USW Pro Max 48 | Garage / Central |
| **Aqara M200 Hubs x2** | IR control for Guest House ACs | PoE from USW Pro Max 48 | Guest House rooms |
| **Olimex ESP32-POE-ISO** | OpenTherm gateway (ESPHome) | PoE from USW Pro Max 48 | Engine Room |

For a full description of the network topology, VLANs, WiFi/Thread configuration, and security rules, see:

- [`networking-master-plan.md`](networking-master-plan.md)

### 12.2 VLAN Design

| Network | VLAN ID | Subnet | Devices | WAN |
| :--- | :--- | :--- | :--- | :--- |
| **Core** | 1 | `192.168.10.x` | UDM SE, switches, APs (management) | Yes |
| **Server** | 6 | `192.168.1.x` | Unraid Server | Yes |
| **IoT** | 3 | `192.168.12.x` | Raspberry Pi 5 (HA), Olimex gateway, M3 Hub, M200 Hubs, Shelly devices, MDV dongle, Gree/LG unit | Restricted |
| **NOT** | 4 | `192.168.13.x` | Non-internet things (WAN blocked) | **No** |

**IoT VLAN rules:**
- Devices can communicate with HA (Pi 5) on same VLAN
- WAN access restricted — only allowed for OTA updates (specific IP/domain whitelist)
- No access to Core or Server VLANs (unidirectional — HA on IoT can query Unraid on Server VLAN, not reverse)

### 12.3 Thread Mesh Configuration

- Thread runs on **Channel 25** — chosen to minimise overlap with 2.4GHz WiFi bands
- Aqara M3 acts as Thread Border Router
- W600 TRVs and W500 thermostats join Thread mesh natively
- U7 Pro XG APs maintain Thread channel discipline — configure in UniFi settings

### 12.4 Connectivity Map

```
[Fiber/Cable WAN]
    └── UDM SE (Gateway)
          └── [10G SFP+] UniFi Aggregation Switch (SFP)
                ├── [10G SFP+] USW Pro Max 48 PoE
                │              ├── [PoE+]  Raspberry Pi 5 (Home Assistant)
                │              ├── [PoE]   Aqara M3 Hub
                │              ├── [PoE]   Olimex ESP32-POE-ISO (Engine Room, OpenTherm)
                │              └── [PoE]   Aqara M200 x2 (Guest House)
                ├── [10G SFP+] USW XG 10 PoE
                │              └── [PoE]   U7 Pro XG APs (all access points)
                └── [10G SFP+] Unraid Server (Ryzen 5900X, 100TB)
```

---

## 13. Control & Display Hierarchy

A layered control hierarchy prevents app fatigue and keeps the system guest-accessible.

| Layer | Device | Location | Role |
| :--- | :--- | :--- | :--- |
| 1 (Primary) | UniFi Connect 21" | Living Room | "Command Tower" — full system overview, scene control, guest-friendly |
| 2 (Tactical) | Shelly Wall Display XL (10.1") | Couch area, Upstairs, Guest Room 2 | Home Assistant mode — fast local control without opening an app |
| 3 (Room) | Aqara W500 | Living Room, Guest House, Bathroom | Room thermostat — physical rotary setpoint adjustment |
| 3 (Room) | Aqara W600 | Each radiator | TRV — manual temperature adjustment at the rad |
| 3 (Room) | Aqara W100 | Playroom, Alex, Vicky | Display + sensor — see temp, no control needed |
| 4 (Remote) | HA Companion App | Mobile | Full remote access when away from home |

---

## 14. Hardware Bill of Materials

### Boiler & OpenTherm

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Viessmann Vitodens 222-F (kW TBD) | 1 | Viessmann dealer, Komfort.pl, Instalcompact.pl |
| Viessmann OpenTherm Extension Module | 1 | Viessmann dealer (order with boiler) |
| Olimex ESP32-POE-ISO | 1 | Botland.pl, Nettigo.pl, Olimex direct |
| DIYLESS OpenTherm Master Shield | 1 | diyless.pl, Nettigo.pl |

### Thermostats & TRVs

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Aqara W500 UFH Thermostat | 2 (or 3 if bathroom loop confirmed independent) | x-kom.pl, Allegro, Media Expert |
| Aqara W600 Radiator TRV | 8 minimum (survey required — 1 per rad head) | x-kom.pl, Allegro |
| Aqara W100 Display Sensor | 3 (Alex, Vicky, Playroom) | x-kom.pl, Allegro |

### Sensors

| Item | Qty | Rooms | Supplier (Poland) |
| :--- | :--- | :--- | :--- |
| Aqara Temperature Sensor T1 (indoor) | 4 | Master Bedroom, Office, Guest Room 1, Guest Room 2 | x-kom.pl, Allegro, Media Expert |
| Aqara Temperature Sensor T1 (outdoor rated) | 1 | Outdoor (weather compensation) | x-kom.pl, Allegro |

### Switching & Control

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Shelly Plus 1 (DIN rail) | 1 (rad circuit enable) | Botland.pl, Nettigo.pl |
| Shelly Wall Display XL | 3 (couch, upstairs, guest room 2) | Botland.pl, Nettigo.pl |

### AirCon Integration

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Aqara M200 Hub (PoE) | 2 (Guest House rooms) | x-kom.pl, Allegro |
| MDV/Midea WiFi Dongle (compatible) | 1 | MDV/Midea HVAC distributor |

### Compute & Network (if not already owned)

| Item | Qty | Notes |
| :--- | :--- | :--- |
| Raspberry Pi 5 (8GB, PoE+ HAT, NVMe HAT + SSD) | 1 | If not already in place |
| Aqara M3 Hub | 1 | If not already in place |

---

## 15. Risks & Critical Maintenance Notes

| Risk | Mitigation |
| :--- | :--- |
| **Galvanic isolation** | Olimex ESP32-POE-ISO provides 3000V isolation. Do not substitute with a non-ISO ESP32 board — boiler electrical transients will damage the UniFi PoE switch port. |
| **Database write wear** | HA Recorder is configured to write to **MariaDB on Unraid** (not the Pi 5 NVMe). Configure in `configuration.yaml`: `recorder: db_url: mysql://...`. Do not leave default SQLite on Pi. |
| **MDV/UFH conflict** | The Living Room MDV/UFH interlock automation (Section 11.2, Automation 5) is mandatory. Test this thoroughly before going live — a misconfigured interlock means the floor heats while the AC cools simultaneously. |
| **Thread channel conflict** | Thread must run on Channel 25. Verify U7 Pro XG APs are not advertising 2.4GHz WiFi on channels that overlap (channels 1–6 are safe; 11 overlaps with Thread Ch.25 slightly — use Ch.1 or Ch.6 for 2.4GHz WiFi). |
| **Installer ecosystem creep** | Explicitly tell the Viessmann installer not to fit VitoConnect or any Viessmann cloud module. Some installers fit these by default. It will complicate the OpenTherm setup. |
| **W600 battery life** | W600 TRVs are battery-powered. Set up a HA automation to alert when any W600 battery drops below 20%. A dead TRV means a stuck valve. |
| **Boiler sizing** | Do not proceed to purchase before heat loss calc. An oversized boiler will short-cycle. An undersized boiler won't meet demand on design days. |
| **Bathroom manifold loop** | If the bathroom shares the Living Room UFH loop and a W500 is incorrectly installed thinking it's independent, you'll have two thermostats fighting one loop. Verify physically at the manifold cabinet before ordering. |

---

## 16. Open Items & Pre-Purchase Decisions

| # | Item | Owner | Status |
| :--- | :--- | :--- | :--- |
| 1 | Heat loss calculation (PN-EN 12831) — determines boiler kW | HVAC engineer | **Blocking — must complete before boiler order** |
| 2 | Bathroom UFH loop independence — verify at manifold cabinet | Owner / plumber | **Blocking — determines W500 qty** |
| 3 | Guest House boiler topology — shared circuit with main house or separate boiler? | Owner / HVAC engineer | **Blocking — affects manifold and pipe design** |
| 4 | Confirm Salus wiring centre zone count — does it have 3 independent zone outputs? | Owner | Quick physical check |
| 5 | Office AirCon — confirm brand/model (Gree vs LG) | Owner | Determines HA integration to use |
| 6 | W600 radiator count — full site survey of all rad heads | Owner | Determines W600 purchase qty |
| 7 | MDV WiFi dongle — confirm compatible dongle model with MDV distributor | Owner | Before MDV integration work |
| 8 | midea-local compatibility test — fit dongle, confirm HA discovery on LAN | Owner / HA installer | Before going live |
| 9 | Gree/LG integration LAN-local test — confirm unit responds without cloud | Owner | Close out to-do |
| 10 | Heating curve tuning — set initial curve coefficients at commissioning | HVAC engineer / HA installer | At commissioning |
| 11 | Salus wiring centre vs new wiring centre — validate reuse and compare with installer’s recommendation | Owner / HVAC installer | To be decided during detailed design; plan currently assumes reuse but is flexible |

---

## 17. System Architecture Diagram

```
PHYSICAL LOCATION       EQUIPMENT & CONNECTIVITY
─────────────────────────────────────────────────────────────────────

GARAGE                  [Fiber / Cable WAN]
(Network Core)               │
                        [UDM SE — Security Gateway]
                             │ (10G SFP+)
                        [UniFi Aggregation Switch (SFP)]
                        ┌────┴──────────────┬──────────────────┐
                        │ (10G SFP+)        │ (10G SFP+)       │ (10G SFP+)
                  [USW Pro Max 48 PoE]  [USW XG 10 PoE]  [Unraid Server]
                        │               │                  (MariaDB, 100TB)
          ┌─────────────┼───────┐        └── [PoE] U7 Pro XG APs
          │ (PoE+)  (PoE) (PoE) │                   (WiFi7 + Thread Ch.25)
    [Pi 5 PoE+] [Aqara M3] [M200 x2]
    (Home Asst)  (Zigbee/   (Guest House
                 Thread BR)  IR control)

─────────────────────────────────────────────────────────────────────

ENGINE ROOM             [Viessmann Vitodens 222-F]
(Heat Source)                │ (internal ribbon)
                        [Viessmann OT Extension Module]
                             │ (2-wire OpenTherm bus)
                        [DIYLESS OT Master Shield]
                             │
                        [Olimex ESP32-POE-ISO]  ←── (Cat6 PoE)
                             │ (ESPHome → HA via LAN)
                             │
                        [Salus Wiring Centre]
                        ┌────┴─────────────────────┐
                   Zone 1 (UFH Guest House)    Zone 2 (UFH Living Room)
                   Aqara W500 #1               Aqara W500 #2
                        │                          │
                   [Manifold Actuators]       [Manifold Actuators]
                        │                          │
                   Zone 3 (Rad Circuit Enable) ── Shelly Plus 1 ← HA

─────────────────────────────────────────────────────────────────────

ROOMS / ENDPOINTS       [Aqara M3 Hub] (Thread/Zigbee/Matter)
                             │
          ┌──────────────────┼──────────────────┬──────────────────┐
          │                  │                  │                  │
    [W600 TRVs]        [W500 UFH stats]   [W100 sensors]    [T1 sensors]
    Bath x2             Living Room        Alex (bedroom)    Master Bedroom
    Alex x1             Guest House        Vicky (bedroom)   Office
    Vicky x1            Bathroom (TBC)     Playroom          Guest Room 1
    Playroom x1                                              Guest Room 2
    Master x2                                               Outdoor (weather)
    Office x1

                        [Aqara M200 PoE x2]  (Guest House)
                             │ IR
                        [Old AirCon R1]  [Old AirCon R2]

─────────────────────────────────────────────────────────────────────

AIRCON (WiFi/LAN)       [MDV Duct System + WiFi Dongle]
                             │ (midea-local, LAN, IoT VLAN)
                        Serves: Living Room, Alex, Vicky, Playroom, Master
                        HA interlock: inhibited when Living Room UFH active

                        [Gree/LG Office Unit]
                             │ (Gree / LG ThinQ integration, LAN)
                        Serves: Office only

─────────────────────────────────────────────────────────────────────

DISPLAYS                [UniFi Connect 21"] — Living Room (Command Tower)
                        [Shelly Wall XL x3] — Couch / Upstairs / Guest Room 2
                             (Home Assistant mode, local control)
                        [HA Companion App]  — Mobile (remote access)
```

---

*Document version: 2026-02-26. Review after each open item is resolved. Next scheduled review: post heat loss calculation and manifold survey.*

