# Stage 5 — PCB Design Rules

Status: `REV.A LAYOUT PREFLIGHT COMPLETE — PLACEMENT FROZEN FOR ROUTING`

This file is the Stage 5 owner for the Rev.A PCB manufacturing and Altium rule baseline. User-confirmed Altium configuration and placement-closeout evidence are recorded below; this does not prove routing, repour, DRC, thermal implementation, or manufacturing readiness.

## Manufacturing Baseline

| Item                 | Rev.A baseline        | Applicability / note                                                                      |
| -------------------- | --------------------- | ----------------------------------------------------------------------------------------- |
| Manufacturer         | JLCPCB / 嘉立创          | Target manufacturer for Rev.A                                                             |
| PCB type             | Standard rigid PCB    | No flex, rigid-flex, or HDI requirement                                                   |
| Material             | FR-4                  | Standard construction                                                                     |
| Layer count          | 2 Layer               | Rev.A baseline                                                                            |
| Finished thickness   | 1.6 mm                | Finished-board baseline                                                                   |
| Copper weight        | 1 oz / 35 µm          | Rev.A baseline                                                                            |
| Solder mask          | Green                 | Both sides as applicable                                                                  |
| Silkscreen           | White                 | Both sides as applicable                                                                  |
| Surface finish       | Lead-Free HASL / 无铅喷锡 | Rev.A baseline                                                                            |
| Controlled impedance | Not required          | USB data is not used                                                                      |
| Assembly             | Manual assembly       | Small exposed-pad power ICs use solder paste plus heating-plate / reflow-style processing |
| PCBA service         | Not required          | Current baseline                                                                          |

Manufacturer capability is not a Project design rule. JLCPCB minimum manufacturing limits must not be copied directly into the default Project rules; the values below are Project design values with deliberate margin.

## Mechanical Baseline

Rev.A mechanical design baseline: `FROZEN / DOCUMENTED`. This records stable design intent; it is not verification of the exact EDA mechanical implementation.

| Item | Rev.A baseline | Verification boundary |
| --- | --- | --- |
| Nominal board size | 55 mm × 35 mm | No further board-outline reduction is planned. The nominal coordinate origin is the PCB lower-left corner, `(0, 0)`. Actual Board Shape, corner geometry, and exact edge geometry remain `.PcbDoc` implementation authority. |
| PCB construction | 2-layer FR-4, 1.6 mm, 1 oz | Manufacturing baseline documented; actual EDA implementation `NOT YET VERIFIED`. |
| Mounting-hole pattern | Left Bottom `(4, 4)` mm; Right Bottom `(51, 4)` mm; Left Top `(4, 31)` mm; Right Top `(51, 31)` mm | Nominal center-position pattern only, with nominal board-edge inset of 4 mm. Hole diameter, drill size, plating, washer diameter, screw-head diameter, and exact mechanical keepout `REQUIRES EDA / MECHANICAL VERIFICATION` where applicable. |
| USBC1 orientation | Left board edge; USB insertion/removal toward board exterior | Exact connector edge clearance and body overhang `REQUIRES EDA / 3D MECHANICAL VERIFICATION`. |
| J_NTC1 orientation | Top-side interface region; cable access from board exterior | Exact connector edge clearance and body relationship `REQUIRES EDA / 3D MECHANICAL VERIFICATION`. |
| J_BAT1 orientation | Top-side interface region; cable access from board exterior | Exact connector edge clearance and body relationship `REQUIRES EDA / 3D MECHANICAL VERIFICATION`. |
| J_OUT1 orientation | Right board edge; wire-entry side faces outward toward the right board exterior | Exact body / board-edge relationship `REQUIRES EDA / 3D MECHANICAL VERIFICATION`. |

This baseline does not treat screenshot review as exact mechanical verification and does not claim placement, mechanical-clearance, manufacturing-clearance, routing, DRC, thermal, polygon, or manufacturing PASS.

## General Altium Rules

Primary units are mil. Parenthetical metric values are informative conversions only.

