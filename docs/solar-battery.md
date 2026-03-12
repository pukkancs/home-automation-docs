# Solar & Battery Storage (PukkancsLak)

*Last updated: 2026-03-11*

Solar PV and high-voltage battery storage for the property. Relevant to power resilience for the rack (UniFi UPS, core equipment, PoE loads), self-consumption, and future EV / V2G expansion. For property layout see [property-overview.md](property-overview.md). For rack and PoE equipment see [networking.md](networking.md).

---

## Current State

No solar or battery storage installed. The rack (UniFi UPS, core switches, PoE equipment) is powered from the grid. The UniFi UPS 2U provides short-term backup for the rack during power outages.

Existing EV equipment:

- **Vehicle:** **Tesla Model 3 Long Range (2022)**
- **Charger:** **Tesla Wall Connector Gen 3, 3-phase**

---

## Planned / Target

### 1. Design Goal

Design a system for:

- **High self-consumption** for the house across most of the year.
- **~9 months practical self-sufficiency** for the house load only, assuming **8 MWh/year** household electricity use, with **limited cheap-tariff night top-up acceptable from March to November**.
- **EV-ready operation** without sizing the battery around the car: assume **6 MWh/year** of EV charging, charged from daytime surplus when available and from cheap night electricity when solar surplus is low.
- **Strong local monitoring** of inverter, grid, and battery data with mature **Home Assistant** integration and no cloud dependency for core telemetry.
- **Straightforward expansion** later if bidirectional charging / V2G becomes viable.

### 2. Recommended System Size

| Component | Recommended target | Why |
| :--- | :--- | :--- |
| **Solar PV** | **9 kWp** | Hard rooftop limit; use all viable roof area and optimize layout/orientation carefully. |
| **Battery storage** | **13.8-16.6 kWh usable**, high-voltage | Best match for the 9 kWp roof cap, subsidy rules, acceptable night top-up, and future expandability. |
| **Inverter architecture** | **3-phase hybrid, DC-coupled** | Best fit for efficiency, backup capability, and clean battery integration. |
| **Backup design** | **Whole-house backup if budget allows; otherwise critical-load backup** | The garage rack should always remain on backed-up circuits. |

### 3. Sizing Logic

#### What 9 kWp changes

With the roof now limited to **9 kWp**, the system needs to be designed around a different objective:

- maximize what the roof can do
- use the battery for evening / overnight shifting and outage backup
- accept **small cheap-tariff top-ups** in lower-solar periods instead of trying to buy full autonomy with a very large battery

- A household load of **8 MWh/year** means an average of about **22 kWh/day**.
- Typical residential PV yield in Poland is roughly **965-1,080 kWh/kWp/year**, depending on location, roof pitch, azimuth, and shading.
- That means a **9 kWp** array will typically produce around **8.7-9.7 MWh/year** in good conditions.

On annual energy alone, **9 kWp** can cover the house load on paper. The real constraint is **seasonality**, not yearly total:

- spring to early autumn can be very strong
- November through February will still require grid imports
- March and October / November become manageable only if the battery is used intelligently and occasional night top-up is allowed

For this reason:

- **9 kWp** is now the correct PV target because it is the physical roof limit.
- The system can still achieve the desired outcome, but the realistic target becomes **high March-November autonomy with occasional cheap-tariff support**, not strict stand-alone operation.

#### Battery role

The battery should not be sized as if it can solve winter energy shortage on its own. In Poland, home batteries mostly help with:

- shifting daytime solar into evening and overnight house usage
- improving self-consumption in spring, summer, and autumn
- reducing peak imports
- maintaining critical loads during outages
- reducing the amount of cheap-tariff night energy needed on weak-forecast days

For this property, the right first step is now **13.8-16.6 kWh**:

- it fits the roof-limited PV generation better than a 20+ kWh battery
- it is large enough to cover meaningful evening / overnight load shifting
- it supports backup without paying for storage that the 9 kWp roof cannot refill efficiently
- it aligns much better with the likely subsidy floor and the "do not overspend" requirement

The EV should **not** drive the initial battery size. The battery is for the house and resilience; the car can absorb solar surplus directly when available and use off-peak grid charging when not.

