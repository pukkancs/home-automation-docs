# Heating & Cooling (PukkancsLak)

*Last updated: 2026-03-15*

Heating, cooling, and climate control. For network and infrastructure see [networking.md](networking.md). For property layout see [property-overview.md](property-overview.md).

---

## Current State


### 1. Boiler

- **Current:** An existing (legacy) boiler is in use.
- The boiler is fired by the Salus wiring centre when any zone calls for heat.

### 2. UFH Manifold & Wiring Centre

- The **Salus wiring centre** is installed and in use.
- It receives call-for-heat signals from Salus thermostats and drives the UFH manifold actuators and the boiler.

**UFH zones (current):**

| Wiring Centre Channel | Zone | Thermostat |
| :--- | :--- | :--- |
| Zone 1 | Living Room | Wired Salus thermostat |
| Zone 2 | Guest House (Room 2) | Wired Salus thermostat |

So there are **2 UFH zones**: Living Room and Guest House Room 2, each controlled by a **wired Salus thermostat**.

### 3. Radiators

- **Radiator circuit:** The radiator zone is controlled by a **wireless Salus thermostat** located in the **Master Bedroom**. When it calls for heat, the wiring centre enables the radiator circuit and the boiler fires.
- **Zone valve actuator:** A **Honeywell VC8010-12** 24V actuator drives the radiator circuit zone valve (2-port or 3-port). The Salus wiring centre energises the actuator when the radiator thermostat calls for heat; the valve opens and allows flow to the radiator circuit.
- **Radiator heads:** All radiators use **manual thermostatic valve (TRV) heads**. There are no smart TRVs (e.g. Aqara W600) today; room-by-room radiator control is manual via the TRV dials.

### 4. MDV Duct System

- The **MDV duct system** is installed and serves the main house (Living Room, bedrooms, Playroom, Master Bedroom as per the future design).
- It is controlled by a **wired controller**: **KJR-12B/DP(T)** with connections A–E (A brown, B red, C yellow, D black, E white). It is **not** integrated with Home Assistant or the rest of the automation stack.

**Equipment model numbers:**

| Component | Model | Notes |
| :--- | :--- | :--- |
| Outdoor unit | MOU-36HFN1-QRC4 | Midea light-commercial ducted VRF, 36k BTU |
| Indoor distribution unit | MTB-36HWFN1-QRC4 | Midea A5 duct, high static pressure, R410A |

> The system is **Midea** equipment (MDV = Midea Ducted VRF / light-commercial). Service manual: *Service-manual-MIDEA-VERSUS-JOHNSON.compressed.pdf*. Nomenclature: HWFN = Heat pump, Wired control standard, Full DC inverter, R410A. QRC4 = 220–240V 1-phase 50 Hz.

### 5. Guest House Air Conditioning

- **2 air conditioning units** are installed in the guest house (one per room).
- They are controlled **only by their original IR remotes**. There is no Aqara M200 hub or Home Assistant integration for these units today.

### 6. Summary: Current vs. Planned

| Subsystem | Current state | Target (see Planned section below) |
| :--- | :--- | :--- |
| Boiler | Legacy boiler, on/off via Salus | Viessmann 111-F, OpenTherm modulation, HA control |
| UFH | 2 zones, wired Salus thermostats (Living Room, Guest House R2) | Aqara W500 thermostats, 2–3 zones, HA schedules |
| Radiators | Wireless Salus stat (Master Bedroom), manual TRVs | W600 TRVs, Shelly rad circuit enable, HA demand logic |
| MDV duct | Installed, standalone control | WF-60A1 WiFi + Midea AC LAN, HA interlocks with UFH |
| Guest house AC | 2 units, IR remotes only | Aqara M200 IR control, HA guest-house modes |


---

## Planned / Target


### 1. Core Principles

| Principle | Implementation |
| :--- | :--- |
| **Local-first** | All control logic runs in Home Assistant on-premises. No internet dependency for heating, cooling, or scheduling. |
| **Ecosystem consolidation** | Primary RF protocol is Aqara Matter/Thread/Zigbee. WiFi (Shelly, ESPHome) is used only where mains switching is required. No new ecosystems added unless unavoidable. |
| **OpenTherm modulation** | Boiler runs "slow and low" — burner modulates rather than bang-bang cycling. Maximises condensing efficiency, prevents short-cycling, extends boiler life. |
| **Safety interlocks** | HA automations enforce hard rules: UFH and cooling never fight each other; MDV cannot heat a room the UFH is already heating. |
| **Graceful degradation** | If HA is unreachable, thermostats (W500, W600) fall back to their last setpoint. Boiler defaults to a safe flow temperature. The house remains warm. |

#### Control Hierarchy (Who Wins)

```
Home Assistant (highest authority)
  └── Physical wall controls (W500, W600, W100) — manual override
        └── Boiler safety limits (lowest level, hardware-enforced)
```

---

### 2. Physical Layout & Zone Map

#### Main House

| Room | Radiators | UFH | Cooling | Wall Sensor | Zone Type |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Living Room | — | W500 (UFH stat) | MDV Duct AirCon | W500 (built-in) | Hybrid — UFH primary heat, MDV cooling |
| Bathroom | 2x W600 | W500 (if loop independent, TBC) | — | W500 (built-in) | Hybrid UFH + Rad |
| Alex (child bedroom) | 1x W600 | — | MDV Duct AirCon (shared) | T1 | Hybrid Rad + MDV |
| Vicky (child bedroom) | 1x W600 | — | MDV Duct AirCon (shared) | T1 | Hybrid Rad + MDV |
| Playroom | 1x W600 | — | MDV Duct AirCon (shared) | W100 (display sensor) | Hybrid Rad + MDV |
| Master Bedroom | 2x W600 | — | MDV Duct AirCon (shared) | W100 (display sensor) | Hybrid Rad + MDV |
| Office | 1x W600 | — | Smart AirCon (Gree/LG) | T1 | Hybrid Rad + AirCon |

> **Sensor logic:** W100 display sensors are used in Master Bedroom and Playroom where visibility of current temperature is useful. T1 sensors are used in all other rooms. An outdoor T1 provides the weather compensation input. W500 and W600 have built-in sensors but wall sensors provide a more representative room-average reading. W600 sensors read close to the radiator and should not be used as the sole room temperature source for HA demand logic — always use the wall sensor as the primary temp source for HA automations.

#### Guest House