| Rule                                              | Value                                                                       | Scope                                                                            | Priority / applicability                                        | Basis / note                                                                                 | EDA status         |
| ------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------ |
| Electrical Clearance                              | Minimum 6 mil (0.1524 mm)                                                   | Default board-wide copper clearance                                              | Default; lower than any approved local exception                | Project default design value, not a JLCPCB manufacturing limit                               | `NOT YET VERIFIED` |
| Width_Default                                     | Min 6 mil; Preferred 8 mil; Max 40 mil                                      | `All`                                                                           | Default width rule; lower priority than `Width_Power`           | Applies to ordinary signal nets                                                     | `NOT YET VERIFIED` |
| Width_Power                                       | Min 12 mil; Preferred 30 mil; Max 100 mil                                   | `InNetClass('POWER')`                                                          | Higher priority than `Width_Default`                            | Short neck-down is allowed at IC pads; restore wider copper promptly after leaving the pad | `PENDING ALTIUM RECONFIGURATION` |
| Routing Via Style — Standard                      | Diameter 24 mil; Hole 12 mil (about 0.61 / 0.305 mm)                        | Ordinary through vias                                                            | Default via rule                                                | Does not automatically apply to exposed-pad thermal vias                                     | `NOT YET VERIFIED` |
| Board Outline Clearance                           | Copper to board edge 12 mil (0.3048 mm)                                     | All copper relative to finished board edge                                       | Board-wide mechanical rule                                      | Preserve the numeric intent if the installed Altium version uses a different canonical label | `NOT YET VERIFIED` |
| Component Clearance                               | 20 mil (0.508 mm)                                                           | Components / component bodies                                                    | Default placement rule                                          | Footprint courtyard and mechanical constraints may require more clearance                    | `NOT YET VERIFIED` |
| Solder Mask Expansion                             | 2 mil (0.0508 mm)                                                           | Ordinary pads                                                                    | Default mask rule                                               | Footprint-specific mask geometry takes priority where explicitly verified                    | `NOT YET VERIFIED` |
| Minimum Solder Mask Sliver                        | 4 mil (0.1016 mm)                                                           | Board-wide solder-mask webs                                                      | Default manufacturing rule                                      | Fine-pitch feasibility must be resolved locally, not by silently changing this baseline      | `NOT YET VERIFIED` |
| Silk To Silk Clearance                             | 6 mil (0.1524 mm)                                                          | Silkscreen relative to silkscreen                                                 | Default silkscreen clearance                                    | Current Rev.A compact-board project design baseline; not a manufacturing PASS claim          | `CONFIRMED BY USER IN ALTIUM` |
| Silk To Solder Mask / Solderable Copper Clearance | 6 mil (0.1524 mm)                                                          | Silkscreen relative to pads and other exposed solderable copper                  | Default silkscreen clearance                                    | Current Rev.A compact-board project design baseline; final manufacturing review remains Stage 7 | `CONFIRMED BY USER IN ALTIUM` |
| Silkscreen — Ordinary Designators                 | Height 40 mil; Stroke 8 mil                                                 | Ordinary component references                                                    | Default legend style                                            | Maintain readability after fabrication                                                       | `NOT YET VERIFIED` |
| Silkscreen — Important Interface / Polarity       | Height 50–60 mil; Stroke 8–10 mil                                           | User-facing labels such as `USB`, `BAT`, `BAT+`, `BAT-`, `NTC`, `3V3`, and `GND` | Use in preference to ordinary legend style for important labels | Final size within the range depends on available mechanical space                            | `NOT YET VERIFIED` |

### Routing and Polygon Rule Baseline

`POWER` is the sole power Net Class. Its members are `VBUS_PROT`, `SYS`, `BAT`, `3V3`, `3V3_PROT`, and `GND`. Power-routing rules use `InNetClass('POWER')`; no per-net power-width expressions are retained. `GND` belongs to the class for consistent rule management but is not required to be routed as an ordinary power trace: it is primarily implemented through the polygon / ground-plane strategy, with local routing or stitching only where required.

The polygon-connect baseline is one rule named `PolygonConnect`, scoped to `All`: SMD pads and vias use Direct Connect; through-hole pads use Relief Connect. Altium Advanced settings may represent those three object types separately within this one rule. A higher-priority, narrowly scoped exception is added only for a verified, real manufacturing or component-specific need.

### Scoped Exceptions