### 4. Mój Prąd Context

Recent Polish prosumer programs strongly favor **battery-backed** systems rather than PV-only installs.

Under the recent **Mój Prąd 6.0** structure, the practically relevant points were:

- up to **7,000 PLN** for PV
- up to **16,000 PLN** for electrical storage
- up to **50%** of eligible cost coverage
- maximum combined support of **23,000 PLN** for PV + battery, or **28,000 PLN** if heat storage is also included
- for newer systems, storage had to be at least **1.5 kWh per 1 kWp** of PV

Practical implication for this project:

- design around **PV + electrical storage**, not PV alone
- treat the battery as part of the baseline architecture, not a later afterthought
- ensure the selected installer can provide the documentation needed for subsidy applications and DSO acceptance

If the same logic remains in the next program round, a **9 kWp** PV system implies a minimum qualifying battery of **13.5 kWh**. That means:

- **13.8 kWh** qualifies
- **16.6 kWh** qualifies with more margin
- anything materially larger does **not** improve the subsidy outcome by much; it only increases owner spend

Current market reporting suggests the next edition may shift further toward **battery-first** support, potentially reducing or removing PV support while keeping support for storage additions or storage-heavy installs. This is **not yet final**, but it matters for planning:

- if PV support remains similar to **Mój Prąd 6.0**, the strongest recommendation remains the best overall-value hybrid system
- if support shifts to **0 for PV and battery-only support**, larger battery variants become easier to justify than before
- even in that battery-heavier scenario, the **9 kWp roof cap** still limits how useful very large storage is from a strict ROI perspective

Even if program rules change again, the direction is clear: storage-supported prosumer systems are the safer long-term design choice in Poland.

### 5. Recommended Architecture

Recommended architecture for this property:

- **3-phase hybrid inverter**
- **high-voltage DC-coupled battery**
- **revenue-grade bidirectional grid meter / smart meter**
- **backup output connected either to a whole-house backup board or a dedicated critical-load board**
- **garage rack retained behind the UniFi UPS 2U**, even when the rack is also on backup power

Why this architecture is preferred:

- high-voltage storage is more efficient and cleaner to install than 48 V stacks at this size
- DC coupling reduces unnecessary conversion losses
- 3-phase backup keeps the design aligned with future larger loads and future bidirectional EV charging
- the rack stays protected from short transfer events because the UPS remains in line

### 6. Recommended Implementation For Poland

#### Shortlist baseline

The shortlist is intentionally restricted to systems that have **official Home Assistant integration** and realistic residential market presence in Poland.

**Serious shortlist:**

- **GoodWe ET + Lynx Home F**
- **Fronius Symo GEN24 Plus + BYD Battery-Box Premium HVM**
- **SolaX X3-Hybrid + Triple Power**

**Baseline comparison only:**

- **Deye SUN 10K + Deye BOS-G Pro HV**

The Deye system remains in the document because there is a known real quote for it, but it is **not** treated as a formal shortlist winner because it does not meet the official-HA requirement.

#### 1. GoodWe shortlist option

**Recommended stack:** **GoodWe ET series + GoodWe Lynx Home F**

Why it stays on the shortlist:

- official **Home Assistant** integration
- local integration path with a cleaner HA story than the community-only brands
- strong value profile in the Polish residential market
- official battery pairing that keeps the system coherent

Role in the shortlist:

- **best value shortlist option**
- strongest compromise between price, battery integration, and official HA support

#### 2. Fronius shortlist option

**Recommended stack:** **Fronius Symo GEN24 Plus + BYD Battery-Box Premium HVM + Fronius Smart Meter**

Why it stays on the shortlist:

- best fit for local-first monitoring
- mature, official **Home Assistant** integration over the local Solar API
- strong Poland presence, support, and installer familiarity
- very mature high-voltage battery pairing with BYD

Role in the shortlist:

- **best premium shortlist option**
- best fit if observability, service confidence, and system polish matter more than lowest capital cost

#### 3. SolaX shortlist option

**Recommended stack:** **SolaX X3-Hybrid + Triple Power**

Why it stays on the shortlist:

