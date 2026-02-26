## Home Automation Roadmap

This document captures our **concrete, technical plans** for how the home automation system should evolve. It is intended to describe specific target states and design decisions, not just ideas or open questions.

### Near-term

This section will focus on planned improvements to the existing networking and heating systems, including configuration changes, hardware upgrades, and integration work.

#### Heating & climate (PukkancsLak)

The detailed design for the heating, cooling, and climate system is described in:

- [`heating-master-plan.md`](heating-master-plan.md) — full technical design for the PukkancsLak smart climate system (boiler, OpenTherm, UFH, radiators, MDV duct AC, guest house AC, sensors, schedules, automations, and risks).

Near-term priorities:

- Select and install the Viessmann Vitodens 222-F with OpenTherm.
- Deploy the Olimex ESP32-POE-ISO OpenTherm gateway with ESPHome and integrate it with Home Assistant.
- Migrate UFH and radiator control to Aqara (W500, W600) while validating the Salus wiring centre vs. installer-recommended alternatives.
- Integrate MDV duct AC, guest house AC (IR), and the office AC into Home Assistant with the safety interlocks defined in the heating master plan.

#### Networking & infrastructure

The underlying network and infrastructure design that supports all home automation subsystems is described in:

- [`networking-master-plan.md`](networking-master-plan.md) — UniFi-based 10GbE backbone, VLAN layout, WiFi/Thread configuration, and core services (Home Assistant, MariaDB, etc.).

Near-term priorities:

- Finalise VLAN and firewall rules for the IoT, Core, Server, and NOT segments.
- Ensure reliable PoE and 10GbE connectivity for Home Assistant, Aqara hubs, OpenTherm gateway, and Unraid.
- Standardise Thread and WiFi channels (e.g. Thread on Ch.25, 2.4 GHz WiFi on non-overlapping channels).

### Mid-term

This section will cover adding new subsystems such as lighting, sensors, alarms, and other components that extend the current setup.

### Long-term

This section will capture more ambitious or experimental ideas for the home automation system that are not yet scheduled but are worth tracking.