- If actual fine-pitch pad geometry cannot satisfy the 6 mil default clearance, create a narrowly scoped, higher-priority rule or documented exception for the affected footprint/pad objects. Do not reduce the board-wide default clearance.
- Fine-pitch and exposed-pad mask/paste geometry may follow a verified manufacturer-recommended land pattern through footprint-specific implementation. Global mask or paste rules must not overwrite those requirements.
- Exposed-pad thermal-via diameter, drill, pitch, count, tenting/filling, and paste segmentation are footprint/layout decisions; the 24 / 12 mil standard-via baseline does not define them.

## Layer / Copper Strategy

| Layer | Rev.A layout strategy | Verification boundary |
| --- | --- | --- |
| Top Layer | Components, critical power loops, primary routing, and local copper | Strategy only; implementation `NOT YET VERIFIED` |
| Bottom Layer | Keep as a continuous GND plane / polygon as practical | Strategy only; continuity, return paths, and repour `NOT YET VERIFIED` |

This is a Layout Strategy / Rule Baseline, not a claim that the PCB implementation is complete. It must be verified during actual placement and routing.

## Project-specific Layout Inputs

| Area | Required layout input | Status |
| --- | --- | --- |
| TPS63802 / M4 | Place the VIN capacitor, VOUT capacitors, and L1 close to the IC; keep the switching-current loop short and compact; keep `FB_3V3` away from switching nodes; keep power paths short and wide. The L1 footprint, pad geometry, and pin mapping were confirmed by the user in Altium. | `CONFIRMED BY USER IN ALTIUM`; routing and copper implementation `NOT YET VERIFIED` |
| BQ25185 / Charger | Place IN / SYS / BAT decoupling close to the device; keep high-current paths short and low-resistance. The exposed-pad implementation was confirmed by the user in Altium; final thermal implementation remains downstream work. | `CONFIRMED BY USER IN ALTIUM`; thermal-via geometry and final copper `NOT YET VERIFIED` |
| USB-C / Protection | Verify connector / ESD / TVS physical ordering and return path during layout. Controlled impedance is not required because USB data is not used. | `NOT YET VERIFIED` |
| Exposed-pad devices | For BQ25185, TPS63802, and TPS259474, the critical exposed / ground-pad implementations were confirmed by the user in Altium. Paste, thermal-pad, and thermal-via geometry must follow the verified footprint and manufacturer guidance; global paste-mask rules must not substitute for footprint-specific implementation. | `CONFIRMED BY USER IN ALTIUM`; final thermal-via geometry and copper `NOT YET VERIFIED` |

## Evidence / Verification Status

| Evidence item | Current status | Meaning |
| --- | --- | --- |
| Documentation baseline freeze | `COMPLETE` | Manufacturing, general PCB-rule, and mechanical-baseline intent is recorded in this authority document |
| Altium PCB rules configured | `PENDING ALTIUM RECONFIGURATION` | The documented baseline requires the single `POWER` class, `Width_Power`, `Width_Default`, and one board-wide `PolygonConnect` rule |
| Rule Scope / Priority manually checked in Altium | `PENDING ALTIUM RECONFIGURATION` | Recheck the class members, the two width-rule scopes and ordering, and the one polygon-connect rule after reconfiguration |
| Critical footprint / pin-pad mapping | `CONFIRMED BY USER IN ALTIUM` | Includes M4 L1, connector numbering / polarity, mounting holes, and critical exposed-pad implementations |
| PCB placement | `COMPLETE — ACCEPTED FOR STAGE 5 CLOSEOUT` | Based on user EDA implementation and interactive visual review; exact clearance, routing, and DRC are not claimed |
| Routing | `NOT YET VERIFIED` | No routing PASS claim |
| Polygon / Repour | `NOT YET VERIFIED` | No copper-continuity or repour PASS claim |
| PCB DRC | `NOT YET VERIFIED` | No DRC result was produced by this documentation update |
| Layout Preflight | `COMPLETE / PASS` | User-confirmed EDA preflight and accepted whole-board placement; this does not approve downstream routing, DRC, thermal, or manufacturing work |
| Manufacturing release | `NOT AUTHORIZED` | This document cannot approve fabrication |

Stage 5 closeout is based on user-confirmed Altium evidence and interactive visual review. It is not PCB DRC PASS, routing PASS, polygon / repour PASS, thermal PASS, or manufacturing approval.