- official **Home Assistant** integration
- local polling path
- common hybrid residential architecture with official battery ecosystem
- likely to price below Fronius while remaining more standards-compliant than the community-only stacks

Role in the shortlist:

- **watch-the-quote shortlist option**
- worth considering if local quotes beat GoodWe meaningfully

#### 4. Deye baseline comparison

**Baseline stack:** **Deye SUN 10K + Deye BOS-G Pro HV**

Why it remains in the document:

- there is a known quote anchor around **45,000 PLN** for an `8.2 kWp` + `Deye 10K` + `Deye HV ~20.48 kWh nominal` system
- it is likely the strongest value benchmark in the local market

Why it is not in the serious shortlist:

- no official **Home Assistant** integration in the same sense as Fronius, GoodWe, or SolaX
- integration is more community / Modbus driven

#### Updated recommendation under the new baseline

- if the priority is **best value while keeping official HA integration**, start from **GoodWe + Lynx Home F**
- if the priority is **best premium local monitoring and service confidence**, start from **Fronius + BYD**
- if the priority is **quote-sensitive comparison inside the official-HA shortlist**, keep **SolaX + Triple Power** in play
- use **Deye + Deye** as the price / ROI baseline, but not as the preferred architectural recommendation under the official-HA rule

### 7. Home Assistant Forecast-Driven Top-Up

Yes, **Home Assistant can be used** to forecast upcoming solar production and decide whether to top up the battery during the cheap night tariff.

Recommended approach:

- use the official integration for the **chosen shortlisted inverter platform** for inverter / grid / battery telemetry
- use the official **Forecast.Solar** integration for next-day and next-hour solar production estimates
- use local tariff sensors and battery SOC sensors to calculate how much grid charging, if any, should happen overnight

Important limitation:

- **forecasting itself is not local-first**; `Forecast.Solar` is a **cloud polling** integration
- the inverter and battery monitoring can remain local
- battery charge control can be done locally via **Modbus TCP** or equivalent inverter control path, but this should be validated with the final installer / firmware combination

Practical note:

- **Fronius** still offers the cleanest local monitoring story in the shortlist
- **GoodWe** and **SolaX** remain valid for the same automation approach, but the exact entity names and control surface will differ by platform

Recommended automation logic:

1. Run the decision logic each evening after the next-day forecast is reasonably stable.
2. Read:
   - tomorrow PV forecast
   - current battery SOC
   - expected overnight house load
   - expected morning load before solar ramps up
   - cheap tariff window length and price
3. Set a target minimum battery SOC for the end of the cheap window.
4. Charge from grid only if the forecast is too weak to meet the next day's expected load buffer.

Practical control strategy for a **9 kWp + 13.8/16.6 kWh** system:

- **strong forecast day:** do not top up
- **medium forecast day:** top up only to a partial SOC target
- **weak forecast day:** top up to a higher SOC target
- **two weak days in a row:** allow deeper top-up because the battery may not fully recover the following day

Example threshold bands to test and tune after commissioning:

| Tomorrow forecast | Suggested night top-up behavior |
| :--- | :--- |
| **>18 kWh** | No top-up |
| **12-18 kWh** | Top up lightly if SOC is low |
| **7-12 kWh** | Top up to a medium reserve |
| **<7 kWh** | Top up aggressively |

This is exactly the type of automation that makes a **13.8 kWh** battery more viable with a **9 kWp** roof cap.

### 8. EV Charging Strategy

The EV charger should be treated as a **separately controlled flexible load**, not as an automatic extension of the house battery logic.

Target behavior:

- on sunny days / in high-surplus season, charge the car from **real-time excess PV**
- in off-season or on weak-solar days, charge the car from the **cheap night tariff**
- avoid unnecessary battery cycling by **not routing cheap night energy into the house battery first and then into the car**

#### Operational principle

The correct control priority is:

1. **House loads first**
2. **Battery charging to the required reserve / target SOC**
3. **EV charging from genuine export surplus**
4. **Night-grid EV charging only in scheduled low-tariff windows when solar surplus is not expected**

This prevents the common failure mode where:

- the system charges the house battery at night
- the car then charges from the house side
- the battery partially discharges into the EV
- the owner pays for unnecessary battery cycling and conversion losses

