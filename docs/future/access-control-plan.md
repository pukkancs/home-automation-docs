# Access Control Plan (PukkancsLak)

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

### 2.1 Planned for Controller 1

- Add another camera.
- Change the Main Pedestrian Gate to a **knob-based storage-room-style lock system** and add something similar to **UACC-DoorCloser**.

### 2.2 Planned for Waste Gate

- Convert from electric strike + UA-G3 Reader to **fixed knobs with key** (manual lock only). Key access is retained. Garage Car Gate continues as-is under Controller 2.

---

## 3. Cameras (Planned)

- **1× G6 Turret** per gate, mounted at the top of the house looking at each gate (Main Car, Garage Car, Pedestrian, Waste).
- **1× G6 Ultra** on each side of the house looking towards the back. Dead zones will be addressed later when adding a similar setup for the back garden.

---

## 4. Main House Front Door

**Current:** Dual lock system (two independent locks). See [current/access-control.md](../current/access-control.md#2-doors).

**Future:**

- Add a **UniFi Gate Hub** at the main house. **CAT6a** required to the door area.
- Convert the **second** locking system to a **knob-key storage-room-style** mechanism with **2 fail-safe electric locks**. The **first** (original) lock remains for longer absences and extra security; standard residential locks are hard to convert to electric, hence the dual approach.
- From outside, automated access is provided by the **linked G6 Entry** (intercom / door station).

**Open question:** How to achieve **one-handed exit** without an exit button that is accessible to children? (To be resolved during design or installation.)

---

## 5. Garage Door

**Current:** Garage door opener (standalone).

**Future:**

- Wire the garage opener to the **UniFi Gate Hub** (Controller 2 or main house Gate Hub as designed).
- Add **position sensors** if required for state feedback and automation.

---

## 6. Guest House Front Door

**Current:** Dual lock system (two independent locks).

**Future:**

- Add a **UniFi Door Hub Mini**. **CAT6a** required to the guest house.
- Same lock strategy as main house: add **2 fail-safe electric locks** to the **second** locking system, converted to **knob-key storage-room-style**; retain the original method for longer absences and extra security.
- From outside, automated access via the **linked G6 Entry**.

**Open question:** Same as main house — **one-handed exit** without a child-accessible exit button.

---

## 7. Open Items

- **Design constraint:** Every gate must retain an open-by-key access method; all changes preserve mechanical key override.
- Resolve **one-handed exit** for main house and guest house: how to allow exit without an exit button that is accessible to children (e.g. lever handle, auto-unlock on interior handle, or other compliant solution).
- Confirm pedestrian gate mechanism (knob-based storage-room-style + UACC-DoorCloser equivalent) and exact hardware.
- Confirm Waste Gate conversion (electric strike + UA-G3 → fixed knobs with key) and any reader removal.
- Confirm garage opener integration with Gate Hub and whether position sensors are required.
