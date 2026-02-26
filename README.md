## Home Automation Documentation

This repository is the central, technical documentation for our home automation setup. The goal is to clearly describe **what we currently have** and **what we are aiming to achieve**, with concrete decisions instead of open-ended questions.

For now, the focus is on:

- **Networking**: How devices connect, communicate, and access external services.
- **Heating**: How we control heating, zones, schedules, and integration with automation.

Over time, this documentation will expand to cover:

- **Lighting**
- **Sensors**
- **Alarms**
- **Other subsystems** that become part of the home automation ecosystem.

This repository is meant to be a **single source of truth** that evolves alongside the actual system.

### Documents

**Reference:**

- [`docs/property-overview.md`](docs/property-overview.md) — property layout (5 half floors, outdoor areas). Link from other docs when room or area names are needed.

**Current state** (what is actually deployed today):

- [`docs/current/`](docs/current/) — [networking](docs/current/networking.md), [access control](docs/current/access-control.md), [heating](docs/current/heating.md).

**Future plans and roadmap:**

- [`docs/future/future-plan.md`](docs/future/future-plan.md) — roadmap and links to master plans.
- [`docs/future/heating-master-plan.md`](docs/future/heating-master-plan.md) — heating & climate (PukkancsLak) detailed design.
- [`docs/future/networking-master-plan.md`](docs/future/networking-master-plan.md) — networking backbone, VLANs, WiFi/Thread, core services.
- [`docs/future/access-control-plan.md`](docs/future/access-control-plan.md) — gates, doors, Gate Hubs, cameras, locks.
- [`docs/future/solar-battery-plan.md`](docs/future/solar-battery-plan.md) — solar (8 kW peak) and high-voltage / 20 kWh battery storage plan.