#### In-season: charge the EV from excess solar

During sunny periods, EV charging should start only when:

- the battery is already above a minimum target SOC
- current PV production exceeds house load by a safe margin
- export power has remained positive for a short stabilization window

Recommended HA logic:

- monitor `PV power`, `house load`, `battery SOC`, `battery charge/discharge power`, and `grid import/export`
- define an `export surplus threshold`, for example:
  - `>1.8 kW` sustained for single-phase charging start
  - higher threshold if 3-phase charging is required
- start the charger only when surplus is stable for several minutes
- modulate current downward or pause charging if the house begins importing from grid

Practical rule:

- the EV may use **only live excess generation**
- the EV should **not** be allowed to discharge the house battery below the daily reserve just to keep charging

#### Off-season / weak-solar days: charge the EV from cheap night energy

When the forecast for the next day is weak or when seasonal surplus is unavailable, the EV should charge in a **dedicated cheap-tariff night window**.

Recommended HA logic:

- forecast tomorrow's PV output
- determine whether the next day is a `solar EV day` or a `night-charge EV day`
- if it is a `night-charge EV day`, enable EV charging only during the low-tariff window
- optionally reduce or disable battery grid-charging during the same window if the EV is the priority load

Key design goal:

- the EV should take energy **directly from the grid during the cheap window**
- the house battery should either:
  - stay idle, or
  - charge only up to its separate minimum reserve target for house use

That is how the system effectively "bypasses the battery" for EV charging, even though all energy still flows through the same house electrical infrastructure.

#### How to avoid charging the EV from the battery

This is mostly a **controls problem**, not a wiring problem.

Recommended control safeguards:

- set a **minimum battery SOC floor** during EV charging sessions
- allow EV charging only when `grid export > threshold` or when the time is inside the approved cheap-tariff schedule
- if the EV is in cheap-tariff mode, prevent discharge of the house battery below the house reserve target
- stop or reduce EV charging when battery discharge is detected outside the allowed strategy

In other words:

- **solar-surplus mode:** EV follows export surplus
- **night mode:** EV follows tariff window
- **battery reserve mode:** house battery is protected from being drained into the car

#### Recommended charger approach

For this project, the EV charger should support:

- local API or Modbus control
- current modulation
- reliable HA integration
- enough flexibility to survive future changes in EV and inverter strategy

Recommended direction:

- the property already has a **Tesla Wall Connector Gen 3 (3-phase)**, so the default assumption should be to **use the existing Tesla charger rather than replace it**
- Home Assistant has a native **Tesla Wall Connector** integration for local monitoring of Gen 3 units
- that integration is strong for telemetry, but charger-control options are more limited than with EVSEs chosen specifically for local automation
- if the existing Tesla charger can deliver the required solar-surplus and night-tariff control through the chosen HA strategy, keep it
- if fine-grained local current modulation proves too limited in practice, then reconsider a more HA-centric EVSE such as **go-e Charger**
- treat **Fronius Wattpilot** as possible, but not the preferred default for this project because the Home Assistant path is less mature and not first-party in the same way as the Fronius inverter integration

The advantage of a HA-friendly EVSE is that Home Assistant can orchestrate:

- battery reserve targets
- solar forecast logic
- real-time export detection
- tariff-window night charging
- later migration toward V2G-aware logic

Practical implication for the current property:

- **phase 1:** keep and integrate the existing **Tesla Wall Connector**
- use it for scheduled **night-tariff charging** immediately
- evaluate whether the Tesla charger plus HA can provide acceptable **solar-surplus behavior**
- replace the charger only if the desired export-following logic cannot be implemented cleanly enough

#### Recommended operating modes

Suggested EV charging modes to implement in HA:

| Mode | Behavior |
| :--- | :--- |
| **Solar surplus** | Charge only from sustained live export after the battery reserve target is satisfied. |
| **Night tariff** | Charge only during the configured cheap-tariff window; do not intentionally cycle the house battery into the EV. |
| **Disabled / hold** | No EV charging; used on poor-value tariff periods or when battery reserve is prioritized for the house. |

This gives the desired seasonal behavior:

