# Stage 5 — PCB Design Rules

Status: `REV.A DOCUMENTATION BASELINE FROZEN — LAYOUT PREFLIGHT NOT YET COMPLETE`

This file is the Stage 5 owner for the Rev.A PCB manufacturing and Altium rule baseline. It records design intent only: it does not prove Altium configuration, placement, routing, repour, DRC, or manufacturing readiness.

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

## General Altium Rules

Primary units are mil. Parenthetical metric values are informative conversions only.

| Rule                                              | Value                                                                       | Scope                                                                            | Priority / applicability                                        | Basis / note                                                                                 | EDA status         |
| ------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------ |
| Electrical Clearance                              | Minimum 6 mil (0.1524 mm)                                                   | Default board-wide copper clearance                                              | Default; lower than any approved local exception                | Project default design value, not a JLCPCB manufacturing limit                               | `NOT YET VERIFIED` |
| Routing Width — Signal                            | Min 6 mil; Preferred 8 mil; Max 40 mil                                      | Ordinary signal nets                                                             | Default signal rule                                             | Use a higher-priority scoped rule where a named net class requires it                        | `NOT YET VERIFIED` |
| Routing Width — Power                             | Min 12 mil; Preferred 30 mil; Max reasonable implementation-dependent value | Main power paths including `VBUS_PROT`, `SYS`, `BAT`, `3V3`, and `3V3_PROT`      | Higher priority than ordinary signal width for matching nets    | Short neck-down is allowed at IC pads; restore wider copper promptly after leaving the pad   | `NOT YET VERIFIED` |
| Routing Via Style — Standard                      | Diameter 24 mil; Hole 12 mil (about 0.61 / 0.305 mm)                        | Ordinary through vias                                                            | Default via rule                                                | Does not automatically apply to exposed-pad thermal vias                                     | `NOT YET VERIFIED` |
| Board Outline Clearance                           | Copper to board edge 12 mil (0.3048 mm)                                     | All copper relative to finished board edge                                       | Board-wide mechanical rule                                      | Preserve the numeric intent if the installed Altium version uses a different canonical label | `NOT YET VERIFIED` |
| Component Clearance                               | 20 mil (0.508 mm)                                                           | Components / component bodies                                                    | Default placement rule                                          | Footprint courtyard and mechanical constraints may require more clearance                    | `NOT YET VERIFIED` |
| Solder Mask Expansion                             | 2 mil (0.0508 mm)                                                           | Ordinary pads                                                                    | Default mask rule                                               | Footprint-specific mask geometry takes priority where explicitly verified                    | `NOT YET VERIFIED` |
| Minimum Solder Mask Sliver                        | 4 mil (0.1016 mm)                                                           | Board-wide solder-mask webs                                                      | Default manufacturing rule                                      | Fine-pitch feasibility must be resolved locally, not by silently changing this baseline      | `NOT YET VERIFIED` |
| Silk To Solder Mask / Solderable Copper Clearance | 10 mil (0.254 mm)                                                           | Silkscreen relative to pads and other exposed solderable copper                  | Default silkscreen clearance                                    | Keep legends off solderable areas                                                            | `NOT YET VERIFIED` |
| Silkscreen — Ordinary Designators                 | Height 40 mil; Stroke 8 mil                                                 | Ordinary component references                                                    | Default legend style                                            | Maintain readability after fabrication                                                       | `NOT YET VERIFIED` |
| Silkscreen — Important Interface / Polarity       | Height 50–60 mil; Stroke 8–10 mil                                           | User-facing labels such as `USB`, `BAT`, `BAT+`, `BAT-`, `NTC`, `3V3`, and `GND` | Use in preference to ordinary legend style for important labels | Final size within the range depends on available mechanical space                            | `NOT YET VERIFIED` |

GND is not forced into the ordinary routed-power-trace rule. It is primarily implemented through the polygon / ground-plane strategy, with local routing or stitching only where required.

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
| TPS63802 / M4 | Place the VIN capacitor, VOUT capacitors, and L1 close to the IC; keep the switching-current loop short and compact; keep `FB_3V3` away from switching nodes; keep power paths short and wide. The new L1 footprint, pad geometry, and pin mapping require EDA verification. | `REQUIRES EDA VERIFICATION` where applicable |
| BQ25185 / Charger | Place IN / SYS / BAT decoupling close to the device; keep high-current paths short and low-resistance; verify exposed/thermal-pad implementation against the footprint requirements. | `REQUIRES EDA VERIFICATION` |
| USB-C / Protection | Verify connector / ESD / TVS physical ordering and return path during layout. Controlled impedance is not required because USB data is not used. | `NOT YET VERIFIED` |
| Exposed-pad devices | For devices including BQ25185, TPS63802, and TPS259474, paste, thermal-pad, and thermal-via geometry must follow the verified footprint and manufacturer guidance. Global paste-mask rules must not substitute for footprint-specific implementation. | `REQUIRES EDA VERIFICATION` |

## Evidence / Verification Status

| Evidence item | Current status | Meaning |
| --- | --- | --- |
| Documentation baseline freeze | `COMPLETE` | Manufacturing and rule intent is recorded in this authority document |
| Altium PCB rules configured | `NOT YET VERIFIED` | No claim that these rules exist in the `.PcbDoc` |
| Rule Scope / Priority manually checked in Altium | `NOT YET VERIFIED` | User EDA confirmation is still required |
| Critical footprint / pin-pad mapping | `REQUIRES EDA VERIFICATION` where applicable | Includes the new M4 L1 and exposed-pad implementations |
| PCB placement | `NOT YET VERIFIED` | No placement PASS claim |
| Routing | `NOT YET VERIFIED` | No routing PASS claim |
| Polygon / Repour | `NOT YET VERIFIED` | No copper-continuity or repour PASS claim |
| PCB DRC | `NOT YET VERIFIED` | No DRC result was produced by this documentation update |
| Layout Preflight | `NOT YET COMPLETE` | Rule documentation alone does not satisfy the complete preflight checklist |
| Manufacturing release | `NOT AUTHORIZED` | This document cannot approve fabrication |

This update is a documentation baseline freeze only. It is not an Altium configuration PASS, Layout Preflight PASS, PCB DRC PASS, or manufacturing approval.