| Room | Radiators | UFH | Cooling | Wall Sensor | Zone Type |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Guest Bedroom | — | UFH (shared zone, W500 #1) | Old AirCon (IR via M200) | T1 | Hybrid UFH + IR AC |
| Guest Living Room | — | UFH (shared zone, W500 #1) | Old AirCon (IR via M200) | T1 | Hybrid UFH + IR AC |
| Guest Kitchen | — | UFH (shared zone, W500 #1) | — | T1 | UFH only |
| Guest Bathroom | — | UFH (shared zone, W500 #1) | — | T1 | UFH only |

> **Guest House UFH:** Guest Bedroom, Guest Living Room, Guest Kitchen, and Guest Bathroom share a single UFH zone controlled by W500 #1. Individual room temperature feedback comes from the T1 sensors — HA uses these for humidity management and to decide whether to trigger the IR AirCon in Guest Bedroom and Guest Living Room independently.

#### UFH Zones Summary

| Zone | Location | Thermostat | Wiring Centre Channel |
| :--- | :--- | :--- | :--- |
| Zone 1 | Guest House (Bedroom, Living Room, Kitchen, Bathroom) | Aqara W500 #1 | Channel 1 |
| Zone 2 | Living Room | Aqara W500 #2 | Channel 2 |
| Zone 3 | Bathroom (if independent loop confirmed) | Aqara W500 #3 | Channel 3 |

> **Pre-purchase action required:** Physically verify at the manifold cabinet whether the bathroom has an independent pipe loop before ordering the 3rd W500. If the bathroom shares the Living Room loop, Zone 2 covers both and no 3rd W500 is needed.

---

### 3. Boiler & OpenTherm Strategy

#### 3.1 Boiler: Viessmann Vitodens 111-F (B1SG)

The **Vitodens 111-F** is a floor-standing condensing storage combi with an integrated 130 L DHW cylinder. It is the correct model for this installation.

**Viessmann Quote (ref: 9520283558, 12 Mar 2026):**

A formal quote has been received from Viessmann Sp. z o.o. (contact: Karol Wieniawski, +48 782 756 346, Karol.Wieniawski@carrier.com) titled *"Pakiet Vitodens 111-F B1SG — sterowanie OpenTherm"*. The quote covers the boiler package and the complete hydraulic distribution system. Valid for 1 month from 12 Mar 2026.

**Two kW variants quoted (select one based on heat loss calc):**

| Pos. | Variant | Order No. | Net Price | Notes |
| :--- | :--- | :--- | :--- | :--- |
| 10 | **Vitodens 111-F B1SG 3.2–32 kW** | Z031665 | 14 958 PLN | Package with top hydraulic/gas connections (rear). Side connection option available. |
| 20* | **Vitodens 111-F B1SG 3.2–25 kW** (alternative) | Z031664 | 14 319 PLN | Same package, lower output. *Not included in quote total — select one.* |

> **Decision required:** A **heat loss calculation (PN-EN 12831)** must be completed before selecting the kW variant. For a 250 m² property with ~2005 Polish insulation standards (wall U ≈ 0.30 W/m²K), expect **19–25 kW** total demand (main house + guest house). The 25 kW variant (Z031664) is the likely candidate; the 32 kW risks short-cycling even with OpenTherm modulation. Commission a Polish HVAC engineer to perform the calc.

**Key specifications relevant to this design:**
- **OpenTherm built-in** — 2-wire OT terminals on the boiler; no extension module required (unlike 222-F). Confirmed by Viessmann in quote position 80: *"Możliwe jest wykorzystanie złącza OpenTherm do podłączenia zewnętrznej automatyki."*
- Modulation range depends on variant: 25 kW → 3.2–25 kW (**1:8**); 32 kW → 3.2–32 kW (**1:10**)
- MatriX-Plus burner with Lambda Pro Plus combustion control
- Inox-Radial stainless heat exchanger — long service life
- Built-in WiFi module (ViCare) — **will NOT be used**; all control via OpenTherm
- Widely serviced across Poland (dense Viessmann dealer network)

**Installer instruction (mandatory):**
> "Do NOT connect the boiler to VitoConnect, Vitotronic, or any Viessmann cloud gateway. Do NOT connect a Viessmann outdoor temperature sensor to the boiler — weather compensation is handled externally via OpenTherm. Leave the OpenTherm terminals accessible. I will be connecting a third-party OpenTherm controller."

#### 3.2 OpenTherm on 111-F

The Vitodens 111-F (1xx series) exposes **OpenTherm directly** on the control board — no Viessmann OpenTherm Extension Module is needed. A 2-wire OT bus runs from the boiler terminals to the gateway. For 1xx series, Viessmann does **not** offer the WAGO Modbus gateway; OpenTherm is the supported path for external automation.

**Viessmann official confirmation (quote position 80):**
> *"Dla kotłów Vitodens serii 1xx nie ma możliwości skorzystania z bramki WAGO i przekazania danych do zewnętrznych systemów nadzoru i sterowania. Możliwe jest wykorzystanie złącza OpenTherm do podłączenia zewnętrznej automatyki. Automatyka OpenTherm poza zakresem oferty!"*
>
> Translation: For Vitodens 1xx series boilers, the WAGO gateway is not available. An OpenTherm connection for external automation **is** available. OpenTherm automation is outside the scope of Viessmann's quote — this is our responsibility (Nodo OTGW + HA).

#### 3.3 Hydraulic Distribution (from Viessmann Quote)

The Viessmann quote specifies a **3-circuit hydraulic separator** with dedicated pump groups. This defines the physical plumbing layout between the boiler and the heating zones.

**Hydraulic separator:**

| Pos. | Item | Order No. | Qty | Net Price | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 40 | Rozdzielacz ze sprzęgłem — 3 heating circuits | 7773837 | 1 | 2 277 PLN | Hydraulic separator with low-loss header; distributes boiler flow to 3 independent circuits |

**Pump groups:**

| Pos. | Item | Order No. | Qty | Net Unit Price | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 50 | **GRA2** DN25 / Wilo PARA 25/6 — with mixing valve + 3-point actuator (230 V) | 7183336 | 2 | 2 144 PLN | Mixed circuits — flow temperature blended down for UFH protection |
| 60 | **GDA2** DN25 / Wilo PARA 25/6 — without mixing valve (direct circuit) | 7183292 | 1 | 1 577 PLN | Direct circuit — full boiler flow temperature to radiators |

**DHW circulation pump connection set:**

| Pos. | Item | Order No. | Qty | Net Price | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 30 | Zestaw przyłączeniowy pompy cyrkulacyjnej | ZK05978 | 1 | 983 PLN | Connection kit for DHW recirculation pump (pump itself to be sourced separately if not included) |

**Flue system:**

| Pos. | Item | Order No. | Qty | Net Price | Notes |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 70 | Zestaw bazowy w szacht 80/125 | 7142128 | 1 | 567 PLN | Concentric flue base kit: 90° tee, 300 mm pipe, rosette, shaft cover with rain collar, flue elbow + bracket |

> **Flue note:** The quote includes a basic flue kit only. Viessmann's note states: *"Zestawienie elementów nie gwarantuje prawidłowego montażu komina. Przed złożeniem zamówienia należy skonsultować dobór elementów z wykonawcą instalacji spalinowej."* — flue element selection must be verified with the flue installer before ordering.

**Circuit assignment (guest house confirmed as shared boiler with own UFH loop):**

```
Viessmann 111-F B1SG
  │
  └── Hydraulic Separator (7773837) — 3 circuits
        │
        ├── Circuit 1 (GRA2 — mixed) → UFH Main House
        │     Mixing valve blends flow down to ~35–45°C for UFH loops
        │     Feeds: Living Room (W500 #2), Bathroom (W500 #3 if confirmed)
        │
        ├── Circuit 2 (GRA2 — mixed) → UFH Guest House
        │     Mixing valve blends flow down to ~35–45°C for UFH loops
        │     Feeds: Guest House zone (W500 #1)
        │
        └── Circuit 3 (GDA2 — direct) → Radiator Circuit
              No mixing — radiators receive full boiler flow temp (up to 55–60°C)
              Feeds: All W600 TRV-equipped radiators
              Master enable: Shelly Plus 1 relay (HA demand logic)
```

> **Key design point:** The 2x GRA2 pump groups with mixing valves solve the UFH temperature protection requirement. UFH loops will never receive water above the mixing valve setpoint (~35–45°C), even when the boiler runs at higher flow temperatures for the radiator circuit. This eliminates the need for a separate thermostatic blending valve on the UFH manifold.

> **DHW priority:** The 111-F has an internal diverter valve. When DHW is demanded (cylinder temperature drops below setpoint), the boiler suspends central heating and diverts flow to the integrated 130 L cylinder. All three heating circuits are temporarily starved. HA should account for DHW reheat pauses in its demand logic — flow temperature setpoint changes sent during a DHW cycle will not take effect until the cylinder is satisfied.

> **Confirmed:** Guest house runs from the same boiler and has its own UFH loop controlled by W500 #1. The exact circuit numbering (which GRA2 port = main house, which = guest house) should be confirmed with the HVAC engineer during installation. The GDA2 (direct, no mixing) must be assigned to radiators — never to UFH.

#### 3.4 OpenTherm Gateway: Nodo OTGW UTP + Ethernet + PoE/USB-C splitter

This is the bridge between the OpenTherm bus and Home Assistant.

| Component | Role |
| :--- | :--- |
| **Nodo OpenTherm Gateway UTP** | PIC-based OTGW kit ([Nodo-shop](https://www.nodo-shop.nl/en/our-products/327-opentherm-gateway-builder-utp.html)); sits on the 2-wire OT bus. Supports **no thermostat present** — HA is the sole controller. |
| **Ethernet interface** | Optional board (e.g. USR-TCP232-T2) added to the OTGW for TCP connectivity so HA can talk to the gateway over the LAN (no USB/serial run to the server). |
| **PoE to USB Type C splitter** | 802.3af PoE from the switch is converted to 5 V USB (Type C or A) to power the OTGW at the engine room; no separate mains outlet required. |
| **Home Assistant** | **OpenTherm Gateway** integration (pyotgw) connects to the OTGW via TCP (Ethernet module) and exposes setpoints, sensors, and controls. |

**Wiring:**
```
Viessmann 111-F
  └── [2-wire OpenTherm bus / twin-core 18/2 bell wire]
        └── Nodo OTGW UTP (Ethernet module fitted)
              ├── [USB] ← PoE-to-USB Type C splitter ← [Cat6 PoE] from USW Pro Max 48
              └── [Ethernet] → Cat6 → USW Pro Max 48 (IoT VLAN) → Home Assistant
```

#### 3.5 Boiler Control Strategy: "Slow and Low"

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

**OpenTherm Gateway (OTGW) entities in HA:**  
The [OpenTherm Gateway](https://www.nodo-shop.nl/en/our-products/327-opentherm-gateway-builder-utp.html) integration exposes (among others):
- **Control setpoint** — flow temperature setpoint (HA writes this)
- **Boiler water temperature** — actual flow temperature (read)
- **Relative modulation level** — burner modulation % (read, for monitoring)
- **Flame status** — burner on/off (read)
- **Fault indication** — boiler fault flag (read, triggers HA alert)
- **Outside temperature** — if boiler has outdoor sensor fitted (read)
- **DHW setpoint** — domestic hot water setpoint (HA can manage this too)

---

### 4. Heating & Cooling Architecture

#### 4.1 Zone Schedules

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
| Guest House (Frost Protection) | **7°C minimum** | Boiler only; AC heating activates at critical 4°C | Humidity management active; long absences |

#### 4.2 Operating Modes

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
    - Frost Protection # All zones → 7°C minimum; boiler only; at critical 4°C,
                       # AC heating mode also activates as backup
```

**Guest House Mode** — controls guest house independently:
```
input_select.guest_house_mode:
  options:
    - Occupied         # Follows same schedules/targets as main house
    - Unoccupied       # 18°C constant; AC inhibited; humidity management active
    - Frost Protection # 7°C minimum; boiler only; AC heating at critical 4°C; humidity active
```

Mode transitions:
- **Season mode:** Automatic via 24h outdoor temp average with 48h guard + hysteresis; manual override always available
- **House modes:** Manual only — set via Shelly Wall Display, UniFi Connect panel, or HA Companion App

---

### 5. UFH Manifold & Wiring Centre

#### 5.1 Existing Salus Wiring Centre — To Be Evaluated

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

#### 5.2 Zone Channel Mapping

| Wiring Centre Channel | Old Thermostat | New Thermostat | Zone |
| :--- | :--- | :--- | :--- |
| Channel 1 | Salus wired stat | **Aqara W500 #1** | Guest House UFH |
| Channel 2 | Salus wired stat | **Aqara W500 #2** | Living Room UFH |
| Channel 3 | Salus wired stat (if present) | **Aqara W500 #3** (if loop confirmed) | Bathroom UFH |

#### 5.3 Radiator Circuit Channel — Modified

Previously the radiator zone was driven by a single wireless Salus thermostat covering all radiators. With W600 TRVs installed on every radiator, individual room thermostats are no longer needed. The radiator circuit channel on the wiring centre is rewired as follows:

- The channel output is wired through a **Shelly Plus 1** (DIN rail, IoT VLAN WiFi)
- HA controls this Shelly as a **"Radiator Circuit Enable"** relay
- When any W600 reports demand (valve opening), HA closes the relay → wiring centre fires the radiator zone → boiler gets a call for heat on the rad circuit
- When no W600 has demand (summer mode, or all rooms at setpoint), HA opens the relay → radiator circuit is fully inhibited

This gives a clean master shutoff for the entire radiator circuit without touching wiring centre internals.

#### 5.4 Aqara W500 — Key Properties

- Zigbee/Matter wall thermostat
- Volt-free relay output for UFH control
- Displays current and target temperature
- Pairs directly with Aqara M3 Hub
- Supports HA climate entity natively
- Physical rotary control for manual override

---

### 6. Radiator Control (W600 TRVs)

#### 6.1 Aqara W600 — Key Properties

- Thread/Matter radiator TRV head
- Reports **valve opening percentage** back to HA — this is the critical data point for load compensation
- Replaces existing thermostatic TRV heads on each radiator
- Pairs with Aqara M3 Hub (Matter/Thread border router)
- Supports HA climate entity natively

#### 6.2 Radiator Inventory

| Location | Quantity | Wall Sensor | Notes |
| :--- | :--- | :--- | :--- |
| Bathroom | 2x W600 | W500 (built-in) | Boost to 23°C during comfort windows |
| Alex (bedroom) | 1x W600 | T1 | Setback schedule |
| Vicky (bedroom) | 1x W600 | T1 | Setback schedule |
| Playroom | 1x W600 | W100 | Active 2pm–9pm schedule |
| Master Bedroom | 2x W600 | W100 | Setback schedule; MDV also serves this room |
| Office | 1x W600 | T1 | Occupancy-based schedule; Gree/LG also serves this room |

**W600 — Office pairing codes (reference):**

| Protocol | Pairing code |
| :--- | :--- |
| Matter | 1369-274-4889 |
| Aqara | 2067-4710 |

**Total W600 count: 8 minimum.** Confirm during site survey — if any room has additional radiators not listed, add one W600 per head.

> **Important:** W600 built-in sensors read temperature near the radiator, which is not representative of room temperature. HA automations must use the dedicated wall sensor (W100 or T1) as the primary temperature source for each room. The W600 sensor is useful only for valve position feedback and load compensation logic.

#### 6.3 W600 Demand Aggregation Logic (HA)

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

### 7. MDV Duct AirCon Integration

#### 7.1 Equipment & Model Numbers

| Component | Model | Notes |
| :--- | :--- | :--- |
| Outdoor unit | MOU-36HFN1-QRC4 | Midea light-commercial VRF, 36k BTU |
| Indoor distribution unit | MTB-36HWFN1-QRC4 | Midea A5 duct, high static pressure, R410A, wired control standard |

The system is **Midea** equipment (MDV = Midea Ducted VRF). Sources: service manual *Service-manual-MIDEA-VERSUS-JOHNSON.compressed.pdf*, WF-60A1 compatibility list [midea.com.ua](https://www.midea.com.ua/en/products/light-commercial-ac-semi-industrial-conditioners/remote-control-panels/wi-fi-module-wf-60a1).

#### 7.2 WiFi Integration: WF-60A1 + Midea AC LAN

**WF-60A1 compatibility research (2026-02):**

| Evidence | Finding |
| :--- | :--- |
| Midea WF-60A1 official list | MTB-18HWFN1-Q, MTB-24HWFN1-Q explicitly listed (Multi-split). MTB-36HRFN1-S, MTB-36HWDN1-R listed for 36k. |
| Service manual | MTB-36HWFN1-QRD0 documented — same product family as MTB-36HWFN1-QRC4 (QRC4 vs QRD0 = power/region suffix). |
| Airconplenums, Midea docs | MTB 12–55 HWFN1-QRD0 marketed as one product line. HWFN1 series shares platform across 18k–55k capacities. |
| WF-60A1 installation | Connects via **CN40** on indoor unit main board; 4-core shielded cable. Manual: *pl_lcac-wifi-manual-wf-60a1-c.pdf*. |

**Compatibility assessment:** High confidence (≈85–90%). MTB HWFN1 at 18k/24k is on the official list; MTB-36HWFN1 is the same platform. Pre-purchase: confirm CN40 connector exists on MTB indoor unit main board (E-box). If unsure, verify with Midea/installer.

**Integration path:**
1. Install **Midea WF-60A1** WiFi module (connects to CN40 on MTB indoor unit).
2. Add **Midea AC LAN** integration (HACS) to Home Assistant.
3. WF-60A1 joins **IoT VLAN** (192.168.12.x); Midea AC LAN discovers device on LAN.
4. HA exposes a single `climate` entity for the MDV system; local control, no cloud required for normal operation.

#### 7.3 Scope

The MDV shared duct system serves **5 rooms**:
- Living Room
- Alex (bedroom)
- Vicky (bedroom)
- Playroom
- Master Bedroom

There is **no individual room regulation** — the MDV system is shared on/off with mode control (heat/cool/fan). Individual room comfort is achieved by combining MDV with the room's primary heat source (UFH or rads) and using Aqara temp sensors to decide when MDV should run.

#### 7.4 MDV/UFH Interlock — Living Room

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

#### 7.5 MDV Demand Logic

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

### 8. Guest House AirCon (IR Control)

#### 8.1 Hardware: Aqara M200 Hub (PoE)

The old AirCon units in Guest Bedroom and Guest Living Room do not have smart connectivity. They are controlled via **IR blasters** using the **Aqara M200 Hub**, which has a built-in IR transmitter.

- **2x Aqara M200 Hubs** — one per Guest Bedroom and Guest Living Room, PoE-powered
- Joins Aqara Zigbee/Matter ecosystem — no new ecosystem introduced
- HA sends IR commands via the M200 to replicate remote control actions
- Pairs with the Aqara M3 Hub

#### 8.2 Limitations & Mitigation

IR control has no state feedback — HA cannot confirm the AC unit turned on or what mode it's in. Mitigation:

- HA maintains a **virtual state** for each AC (tracks last command sent)
- Aqara room temp sensors provide indirect feedback — if room temp moves toward setpoint after an IR ON command, assume it worked
- **Interlock:** HA automation prevents IR AC and UFH from running simultaneously in the same Guest House room

#### 8.3 Guest House Occupancy Mode

The Guest House operates on a dedicated `input_select.guest_house_mode` with three states:

| Mode | UFH Setpoint | AC | Humidity Management | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Occupied** | Follows main house schedule (21°C / setbacks) | Available, same logic as main house | Standard | Guests treated as full occupants — same comfort targets and season mode apply |
| **Unoccupied** | 18°C constant setback | Inhibited (temperature) | Active (see below) | House is empty but maintained |
| **Frost Protection** | 7°C minimum — pipes safe | Inhibited unless critical 4°C | Active (see below) | Boiler only; AC heating activates at 4°C as backup |

```
input_select.guest_house_mode:
  options: [Occupied, Unoccupied, Frost Protection]
```

**Occupied mode detail:** When set to Occupied, the Guest House UFH zone follows the same season mode and schedule logic as the main house. The old AirCon units (IR via M200) are available for heating and cooling on the same demand thresholds. The Guest House is treated as a full zone of the property.

**Toggle locations:** Shelly Wall Display in Guest Living Room, UniFi Connect panel, HA Companion App.

---

### 9. Office AirCon (Gree/LG)

The Office has a smart AirCon unit (Gree or LG — confirm model). This is integrated via the **Gree integration** (built into Home Assistant core — no HACS required).

- Integration operates **LAN-local** — no cloud dependency
- Unit must be on the **IoT VLAN** (192.168.12.x)
- HA sees a `climate` entity for the Office AC
- Schedule: 21°C when office is occupied; setback to 18°C otherwise
- Occupancy can be presence-based (HA companion app on phone) or time-scheduled

> **Action:** Confirm whether unit is Gree or LG. If LG, use the **LG ThinQ integration** (available in HA, also supports LAN-local for compatible models).

---

### 10. Sensor Strategy

#### 10.1 Temperature Sensors — Aqara TVOC T1 (Zigbee)

One sensor per room for room temperature feedback to HA. These drive the demand logic for UFH, radiators, MDV, and IR AC.

| Location | Wall Sensor | Primary HA Temp Source | Notes |
| :--- | :--- | :--- | :--- |
| Living Room | W500 #2 (built-in) | W500 built-in | UFH demand + MDV interlock logic |
| Bathroom | W500 #3 (built-in, if loop independent) | W500 built-in | UFH + rad boost logic |
| Alex (bedroom) | T1 | T1 | Rad demand + MDV demand |
| Vicky (bedroom) | T1 | T1 | Rad demand + MDV demand |
| Playroom | W100 (display sensor) | W100 | Rad demand + MDV demand; display for quick check |
| Master Bedroom | W100 (display sensor) | W100 | Rad demand + MDV demand; display useful |
| Office | T1 | T1 | Rad + Gree/LG demand |
| Guest Bedroom | T1 | T1 | UFH feedback + IR AC trigger |
| Guest Living Room | T1 | T1 | UFH feedback + IR AC trigger |
| Guest Kitchen | T1 | T1 | UFH feedback |
| Guest Bathroom | T1 | T1 | UFH feedback |
| Outdoor | T1 (outdoor rated) | T1 outdoor | Weather compensation heating curve input |

> **Critical:** W600 TRV built-in sensors must never be used as the primary room temperature source in HA automations. They read near the radiator and will be several degrees higher than actual room temperature when the rad is firing. Always use the dedicated wall sensor (W100 or T1) as `sensor.room_temperature` in all automations and climate entities.

> **Humidity:** The Aqara T1, W100, and W500 all report **temperature and humidity**. No additional humidity sensors are needed. The Guest House T1 sensors (Guest Bedroom, Guest Living Room) provide the humidity readings used by the Unoccupied and Frost Protection humidity management automations (Automation 11). Ensure HA exposes the humidity entities from these T1 devices.

#### 10.2 W100 Display Sensor

The **Aqara W100** combines a temperature/humidity sensor with a small e-ink display. Use in **Master Bedroom** and **Playroom** where visibility of current temp is useful. Other rooms use T1 sensors. Stays in the Aqara ecosystem.

---

### 11. Schedule & Automation Logic

#### 11.1 HA Entities Overview

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
  frost_critical_temp:      # Critical temp below which AC heating activates (default: 4°C)
  humidity_threshold_high:  # Humidity % above which dehumidify cycle triggers (default: 70%)
  humidity_run_minutes:     # Duration of AC dehumidify cycle in minutes (default: 30)

input_datetime:
  mdv_last_mode_change:       # Timestamp of last MDV heat↔cool mode transition
  guest_ac_bedroom_last_dehumid:  # Last dehumidify run timestamp — Guest Bedroom
  guest_ac_living_last_dehumid:   # Last dehumidify run timestamp — Guest Living Room

input_text:
  mdv_last_mode:         # Tracks last active MDV mode: HEAT / COOL / FAN / OFF
                         # Needed because Midea AC LAN may not reliably report
                         # previous mode after an OFF state
```

#### 11.2 Key Automations

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
  In Frost Protection, the boiler is always primary. If any room reaches
  frost_critical_temp (default 4°C), AC in heating mode also activates as backup.
  When season_mode changes, set input_datetime.mdv_last_mode_change = now
  to force MDV through its 30-min grace period.
```

**Automation 2: Boiler Flow Temp (Weather Compensation)**
```
Trigger: outdoor temperature sensor updates (every 15 min)
Action:
  flow_temp = heating_curve(outdoor_temp)
  # Example curve: flow = 70 - (2.0 * outdoor_temp), clamped 35–60°C
  Write flow_temp to OT setpoint via OpenTherm Gateway integration
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
         OR (main_house_mode = Frost Protection AND any room temp < frost_critical_temp)

Action when entering Frost Protection:
  → All W500 UFH setpoints → frost_temp_main (default 7°C)
  → All W600 TRV setpoints → frost_temp_main
  → MDV: INHIBITED for normal frost (boiler only)
  → Boiler flow temp: minimum curve value (35°C) — runs only if zone calls
  → All schedules and setbacks suspended
  → HA notification: "Main house set to Frost Protection"

Action when any room < frost_critical_temp (default 4°C):
  → MDV: enable heating mode (target 5°C) — backup to boiler
  → Office AC: enable heating if office temp < frost_critical_temp
  → HA notification: "Critical frost — AC heating backup activated"
  → When all rooms > frost_critical_temp + 1°C hysteresis: revert MDV/AC to inhibited

Action when leaving Frost Protection (back to Normal):
  → Restore all setpoints to scheduled values
  → Re-enable MDV and schedule logic
  → HA notification: "Main house returned to Normal mode"

Note: Frost Protection uses boiler as primary. AC heating only activates at
critical temp (4°C) as a safety backup. The boiler has its own hardware
anti-freeze at ~3°C flow temp; the HA layer acts earlier (7°C room) to avoid it.
```

**Automation 9: Guest House Frost Protection**
```
Trigger: guest_house_mode changes to "Frost Protection"
         OR (guest_house_mode = Frost Protection AND
             any guest room temp < frost_temp_guest)
         OR (guest_house_mode = Frost Protection AND
             any guest room temp < frost_critical_temp)

Action when entering Frost Protection:
  → Guest House W500 #1 setpoint → frost_temp_guest (default 7°C)
  → Guest House IR AC: INHIBITED for normal frost (boiler only)
  → Humidity management: ACTIVE (see Automation 11)
  → HA notification: "Guest house set to Frost Protection"

Action when any guest room < frost_critical_temp (default 4°C):
  → IR AC for affected room(s): enable heating mode (target 5°C)
  → HA notification: "Critical frost — Guest House AC heating backup activated"
  → When all guest rooms > frost_critical_temp + 1°C hysteresis: revert IR AC to inhibited

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
    → Record timestamp to guest_ac_bedroom_last_dehumid or guest_ac_living_last_dehumid

  ELIF room_humidity > 80% (emergency threshold — override season restriction):
    → Run dehumidify cycle regardless of season_mode
    → HA notification: "High humidity alert — Guest Bedroom / Guest Living Room dehumidifying"

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

#### 11.3 Presence & Occupancy

- **Children's rooms:** Time-schedule based (school hours)
- **Playroom:** Time-schedule based (2pm–9pm active)
- **Office:** HA companion app presence detection OR time-schedule
- **Guest House:** `input_select.guest_house_mode` — Occupied / Unoccupied / Frost Protection
- **Master Bedroom:** Time-schedule (setback 8am–7pm)
- **Main House:** `input_select.main_house_mode` — Normal / Frost Protection

---

### 12. Network Architecture

#### 12.1 Core Hardware

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
| **Nodo OTGW UTP (Ethernet)** | OpenTherm gateway | PoE via USB-C splitter + Ethernet to USW Pro Max 48 | Engine Room |

For a full description of the network topology, VLANs, WiFi/Thread configuration, and security rules, see:

- [`networking.md`](networking.md)

#### 12.2 VLAN Design

| Network | VLAN ID | Subnet | Devices | WAN |
| :--- | :--- | :--- | :--- | :--- |
| **Core** | 1 | `192.168.10.x` | UDM SE, switches, APs (management) | Yes |
| **Server** | 6 | `192.168.1.x` | Unraid Server | Yes |
| **IoT** | 3 | `192.168.12.x` | Raspberry Pi 5 (HA), Nodo OTGW (Ethernet), M3 Hub, M200 Hubs, Shelly devices, MDV WF-60A1, Gree/LG unit | Restricted |
| **NOT** | 4 | `192.168.13.x` | Non-internet things (WAN blocked) | **No** |

**IoT VLAN rules:**
- Devices can communicate with HA (Pi 5) on same VLAN
- WAN access restricted — only allowed for OTA updates (specific IP/domain whitelist)
- No access to Core or Server VLANs (unidirectional — HA on IoT can query Unraid on Server VLAN, not reverse)

#### 12.3 Thread Mesh Configuration

- Thread runs on **Channel 25** — chosen to minimise overlap with 2.4GHz WiFi bands
- Aqara M3 acts as Thread Border Router
- W600 TRVs and W500 thermostats join Thread mesh natively
- U7 Pro XG APs maintain Thread channel discipline — configure in UniFi settings

#### 12.4 Connectivity Map

```
[Fiber/Cable WAN]
    └── UDM SE (Gateway)
          └── [10G SFP+] UniFi Aggregation Switch (SFP)
                ├── [10G SFP+] USW Pro Max 48 PoE
                │              ├── [PoE+]  Raspberry Pi 5 (Home Assistant)
                │              ├── [PoE]   Aqara M3 Hub
                │              ├── [PoE + Ethernet]   Nodo OTGW (Engine Room, OpenTherm; PoE→USB-C splitter)
                │              └── [PoE]   Aqara M200 x2 (Guest House)
                ├── [10G SFP+] USW XG 10 PoE
                │              └── [PoE]   U7 Pro XG APs (all access points)
                └── [10G SFP+] Unraid Server (Ryzen 5900X, 100TB)
```

---

### 13. Control & Display Hierarchy

A layered control hierarchy prevents app fatigue and keeps the system guest-accessible.

| Layer | Device | Location | Role |
| :--- | :--- | :--- | :--- |
| 1 (Primary) | UniFi Connect 21" | Living Room | "Command Tower" — full system overview, scene control, guest-friendly |
| 2 (Tactical) | Shelly Wall Display XL (10.1") | Couch area, Upstairs, Guest Living Room | Home Assistant mode — fast local control without opening an app |
| 3 (Room) | Aqara W500 | Living Room, Guest House, Bathroom | Room thermostat — physical rotary setpoint adjustment |
| 3 (Room) | Aqara W600 | Each radiator | TRV — manual temperature adjustment at the rad |
| 3 (Room) | Aqara W100 | Master Bedroom, Playroom | Display + sensor — see temp, no control needed |
| 4 (Remote) | HA Companion App | Mobile | Full remote access when away from home |

---

### 14. Hardware Bill of Materials

#### Boiler, Hydraulics & OpenTherm (Viessmann Quote 9520283558)

| Pos. | Item | Order No. | Qty | Net Price | Supplier |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 10 | Viessmann Vitodens 111-F B1SG 3.2–32 kW (package) | Z031665 | 1 | 14 958 PLN | Viessmann Sp. z o.o. |
| 20* | *Alt:* Vitodens 111-F B1SG 3.2–25 kW (package) | Z031664 | 1 | 14 319 PLN | Viessmann Sp. z o.o. |
| 30 | DHW circulation pump connection set | ZK05978 | 1 | 983 PLN | Viessmann Sp. z o.o. |
| 40 | Hydraulic separator — 3 heating circuits | 7773837 | 1 | 2 277 PLN | Viessmann Sp. z o.o. |
| 50 | Pump group GRA2 DN25 / Wilo PARA 25/6 (with mixing valve + 3-pt actuator 230V) | 7183336 | 2 | 2 144 PLN ea. | Viessmann Sp. z o.o. |
| 60 | Pump group GDA2 DN25 / Wilo PARA 25/6 (direct, no mixing valve) | 7183292 | 1 | 1 577 PLN | Viessmann Sp. z o.o. |
| 70 | Flue base set — shaft 80/125 | 7142128 | 1 | 567 PLN | Viessmann Sp. z o.o. |

**Quote total (pos. 10 selected): 24 650 PLN net + 23% VAT = 30 319,50 PLN gross.**
*If pos. 20 selected instead of 10: 24 011 PLN net + VAT = 29 533,53 PLN gross.*

> Pos. 20 (25 kW) is marked as an alternative — not included in the quote total. Select one variant based on heat loss calc.

| Item | Qty | Supplier |
| :--- | :--- | :--- |
| Nodo OpenTherm Gateway UTP kit (Ethernet interface + enclosure) | 1 | [Nodo-shop](https://www.nodo-shop.nl/en/our-products/327-opentherm-gateway-builder-utp.html) |
| PoE to USB Type C splitter (802.3af → 5 V USB) | 1 | Nodo-shop (optional), or generic (e.g. 802.3af to USB-C) |

#### Thermostats & TRVs

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Aqara W500 UFH Thermostat | 2 (or 3 if bathroom loop confirmed independent) | x-kom.pl, Allegro, Media Expert |
| Aqara W600 Radiator TRV | 8 minimum (survey required — 1 per rad head) | x-kom.pl, Allegro |
| Aqara W100 Display Sensor | 2 (Master Bedroom, Playroom) | x-kom.pl, Allegro |

#### Sensors

| Item | Qty | Rooms | Supplier (Poland) |
| :--- | :--- | :--- | :--- |
| Aqara Temperature Sensor T1 (indoor) | 7 | Alex, Vicky, Office, Guest Bedroom, Guest Living Room, Guest Kitchen, Guest Bathroom | x-kom.pl, Allegro, Media Expert |
| Aqara Temperature Sensor T1 (outdoor rated) | 1 | Outdoor — weather compensation heating curve input | x-kom.pl, Allegro |

#### Switching & Control

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Shelly Plus 1 (DIN rail) | 1 (rad circuit enable) | Botland.pl, Nettigo.pl |
| Shelly Wall Display XL | 3 (couch, upstairs, guest room 2) | Botland.pl, Nettigo.pl |

#### AirCon Integration

| Item | Qty | Supplier (Poland) |
| :--- | :--- | :--- |
| Aqara M200 Hub (PoE) | 2 (Guest House rooms) | x-kom.pl, Allegro |
| Midea WF-60A1 WiFi module | 1 | Midea / HVAC distributor. Compatible with MTB-36HWFN1 (see Section 7.2). |

#### Compute & Network (if not already owned)

| Item | Qty | Notes |
| :--- | :--- | :--- |
| Raspberry Pi 5 (8GB, PoE+ HAT, NVMe HAT + SSD) | 1 | If not already in place |
| Aqara M3 Hub | 1 | If not already in place |

---

### 15. Risks & Critical Maintenance Notes

| Risk | Mitigation |
| :--- | :--- |
| **OTGW placement** | Nodo OTGW is powered via PoE→USB-C splitter and connected over Ethernet. Place OTGW and cables away from boiler ignition/mains to reduce interference. The OTGW does not provide galvanic isolation; if boiler electrical noise is a concern, use a serial connection to a host with isolated USB, or keep Ethernet run short and away from high-current boiler wiring. |
| **Database write wear** | HA Recorder is configured to write to **MariaDB on Unraid** (not the Pi 5 NVMe). Configure in `configuration.yaml`: `recorder: db_url: mysql://...`. Do not leave default SQLite on Pi. |
| **MDV/UFH conflict** | The Living Room MDV/UFH interlock automation (Section 11.2, Automation 5) is mandatory. Test this thoroughly before going live — a misconfigured interlock means the floor heats while the AC cools simultaneously. |
| **Thread channel conflict** | Thread must run on Channel 25. Verify U7 Pro XG APs are not advertising 2.4GHz WiFi on channels that overlap (channels 1–6 are safe; 11 overlaps with Thread Ch.25 slightly — use Ch.1 or Ch.6 for 2.4GHz WiFi). |
| **Installer ecosystem creep** | Explicitly tell the Viessmann installer not to fit VitoConnect or any Viessmann cloud module. Some installers fit these by default. It will complicate the OpenTherm setup. |
| **W600 battery life** | W600 TRVs are battery-powered. Set up a HA automation to alert when any W600 battery drops below 20%. A dead TRV means a stuck valve. |
| **Boiler sizing** | Do not proceed to purchase before heat loss calc. An oversized boiler will short-cycle. An undersized boiler won't meet demand on design days. |
| **Bathroom manifold loop** | If the bathroom shares the Living Room UFH loop and a W500 is incorrectly installed thinking it's independent, you'll have two thermostats fighting one loop. Verify physically at the manifold cabinet before ordering. |

---

### 16. Open Items & Pre-Purchase Decisions

> **Viessmann quote received:** Quote 9520283558 (12 Mar 2026) covers boiler package, hydraulic separator, pump groups, DHW circ pump set, and flue base kit. Valid 1 month. OpenTherm confirmed available; automation is owner's responsibility. See Sections 3.1 and 3.3 for full breakdown.

| # | Item | Owner | Status |
| :--- | :--- | :--- | :--- |
| 1 | Heat loss calculation (PN-EN 12831) — determines boiler kW variant (25 kW vs 32 kW). Viessmann quote valid 1 month from 12 Mar 2026. | HVAC engineer | **Blocking — must complete before boiler order** |
| 2 | Bathroom UFH loop independence — verify at manifold cabinet | Owner / plumber | **Blocking — determines W500 qty** |
| 3 | ~~Guest House boiler topology~~ — **Resolved.** Guest house shares the main boiler. It has its own UFH loop fed from one of the 3-circuit separator's GRA2 mixed circuits, controlled by W500 #1. | Owner | **Closed** |
| 4 | Confirm Salus wiring centre role — with the Viessmann 3-circuit separator + pump groups, the Salus centre may become redundant or serve a different function. Discuss with installer. | Owner / HVAC installer | To be decided during detailed design |
| 5 | Office AirCon — confirm brand/model (Gree vs LG) | Owner | Determines HA integration to use |
| 6 | W600 radiator count — full site survey of all rad heads | Owner | Determines W600 purchase qty |
| 7 | MDV WiFi: WF-60A1 — verify CN40 connector on MTB indoor unit before purchase (see Section 7.2) | Owner | Before MDV integration work |
| 8 | Midea AC LAN compatibility test — fit WF-60A1, confirm HA discovery on LAN | Owner / HA installer | Before going live |
| 9 | Gree/LG integration LAN-local test — confirm unit responds without cloud | Owner | Close out to-do |
| 10 | Heating curve tuning — set initial curve coefficients at commissioning | HVAC engineer / HA installer | At commissioning |
| 11 | Salus wiring centre vs new wiring centre — the Viessmann 3-circuit separator + pump groups may partially or fully replace the Salus centre’s function. Validate with installer. | Owner / HVAC installer | To be decided during detailed design |
| 12 | Confirm circuit assignment on 3-circuit separator: which GRA2 = main house UFH, which GRA2 = guest house UFH, GDA2 = radiators. See Section 3.3. | Owner / HVAC engineer | Before installation |
| 13 | Flue element verification — quote includes base kit (80/125 shaft) only; full flue run must be specified with flue installer before ordering. | Owner / flue installer | Before ordering |
| 14 | DHW circulation pump — quote includes connection set (ZK05978) but confirm whether the pump itself is included or must be sourced separately. | Owner / Viessmann dealer | Before ordering |
| 15 | Hydraulic connection orientation — top connections (default in quote) vs. side outlets. Decide based on engine room layout. | Owner / HVAC installer | Before ordering |

---

### 17. System Architecture Diagram

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

ENGINE ROOM             [Viessmann Vitodens 111-F]
(Heat Source)                │ (2-wire OpenTherm bus)
                        [Nodo OTGW UTP + Ethernet]
                             ├── USB ← PoE-to-USB-C splitter ← Cat6 PoE
                             └── Ethernet → Cat6 → (HA via IoT VLAN)
                             │
                        [Salus Wiring Centre]  (unchanged; zone calls from W500/Shelly)
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
    Bath x2             Living Room        Master Bedroom    Alex, Vicky, Office
    Alex x1             Guest House        Playroom          Guest Bedroom, Guest Living Room
    Vicky x1            Bathroom (TBC)                        Guest Kitchen, Guest Bathroom
    Playroom x1                                              Outdoor (weather comp)
    Master x2                                               Outdoor (weather)
    Office x1

                        [Aqara M200 PoE x2]  (Guest House)
                             │ IR
                        [Old AirCon Guest Bedroom]  [Old AirCon Guest Living Room]

─────────────────────────────────────────────────────────────────────

AIRCON (WiFi/LAN)       [MDV Duct System + WF-60A1 WiFi Module]
                             │ (Midea AC LAN, LAN, IoT VLAN)
                        Serves: Living Room, Alex, Vicky, Playroom, Master
                        HA interlock: inhibited when Living Room UFH active

                        [Gree/LG Office Unit]
                             │ (Gree / LG ThinQ integration, LAN)
                        Serves: Office only

─────────────────────────────────────────────────────────────────────

DISPLAYS                [UniFi Connect 21"] — Living Room (Command Tower)
                        [Shelly Wall XL x3] — Couch / Upstairs / Guest Living Room
                             (Home Assistant mode, local control)
                        [HA Companion App]  — Mobile (remote access)
```

---

### 18. Product References

Official product pages and documentation:

| Product | Store / manufacturer |
| :--- | :--- |
| Viessmann Vitodens 111-F B1SG | [Viessmann PL](https://www.viessmann.pl/pl/produkty/gazowe-kotly-kondensacyjne/vitodens-111-f.html), [V-market](https://v-market.pl/kategoria/viessmann-kondensacyjne-kotly-gazowe/vitodens-111f/) |
| Viessmann Quote 9520283558 (12 Mar 2026) | Viessmann Sp. z o.o., contact: Karol Wieniawski +48 782 756 346 |
| Viessmann hydraulic separator 3-circuit (7773837) | Viessmann Sp. z o.o. (included in quote) |
| Viessmann GRA2 pump group DN25 / Wilo PARA 25/6 (7183336) | Viessmann Sp. z o.o. (included in quote) |
| Viessmann GDA2 pump group DN25 / Wilo PARA 25/6 (7183292) | Viessmann Sp. z o.o. (included in quote) |
| Viessmann DHW circ pump connection set (ZK05978) | Viessmann Sp. z o.o. (included in quote) |
| Nodo OpenTherm Gateway UTP (Ethernet + PoE/USB-C) | [Nodo-shop](https://www.nodo-shop.nl/en/our-products/327-opentherm-gateway-builder-utp.html) |
| OTGW firmware & tools | [otgw.tclcode.com](http://otgw.tclcode.com/) |
| Aqara Hub M3 | [aqara.com](https://us.aqara.com/products/aqara-smart-hub-m3) |
| Aqara W500 UFH Thermostat | [aqara.com](https://www.aqara.com/en/product/floor-heating-thermostat-w500/) |
| Aqara W600 Radiator TRV | [aqara.com](https://www.aqara.com/en/product/radiator-thermostat-w600/) |
| Aqara W100 Display Sensor | [aqara.com](https://www.aqara.com/eu/product/climate-sensor-w100/) |
| Aqara Temperature Sensor T1 | [aqara.com](https://www.aqara.com/en/product/temperature-and-humidity-sensor-t1) |
| Aqara M200 Hub | [aqara.com](https://www.aqara.com/us/product/hub-m200/) |
| Shelly Plus 1 | [shelly.com](https://www.shelly.com/en-gb/products/product-overview/shelly-plus-1) |
| Shelly Wall Display XL | [shelly.com](https://www.shelly.com/products/shelly-wall-display-xl-black) |
| Midea WF-60A1 WiFi module | [Midea Ukraine](https://www.midea.com.ua/en/products/light-commercial-ac-semi-industrial-conditioners/remote-control-panels/wi-fi-module-wf-60a1) |
| Midea AC LAN (HA integration) | [HACS / GitHub](https://github.com/georgezhao2010/midea_ac_lan) |

---