- **spring / summer / sunny shoulder days:** EV mainly charges from PV surplus
- **autumn / winter / low-solar periods:** EV mainly charges from cheap night electricity
- the house battery remains focused on house autonomy and backup, not routine EV energy transfer

#### Tesla-specific note

With the existing **Tesla Model 3 Long Range (2022)** and **Tesla Wall Connector Gen 3**:

- **night-tariff charging** is straightforward and should be treated as the baseline low-solar mode
- **solar-surplus charging** is feasible in principle, but may require a combination of:
  - Tesla Wall Connector telemetry in Home Assistant
  - Tesla vehicle-side charging control
  - additional automation logic or community tooling for dynamic adjustment
- this means the existing Tesla setup is good enough to start with, but it is not the cleanest possible HA-native export-following platform compared with a charger selected specifically for local automation

### 9. Backup Scope

Preferred order of backup design:

1. **Whole-house backup**, if installer cost and board changes remain reasonable.
2. **Critical-load backup**, if whole-house backup is too expensive or restrictive.

Minimum critical-load list:

- garage rack
- Home Assistant host
- core PoE switches, router, APs, Aqara hub
- fridge / freezer circuits
- selected lighting circuits
- garage door / gates if compatible

The current **UniFi UPS 2U** should stay in line for the rack even if the house battery backs the same circuit. The UPS handles ride-through and short disturbances; the house battery handles longer outages.

### 10. Tariff Recommendation

For tariff benchmarking, use **PGE Obrót** as the reference supplier. PGE is the largest electricity supplier in Poland and publishes the main household tariff families relevant to this project.

To keep the ROI model simple while still matching real operation, use **planning import rates** that distinguish:

- **day-rate imports** for normal house consumption
- **night / low-zone imports** for EV charging and battery top-up
- **solar-surplus charging** as avoided imports rather than tariff-priced imports

For planning purposes, use these simplified all-in import rates:

| Tariff | Planning rate | Notes |
| :--- | :--- | :--- |
| **PGE G11** | **~1.14 PLN/kWh flat** | Simple but weakest fit for PV + battery + EV |
| **PGE G12** | **~1.25 PLN/kWh day / ~0.61 PLN/kWh night** | Cheapest option if imported energy is pushed mainly into weekday night hours |
| **PGE G12w** | **~1.30 PLN/kWh day / ~0.69 PLN/kWh low zone** | More flexible than G12 because weekends and holidays are cheap, but not the cheapest night-rate option |

These figures are intentionally simplified:

- they represent realistic **customer import costs** rather than supplier headline energy rates
- they do not attempt to reproduce every invoice line item exactly
- they are suitable for planning-level ROI work without turning the document into a tariff-engineering exercise

Recommended default tariff:

- choose **PGE G12** as the default planning tariff if EV charging and battery top-up are concentrated in night hours
- choose **PGE G12w** only if weekend and holiday low-zone charging flexibility is important enough to justify the slightly higher low-zone rate

Useful timing for automation under the multi-zone tariff:

- cheap zone: typically **22:00-06:00**
- this is the preferred window for:
  - EV charging on low-solar days
  - house-battery top-up when the forecast is weak

### 11. Pricing, Subsidy Scenarios & ROI

Indicative pricing in Poland is highly installer-dependent, so the numbers below are meant as **planning bands**, not procurement targets.

Shortlist cost anchors used in this document:

- **Deye baseline quote:** about **45,000 PLN gross** for `8.2 kWp Longi 530 + Deye SUN 10K + Deye BOS-G Pro HV ~20.48 kWh nominal`
- **GoodWe ET + Lynx Home F:** market signal suggests a system of this class will likely land around **52,000-62,000 PLN gross**
- **SolaX X3-Hybrid + Triple Power:** likely around **50,000-60,000 PLN gross**
- **Fronius + BYD HVM:** likely around **62,000-78,000 PLN gross** for a comparable `8-9 kWp` residential system

Realistic installed-system guidance under the new shortlist baseline:

