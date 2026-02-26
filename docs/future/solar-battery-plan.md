# Solar & Battery Storage Plan (PukkancsLak)

This document describes the planned **solar PV** and **high-voltage battery storage** for the property. It is separate from the networking and heating plans but relevant to power resilience for the rack (UniFi UPS, core equipment, PoE loads).

For property layout see [property overview](../property-overview.md). For how the rack and PoE equipment are powered today see [current/networking.md](../current/networking.md#1-core-equipment).

---

## 1. Target System

| Component | Target | Notes |
| :--- | :--- | :--- |
| **Solar PV** | **8 kW peak** | Grid-tied; inverter and panel placement TBD (roof, orientation, shading). |
| **Battery storage** | **20 kWh** or **high-voltage battery** (preferred) | DC-coupled or AC-coupled; high-voltage stack preferred for efficiency and longevity. |

The combination provides:

- **Self-consumption** of solar during the day (reduced grid draw for house and rack).
- **Backup / resilience** when the battery is configured for backup: the UniFi UPS in the garage rack can be supplemented or eventually replaced by the whole-house battery for longer runtime during grid outages.
- **Peak shaving** (optional): use battery to avoid high grid tariffs if time-of-use or demand charges apply.

---

## 2. Design Principles

- **High-voltage battery preferred** over low-voltage (48 V) stacks where feasible: better efficiency, fewer amps, simpler wiring, and often better cycle life.
- **Integration with existing backup:** The current UniFi UPS 2U in the rack will continue to protect core networking and PoE loads; the solar/battery system may feed the same cabinet or the main distribution board depending on final design. Clarify with installer how the battery output interfaces with the garage feed and the UPS.
- **Sizing:** 8 kW peak solar and 20 kWh storage are initial targets; final sizing depends on consumption profile, roof area, and local regulations.

---

## 3. Open Items & Pre-Install Decisions

| # | Item | Owner | Status |
| :--- | :--- | :--- | :--- |
| 1 | Confirm roof area, orientation, and shading for 8 kW peak (or adjusted capacity). | Owner / installer | Before system design |
| 2 | Choose AC-coupled vs DC-coupled architecture; confirm inverter and battery brand/model. | Owner / installer | Affects compatibility and backup behaviour |
| 3 | Decide whether battery is backup-capable (island mode / critical loads panel) and which circuits (e.g. garage rack) are on backup. | Owner / installer | Affects panel layout and UPS strategy |
| 4 | Clarify interface between battery output and garage rack supply (UPS remains in line vs. battery feeding rack directly). | Owner / installer | Ensures network stays up during outage as intended |
| 5 | Local permits, grid connection agreement, and feed-in / net metering rules. | Owner / installer | Blocking for installation |

---

*This document will be updated as the design is firmed up and installation progresses. Next review: after installer site survey and first draft design.*
