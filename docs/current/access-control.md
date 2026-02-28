# Current Access Control (PukkancsLak)

*Last updated: 2026-02-26*

This document describes the **current, deployed** access control (gates and doors). For the target design see the [future plan](../future/future-plan.md) and [access control plan](../future/access-control-plan.md). For property layout see [property overview](../property-overview.md).

---

## 1. Gate Hubs and Controllers

We have **2 UniFi Gate Hubs**:

| Controller | Cameras / Devices | Controlled Access |
| :--- | :--- | :--- |
| **Controller 1** | G6 Turret (right side, road), UniFi Intercom | Main Car Gate, Main Pedestrian Gate |
| **Controller 2** | UA-G3 Reader (Waste Gate) | Garage Car Gate, Waste Storage Gate (fail-safe electric strike lock + UA-G3 Reader) |

**Important:** Every gate must retain an **open-by-key** access method (mechanical override).

**Planned changes for Controller 1:** Add another camera; change the Pedestrian Gate to a knob-based storage-room-style lock system and add something similar to UACC-DoorCloser.

**Planned changes for Waste Gate:** Convert from electric strike + UA-G3 Reader to **fixed knobs with key** (manual lock). Key access is retained as above.

## 2. Doors

| Door | Current |
| :--- | :--- |
| **Main house front door** | Dual lock system (two independent locks). |
| **Guest house front door** | Dual lock system (two independent locks). |
| **Garage door** | Garage door opener (standalone). |

No UniFi Door Hub or electric locks on house or guest doors today. Garage opener is not yet wired to a Gate Hub.

---

*Update this document as access control changes so it stays an accurate snapshot of what is deployed.*
