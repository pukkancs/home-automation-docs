# Access Control (PukkancsLak)

*Last updated: 2026-02-26*

Gates, doors, Gate Hubs, cameras, and locks. For property layout see [property-overview.md](property-overview.md). For cabling see [networking.md](networking.md).

---

## Current State

### 1. Gate Hubs and Controllers

**2 UniFi Gate Hubs:**

| Controller | Cameras / Devices | Controlled Access |
| :--- | :--- | :--- |
| **Controller 1** | G6 Turret (right side, road), UniFi Intercom | Main Car Gate, Main Pedestrian Gate |
| **Controller 2** | UA-G3 Reader (Waste Gate) | Garage Car Gate, Waste Storage Gate (fail-safe electric strike + UA-G3 Reader) |

**Constraint:** Every gate retains **open-by-key** access (mechanical override).

### 2. Doors

| Door | Current |
| :--- | :--- |
| **Main house front door** | Dual lock system (two independent locks). |
| **Guest house front door** | Dual lock system (two independent locks). |
| **Garage door** | Garage door opener (standalone). |

No UniFi Door Hub or electric locks on house or guest doors. Garage opener not wired to a Gate Hub.

---

## Planned / Target

### 1. Design Principle

**Every gate must retain an open-by-key access method** (mechanical override). All designs preserve key access.

### 2. Gate Hubs

**Third Gate Hub** (new) dedicated to main house door and garage. Controls main house front door fail-safe lock and outside garage door (via opener). Controllers 1 and 2 remain as-is.

**Controller 1 (planned):** Add another camera; change Pedestrian Gate to knob-based storage-room-style lock + UACC-DoorCloser equivalent.

**Controller 2 / Waste Gate (planned):** Convert from electric strike + UA-G3 to **fixed knobs with key** (manual lock only). Garage Car Gate continues as-is.

### 3. Cameras (Planned)

- 1× G6 Turret per gate (Main Car, Garage Car, Pedestrian, Waste).
- 1× G5 Turret Ultra on each side of house looking towards back.

### 4. Main House Front Door

- Add third Gate Hub at main house. **CAT6a** to door area.
- Convert second lock to knob-key storage-room-style + 1 fail-safe electric lock. First lock remains.
- Outside: G6 Entry (intercom / door station).

**Open question:** One-handed exit without child-accessible exit button.

### 5. Outside Garage Door

- Wire opener to main-house Gate Hub (third hub).
- **Verify with installer:** relay outputs from Gate Hub can drive opener; main house lock and opener on same hub.
- Add position sensors if needed.

### 6. Garage Interior Door

**UniFi Access Ultra** + fail-safe electric lock on door from garage into house. PoE, standalone; exit button. No Gate Hub required.

### 7. Guest House Front Door

- Add **UniFi Door Hub Mini**. CAT6a — part of direct Rack→Guest House cable bundle (see [networking.md](networking.md#3-cable-routing--guest-house)).
- Same lock strategy as main house: knob-key + fail-safe electric; retain original for absences.
- Outside: G6 Entry.

**Open question:** One-handed exit without child-accessible exit button.

### 8. Open Items

- Resolve one-handed exit (main house and guest house).
- Confirm pedestrian gate mechanism (knob + UACC-DoorCloser) and hardware.
- Confirm Waste Gate conversion (electric strike → fixed knobs with key).
- Verify Gate Hub relay → garage opener wiring with installer.
- Confirm Access Ultra integration and placement (Garage interior door).

### 9. Product References

| Product | Store / Tech specs |
| :--- | :--- |
| UniFi Gate Hub, Door Hub Mini, Access Ultra | [store.ui.com](https://store.ui.com) / [techspecs.ui.com](https://techspecs.ui.com) |
| G6 Turret, G5 Turret Ultra, G6 Entry, UA-G3, UACC-DoorCloser | [store.ui.com](https://store.ui.com) / [techspecs.ui.com](https://techspecs.ui.com) |
