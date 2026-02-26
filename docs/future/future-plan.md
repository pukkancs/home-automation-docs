# Home Automation Roadmap

This document captures our **concrete, technical plans** for how the home automation system should evolve. It points to detailed master plans for each subsystem; there is no near/mid/long-term breakdown for now.

### Heating & climate

The full target design for heating, cooling, and climate is in:

- **[`heating-master-plan.md`](heating-master-plan.md)** — boiler (Viessmann 222-F, OpenTherm), UFH (Aqara W500), radiators (W600 TRVs, Shelly rad circuit), MDV duct integration, guest house AC (Aqara M200 IR), office AC (Gree/LG), sensors, schedules, automations, and risks.

Current state is documented in [**current/heating.md**](../current/heating.md).

### Networking & infrastructure

The full target design for the network backbone and core services is in:

- **[`networking-master-plan.md`](networking-master-plan.md)** — UniFi 10GbE topology, VLANs, WiFi/Thread configuration, Home Assistant host, MariaDB on Unraid, and security rules.

Current state is documented in [**current/networking.md**](../current/networking.md).

### Access control

Gates (Main Car, Garage Car, Pedestrian, Waste) and doors (main house, guest house, garage) with UniFi Gate Hubs, cameras, and electric locks.

- **[`access-control-plan.md`](access-control-plan.md)** — target design for Gate Hubs, cameras, doors, and locks.

Current state is documented in [**current/access-control.md**](../current/access-control.md).

### Solar & battery storage

Planned **8 kW peak solar** and **20 kWh or high-voltage battery storage** for self-consumption and backup. The rack (UniFi UPS, core equipment, PoE loads) will benefit from this once the system is installed and integrated.

- **[`solar-battery-plan.md`](solar-battery-plan.md)** — target system, design principles, and open items.

### Other subsystems

Lighting, sensors, alarms, and other components will be added to the roadmap and to current as they are introduced. For now the documentation focus remains networking, heating, and (as planned) solar and battery.