| Variant | Likely installed range | Comment |
| :--- | :--- | :--- |
| **GoodWe + Lynx Home F + 8-9 kWp PV** | **~52,000-62,000 PLN gross** | Best value within the official-HA shortlist |
| **SolaX + Triple Power + 8-9 kWp PV** | **~50,000-60,000 PLN gross** | Similar lane to GoodWe; quote-sensitive |
| **Fronius + BYD + 8-9 kWp PV** | **~62,000-78,000 PLN gross** | Premium shortlist option |
| **Deye + Deye + ~8.2 kWp PV** | **~45,000 PLN gross** | Price baseline only, not shortlist-compliant on HA |

For ROI modelling below, use these planning assumptions:

- **9 kWp** PV annual yield: roughly **8.7-9.7 MWh/year**
- house load: **8 MWh/year**
- EV load: **6 MWh/year**
- tariff benchmark:
  - **house daytime imports:** `PGE G12 day ~1.25 PLN/kWh`
  - **EV charging and battery top-up imports:** `PGE G12 night ~0.61 PLN/kWh`
- solar export value in net-billing: modeled conservatively, so ROI numbers are **indicative**, not bank-grade
- EV charging is assumed to happen either:
  - from **solar surplus**, or
  - from the **night tariff**
- simple payback only; no financing cost, degradation curve, or maintenance reserve included

Model the following subsidy scenarios:

| Subsidy scenario | Assumption used in this document |
| :--- | :--- |
| **Scenario A: PV + battery support** | **23,000 PLN** total support, consistent with `Mój Prąd 6.0` style PV + battery support |
| **Scenario B: Battery-only support** | **23,000 PLN** support for storage only, with **0 PLN** support assumed for PV |
| **Scenario C: No support** | **0 PLN** |

Indicative annual savings using the simplified **day / low-zone planning model**:

| Variant | Indicative annual savings | Notes |
| :--- | :--- | :--- |
| **GoodWe + Lynx Home F + 8-9 kWp PV** | **~5,700-6,700 PLN/year** | Best value inside the official-HA shortlist |
| **SolaX + Triple Power + 8-9 kWp PV** | **~5,600-6,600 PLN/year** | Similar expected economics to GoodWe |
| **Fronius + BYD + 8-9 kWp PV** | **~5,900-7,000 PLN/year** | Best premium shortlist option |
| **Deye + Deye + ~8.2 kWp PV** | **~5,600-6,800 PLN/year** | Value baseline, not shortlist-compliant |

Indicative simple payback under the simplified day / low-zone planning model:

| Variant | With `23k` PV + battery subsidy | With `23k` battery-only subsidy | With `0k` subsidy |
| :--- | :--- | :--- | :--- |
| **GoodWe + Lynx Home F + 8-9 kWp PV** | **~4.7-7.2 years** | **~4.7-7.2 years** | **~7.8-11.0 years** |
| **SolaX + Triple Power + 8-9 kWp PV** | **~4.5-6.9 years** | **~4.5-6.9 years** | **~7.5-10.5 years** |
| **Fronius + BYD + 8-9 kWp PV** | **~5.8-8.8 years** | **~5.8-8.8 years** | **~9.4-13.2 years** |
| **Deye + Deye + ~8.2 kWp PV** | **~3.3-5.3 years** | **~3.3-5.3 years** | **~6.6-9.0 years** |

Interpretation using the simplified day / low-zone planning model:

- **best pure ROI overall:** `Deye + Deye`, based on the known quote anchor
- **best ROI inside the official-HA shortlist:** `SolaX + Triple Power`, closely followed by `GoodWe + Lynx Home F`
- **best premium shortlist option:** `Fronius + BYD`
- **best practical shortlist balance:** `GoodWe + Lynx Home F`

Effect of tariff choice on ROI:

- moving from **G11** to a well-used multi-zone tariff is beneficial because it lowers the cost of:
  - EV charging in low-solar months
  - battery top-up on weak-forecast nights
- for this house, **G12** is the better default economic assumption if most imported energy lands in weekday night hours
- **G12w** remains useful if weekends and holidays are important charging windows, but it should be treated as the more flexible tariff rather than the cheapest one
- relative to **G11**, a well-used multi-zone tariff should shorten simple payback by roughly **0.5-1.5 years**, depending on how much EV charging actually lands in night or low-zone hours and how aggressively HA shifts battery top-up away from the day rate

