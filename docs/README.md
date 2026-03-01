# Home Automation Documentation (PukkancsLak)

*Last updated: 2026-02-28*

Technical documentation for the home automation system. Each document has a **Current State** section (how it is now) and a **Planned / Target** section (how it will be).

### Documents

- [**property-overview.md**](property-overview.md) — Building layout, floors, outdoor areas.
- [**networking.md**](networking.md) — Network topology, UniFi equipment, cabling, VLANs, WiFi/Thread.
- [**heating.md**](heating.md) — Heating, cooling, and climate (boiler, UFH, radiators, MDV, guest house AC).
- [**access-control.md**](access-control.md) — Gates, doors, Gate Hubs, cameras, locks.
- [**solar-battery.md**](solar-battery.md) — Solar (8 kW peak) and high-voltage / 20 kWh battery storage.

### Cross-references

- Heating depends on the network (HA, OpenTherm gateway, Aqara hubs).
- Access control cabling is planned in the networking doc.
- Solar/battery will improve rack power resilience (see networking).
