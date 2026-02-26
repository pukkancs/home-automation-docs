# Current Heating & Cooling (PukkancsLak)

This document describes the **current, deployed** heating and cooling setup. For the target design see the [future plan](../future/future-plan.md) and [heating master plan](../future/heating-master-plan.md). For property layout (floors, room names) see [property overview](../property-overview.md).

---

## 1. Boiler

- **Current:** An existing (legacy) boiler is in use. It is not the Viessmann Vitodens 222-F and is not under OpenTherm or Home Assistant control.
- The boiler is fired by the Salus wiring centre when any zone calls for heat.

## 2. UFH Manifold & Wiring Centre

- The **Salus wiring centre** is installed and in use.
- It receives call-for-heat signals from Salus thermostats and drives the UFH manifold actuators and the boiler.

**UFH zones (current):**

| Wiring Centre Channel | Zone | Thermostat |
| :--- | :--- | :--- |
| Zone 1 | Living Room | Wired Salus thermostat |
| Zone 2 | Guest House (Room 2) | Wired Salus thermostat |

So there are **2 UFH zones**: Living Room and Guest House Room 2, each controlled by a **wired Salus thermostat**.

## 3. Radiators

- **Radiator circuit:** The radiator zone is controlled by a **wireless Salus thermostat** located in the **Master Bedroom**. When it calls for heat, the wiring centre enables the radiator circuit and the boiler fires.
- **Radiator heads:** All radiators use **manual thermostatic valve (TRV) heads**. There are no smart TRVs (e.g. Aqara W600) today; room-by-room radiator control is manual via the TRV dials.

## 4. MDV Duct System

- The **MDV duct system** is installed and serves the main house (Living Room, bedrooms, Playroom, Master Bedroom as per the future design).
- It is controlled by its **own remote or wall controller**. It is **not** integrated with Home Assistant or the rest of the automation stack.

## 5. Guest House Air Conditioning

- **2 air conditioning units** are installed in the guest house (one per room).
- They are controlled **only by their original IR remotes**. There is no Aqara M200 hub or Home Assistant integration for these units today.

## 6. Summary: Current vs. Planned

| Subsystem | Current state | Target (see heating master plan) |
| :--- | :--- | :--- |
| Boiler | Legacy boiler, on/off via Salus | Viessmann 222-F, OpenTherm modulation, HA control |
| UFH | 2 zones, wired Salus thermostats (Living Room, Guest House R2) | Aqara W500 thermostats, 2–3 zones, HA schedules |
| Radiators | Wireless Salus stat (Master Bedroom), manual TRVs | W600 TRVs, Shelly rad circuit enable, HA demand logic |
| MDV duct | Installed, standalone control | midea-local, HA interlocks with UFH |
| Guest house AC | 2 units, IR remotes only | Aqara M200 IR control, HA guest-house modes |

---

*Update this document as equipment or wiring changes so it stays an accurate snapshot of what is deployed.*