Updated recommendation under the new assumptions:

- if the next `Mój Prąd` is still broadly **PV + battery**, start with **GoodWe + Lynx Home F** as the default shortlist option
- if the next `Mój Prąd` becomes effectively **battery-first / PV-zero**, `GoodWe + Lynx Home F` and `Fronius + BYD` both remain valid, with `Fronius + BYD` easier to justify if premium monitoring and support are valued
- use `Deye + Deye` to challenge the shortlist on economics, but not as the preferred recommendation under the official-HA rule

Recommended spending strategy:

- **best pure ROI overall:** `Deye + Deye + PGE G12`
- **best shortlist value:** `GoodWe + Lynx Home F + PGE G12`
- **best shortlist quote-sensitive alternative:** `SolaX + Triple Power + PGE G12`
- **best premium / smart-home fit:** `Fronius + BYD + PGE G12`

### 12. V2G / V2H Expansion Readiness

The first installation should be made **V2G-ready**, even if no bidirectional EV is available yet.

Current vehicle reality:

- the existing **Tesla Model 3 Long Range (2022)** should be treated as a **non-V2G / non-V2H vehicle for planning purposes**
- even if third-party experiments exist, there is no mature official European Tesla path that should be assumed for this project
- therefore, future V2G readiness should be understood as:
  - readiness for a **future supported vehicle**
  - or readiness for a future **officially supported Tesla bidirectional ecosystem**, if Tesla eventually enables one

Design requirements:

- reserve **switchboard space** for a future bidirectional EV charger and protection devices
- install **spare conduit and CAT6a** from the main electrical area to the car charging location
- keep **revenue-grade metering / CT layout** simple and documented so a future bidirectional EVSE can be added without rewiring the whole system
- avoid locking EV charging control to a cloud-only vendor ecosystem
- prefer an installer comfortable with **dynamic export control**, smart metering, and HA-friendly local integrations

Practical strategy:

- build the house PV and battery system first
- add smart unidirectional EV charging in phase 1
- later, if Tesla enables useful bidirectional support in Europe or the vehicle changes to a supported V2G/V2H model, add the compatible charger and integrate it into the existing metering and energy management design

### 13. Open Items & Pre-Install Decisions

| # | Item | Owner | Status |
| :--- | :--- | :--- | :--- |
| 1 | Confirm exact roof layout, azimuth, pitch, and shading to maximize the fixed **9 kWp** roof allocation. | Owner / installer | Blocking final panel layout |
| 2 | Confirm DSO / utility constraints for inverter export power and backup configuration. | Owner / installer | Blocking final inverter size |
| 3 | Decide between **whole-house backup** and **critical-load backup**. | Owner / installer | Affects switchboard scope and cost |
| 4 | Confirm the garage rack circuit path so the UniFi UPS remains in line on a backed-up feed. | Owner / installer | Important for network resilience |
| 5 | Confirm battery room / mounting location, ventilation, and cable route. | Owner / installer | Required before final quote |
| 6 | Choose an EV charger with local API / HA-friendly control and confirm whether it can modulate current based on export surplus and tariff schedules. | Owner / installer | Important for EV strategy |
| 7 | Confirm EV charger location and add spare conduit / data cabling for future bidirectional charging. | Owner / installer | Important for V2G readiness |
| 8 | Request installer quotes for **Fronius + BYD HVM 13.8**, **Fronius + BYD HVM 16.6**, and **Sungrow + SBR 16.0** using the same PV, backup scope, and EV-charger-control assumptions. | Owner / installer | Needed for price and ROI comparison |
| 9 | Confirm whether **PGE G12** is available for the exact supply point and whether meter / settlement changes are needed. | Owner / installer / supplier | Important for tariff strategy |
| 10 | Confirm whether the chosen inverter firmware and installer policy allow local battery charge control for HA-driven night top-up automation. | Owner / installer | Important for automation strategy |
| 11 | Validate whether the existing **Tesla Wall Connector Gen 3** can deliver acceptable HA-controlled solar-surplus charging behavior, or whether a future charger replacement should remain an option. | Owner / installer | Important for EV control strategy |
