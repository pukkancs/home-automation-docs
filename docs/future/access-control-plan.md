# Access Control Plan (PukkancsLak)

*Draft · Version 1.0*

This document describes the **target design** for access control: gates (front outdoor area, garage/waste) and doors (main house, guest house, garage door). Current state is in [current/access-control.md](../current/access-control.md). For property layout see [property overview](../property-overview.md).

---

## 1. Design Principle

**Every gate must retain an open-by-key access method** (mechanical override). All designs and conversions preserve key access.

---

## 2. Gate Hubs (Current and Planned)

We have **2 UniFi Gate Hubs** today:

| Controller | Cameras / Devices | Controlled Access |
| :--- | :--- | :--- |
| **Controller 1** | G6 Turret (right side, road), UniFi Intercom | Main Car Gate, Main Pedestrian Gate |
| **Controller 2** | UA-G3 Reader (Waste Gate) | Garage Car Gate, Waste Storage Gate (fail-safe electric strike lock + UA-G3 Reader at Waste Gate) |

A **new third Gate Hub** will be dedicated to the main house door and garage. It controls the main house front door fail-safe lock and the **outside garage door** (via the garage door opener). The existing two Gate Hubs remain for Controller 1 (Main Car + Pedestrian) and Controller 2 (Garage Car Gate, Waste Gate).

### 2.1 Planned for Controller 1

- Add another camera.
- Change the Main Pedestrian Gate to a **knob-based storage-room-style lock system** and add something similar to **UACC-DoorCloser**.

### 2.2 Planned for Waste Gate

- Convert from electric strike + UA-G3 Reader to **fixed knobs with key** (manual lock only). Key access is retained. Garage Car Gate continues as-is under Controller 2.

---

## 3. Cameras (Planned)

- **1× G6 Turret** per gate, mounted at the top of the house looking at each gate (Main Car, Garage Car, Pedestrian, Waste).
- **1× G5 Turret Ultra** on each side of the house looking towards the back. Dead zones will be addressed later when adding a similar setup for the back garden.

---

## 4. Main House Front Door

**Current:** Dual lock system (two independent locks). See [current/access-control.md](../current/access-control.md#2-doors).

**Future:**

- Add a **UniFi Gate Hub** at the main house (third Gate Hub, dedicated to main house and garage). **CAT6a** required to the door area.
- Convert the **second** lock to a **knob-key storage-room-style** mechanism. Add **1 fail-safe electric lock** paired with this lock for remote/automated control. The **first** (original) lock remains for longer absences and extra security; standard residential locks are hard to convert to electric, hence the dual approach.
- From outside, automated access is provided by the **linked G6 Entry** (intercom / door station).

**Open question:** How to achieve **one-handed exit** without an exit button that is accessible to children? (To be resolved during design or installation.)

---

## 5. Outside Garage Door

**Current:** Garage door opener (standalone).

**Future:**

- Wire the garage door opener to the **main-house Gate Hub** (third Gate Hub), which controls the **outside garage door** via the opener. The main house door fail-safe lock will also be wired to this Gate Hub.
- UniFi Gate Hubs provide relay outputs; garage openers typically need dry-contact or relay inputs for open/close. **Verify with installer** that relay outputs from the Gate Hub can drive the garage opener and how the main house door lock and garage opener share the same hub.
- Add **position sensors** if required for state feedback and automation.

---

## 6. Garage Interior Door (to House)

An **UniFi Access Ultra** and a **fail-safe electric lock** will be added to the **inside door** in the Garage (the door from garage into the house). The Access Ultra is a standalone controller (PoE, network-connected); it drives the electric lock and accepts exit requests from a button. No Gate Hub required for this door. This adds an extra layer of protection when entering the house, since the outside garage door, garage car gate, and main car gate all have remote-control options that reduce physical security.

---

## 7. Guest House Front Door

**Current:** Dual lock system (two independent locks).

**Future:**

- Add a **UniFi Door Hub Mini**. **CAT6a** required — part of the direct Rack→Guest House cable bundle (see [networking master plan](networking-master-plan.md#24-cable-routing--networking-points-strategy)).
- Same lock strategy as main house: convert the **second** lock to **knob-key storage-room-style** and add **1 fail-safe electric lock** paired with it; retain the original method for longer absences and extra security.
- From outside, automated access via the **linked G6 Entry**.

**Open question:** Same as main house — **one-handed exit** without a child-accessible exit button.

---

## 8. Open Items

- **Design constraint:** Every gate must retain an open-by-key access method; all changes preserve mechanical key override.
- Resolve **one-handed exit** for main house and guest house: how to allow exit without an exit button that is accessible to children (e.g. lever handle, auto-unlock on interior handle, or other compliant solution).
- Confirm pedestrian gate mechanism (knob-based storage-room-style + UACC-DoorCloser equivalent) and exact hardware.
- Confirm Waste Gate conversion (electric strike + UA-G3 → fixed knobs with key) and any reader removal.
- **Verify with installer:** relay outputs from main-house Gate Hub driving garage opener; main house door fail-safe lock and garage opener on same hub; wiring and compatibility.
- Confirm Garage interior door: Access Ultra + fail-safe electric lock integration and placement.

---

## 9. Product References

Official product pages and documentation:

| Product | Store | Tech specs |
| :--- | :--- | :--- |
| UniFi Gate Hub | [store.ui.com](https://store.ui.com/us/en/products/ua-hub-gate) | [techspecs.ui.com](https://techspecs.ui.com/unifi/door-access/ua-hub-gate) |
| UniFi Door Hub Mini | [store.ui.com](https://store.ui.com/us/en/products/ua-hub-door-mini) | [techspecs.ui.com](https://techspecs.ui.com/unifi/door-access/ua-hub-door-mini) |
| UniFi Access Ultra | [store.ui.com](https://store.ui.com/us/en/products/ua-ultra) | [techspecs.ui.com](https://techspecs.ui.com/unifi/door-access/ua-ultra) |
| G6 Turret | [store.ui.com](https://store.ui.com/us/en/products/uvc-g6-turret) | [techspecs.ui.com](https://techspecs.ui.com/unifi/cameras-nvrs/uvc-g6-turret) |
| G5 Turret Ultra | [store.ui.com](https://store.ui.com/us/en/pro/category/cameras-dome-turret/products/uvc-g5-turret-ultra) | [techspecs.ui.com](https://techspecs.ui.com/unifi/cameras-nvrs/uvc-g5-turret-ultra) |
| G6 Entry | [store.ui.com](https://store.ui.com/us/en/products/uvc-g6-entry) | [techspecs.ui.com](https://techspecs.ui.com/unifi/door-access/uvc-g6-entry) |
| UA-G3 Reader | [store.ui.com](https://store.ui.com/us/en/products/ua-g3) | [techspecs.ui.com](https://techspecs.ui.com/unifi/door-access/ua-g3) |
| UACC-DoorCloser | [store.ui.com](https://store.ui.com/us/en/products/uacc-doorcloser-us) | [techspecs.ui.com](https://techspecs.ui.com/unifi/door-access/uacc-doorcloser-us) |
