# Home Automation Documentation (PukkancsLak)

*Last updated: 2026-03-11*

Technical documentation for the home automation system. The subsystem documents describe both **Current State** (how it is now) and **Planned / Target** (how it will be). `property-overview.md` is the shared reference document for room names, floors, and outdoor areas.

### Documents

- [**property-overview.md**](property-overview.md) — Building layout, floors, outdoor areas.
- [**networking.md**](networking.md) — Network topology, UniFi equipment, cabling, VLANs, WiFi/Thread.
- [**heating.md**](heating.md) — Heating, cooling, and climate (boiler, UFH, radiators, MDV, guest house AC).
- [**access-control.md**](access-control.md) — Gates, doors, Gate Hubs, cameras, locks.
- [**solar-battery.md**](solar-battery.md) — Poland-focused solar, battery, backup, and V2G-readiness recommendation for the property.

### Cross-References

- [`heating.md`](heating.md) depends on [`networking.md`](networking.md) for HA connectivity, VLANs, Aqara hubs, and the OpenTherm gateway.
- [`access-control.md`](access-control.md) depends on [`networking.md`](networking.md) for door and guest-house cabling planning.
- [`solar-battery.md`](solar-battery.md) feeds into rack resilience planning described in [`networking.md`](networking.md).
