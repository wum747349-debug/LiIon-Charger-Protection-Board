# PCB Design Rules / Rev.A Final Engineering Baseline

Status: `STAGE 6 ROUTING / COPPER COMPLETE — STAGE 7 RELEASE REVIEW ACTIVE`

This file owns the Rev.A PCB manufacturing and Altium rule baseline. `.PcbDoc` remains the EDA implementation authority. User-confirmed Altium configuration, screenshots, and DRC evidence are recorded as implementation evidence; manufacturing release still requires Stage 7 output review.

## Manufacturing Baseline

| Item | Rev.A baseline | Note |
| --- | --- | --- |
| Manufacturer | JLCPCB / 嘉立创 | Target manufacturer |
| PCB type | Standard rigid PCB | No flex / HDI |
| Material | FR-4 | Standard construction |
| Layer count | 2 Layer | Rev.A baseline |
| Finished thickness | 1.6 mm | Finished board |
| Copper weight | 1 oz / 35 µm | Both copper layers |
| Solder mask | Green | Both sides as applicable |
| Silkscreen | White | Top-side user / assembly markings |
| Surface finish | Lead-Free HASL / 无铅喷锡 | Rev.A baseline |
| Controlled impedance | Not required | USB data not used |
| Assembly | Manual assembly | Small exposed-pad power ICs use solder paste plus heating-plate / reflow-style processing |
| PCBA service | Not required | Bare PCB manufacturing only |

Manufacturer capability values are not copied directly into Project default rules. Project values retain deliberate margin where practical.

## Mechanical Baseline

| Item | Rev.A baseline | Verification status |
| --- | --- | --- |
| Nominal board size | 55 mm × 35 mm | Frozen; final output viewer still required |
| Mounting-hole centers | `(4,4)`, `(51,4)`, `(4,31)`, `(51,31)` mm | Frozen nominal pattern |
| Mounting holes | M3 mechanical NPTH, actual hole ≈ 3.20 mm / 125.984 mil, `Plated = OFF` | Confirmed by user in Altium; final Drill / NPTH viewer check required |
| USBC1 | Left board edge; insertion/removal outward | Placement complete |
| J_NTC1 / J_BAT1 | Top-side interface region; cable access outward | Placement complete |
| J_OUT1 | Right edge; wire-entry outward | Placement complete |

The four M3 NPTH holes intentionally exceed the current global `Hole Size Max = 100 mil`. User explicitly elected not to add a separate Hole Size Rule exception. The resulting four Hole Size DRC hits are therefore known Release findings and do not imply that the M3 holes should be reduced. Final Drill output must still show these holes as NPTH.

## General Altium Rules

Primary units are mil.

| Rule | Final Rev.A value | Scope | Engineering interpretation | Current evidence |
| --- | --- | --- | --- | --- |
| Electrical Clearance | Minimum 6 mil | Default board-wide | Project design default | Latest DRC: 0 violations |
| Width_Default | Min 6 / Preferred 8 / Max 40 mil | `All` | Ordinary signals | Latest DRC: 0 violations |
| Width_Power | **Min 8 / Preferred 20 / Max 100 mil** | `InNetClass('POWER')` | 8 mil only for short small-pad escape / neck-down; ordinary power target is 20 mil or wider copper / polygon | Latest DRC: 0 violations |
| Routing Via Style — Standard | Diameter 24 mil / Hole 12 mil | Ordinary through vias | Does not define special exposed-pad thermal via geometry | User implementation evidence |
| Board Outline Clearance | Copper to board edge 12 mil | Board-wide | Preserve design margin | Stage 7 viewer check pending |
| Component Clearance | 20 mil | Default component bodies | Placement design default | Placement review complete |
| Solder Mask Expansion | 2 mil | Ordinary pads | Footprint-specific geometry may override | Stage 7 viewer check pending |
| Minimum Solder Mask Sliver | 4 mil | Board-wide | Manufacturing rule | Latest DRC: 0 violations |
| Silk To Silk Clearance | 6 mil | Silkscreen-to-silkscreen | Compact-board project rule | Latest DRC: 9 findings retained |
| Silk To Solder Mask / solderable copper | 6 mil | Top Overlay relative to exposed solderable features | Compact-board project rule | Latest DRC: 200 findings retained |
| Hole Size | Min 1 / Max 100 mil | Current global rule | Four intentional M3 NPTH holes exceed this global max | Latest DRC: 4 known findings |
| Hole-To-Hole | 10 mil | Board-wide | Drill separation | Latest DRC: 0 violations |

### POWER Net Class

`POWER` is the sole power Net Class and contains:

```text
VBUS_PROT
SYS
BAT
3V3
3V3_PROT
GND
```

`GND` remains in the class for simple rule management but is implemented primarily by polygon / ground-plane strategy rather than ordinary routed traces.

## Polygon / Copper Baseline

Top and Bottom GND Polygon final Rev.A settings:

```text
Pour style                  = Solid Pour
Remove Islands Less Than    = 2500 sq.mil
Remove Necks Less Than      = 10 mil
Remove Dead Copper          = enabled
Pour Over Same Net Objects  = enabled
```

Do not increase `Remove Necks Less Than` mechanically beyond 10 mil without a specific copper-continuity reason. After any Polygon, via, or routing change, Repour before judging the result.

Layer strategy:

| Layer | Final Rev.A strategy | Stage 6 result |
| --- | --- | --- |
| Top | Components + primary signal / critical power routing + local GND copper | Complete |
| Bottom | Continuous GND plane as practical + only limited unavoidable crossover routing | Visual review PASS; no identified plane-splitting blocker |

GND via strategy is functional / return-path driven rather than an arbitrary grid. Key local GND connections and a limited number of stitching vias have been implemented. Stage 6 review found no need for additional blanket stitching.

## Polygon Connect Baseline

One board-wide `PolygonConnect` intent is retained:

- SMD pads: Direct Connect where implemented / applicable.
- Vias: Direct Connect.
- Through-hole pads: Relief Connect.
- Narrow exceptions should exist only for verified component- or manufacturing-specific reasons.

Actual manufacturing appearance must be checked in Gerber Viewer.

## Project-Specific Layout Inputs / Final Disposition

| Area | Rev.A requirement | Final Stage 6 disposition |
| --- | --- | --- |
| TPS63802 / M4 | Compact input/output current loops; L1/L2 switching region controlled; FB kept away from switching nodes; power paths short/wide | No blocker identified in final routing/copper review |
| BQ25185 / M2 | IN / SYS / BAT decoupling close; power paths short; GND / exposed-pad return effective | No blocker identified in final routing/copper review |
| USB-C / Protection / M1 | Protection close to interface; short protection return; USB_SHIELD network not bypassed by accidental direct GND stitching | No blocker identified in final review |
| TPS259474 / M5 | Direct 3V3 → eFuse → 3V3_PROT power path; avoid unnecessary ground disruption | No blocker identified in final review |
| GND / stitching | Preserve Bottom GND continuity; use via placement where it shortens functional / return path or ties useful local Top GND copper to Bottom | Complete for Stage 6 |

## Silkscreen Release Disposition

Latest DRC contains 200 `Silk To Solder Mask` and 9 `Silk To Silk` findings. User explicitly elected not to perform broad silkscreen cleanup.

These findings are not automatically treated as an unconditional manufacturing PASS. Stage 7 must inspect the actual Gerber-rendered result for at least:

- `J_BAT1`, battery polarity / GND indication.
- `J_NTC1`, NTC / GND indication.
- `J_OUT1`, 3V3 / GND indication.
- Critical Pin 1 / polarity markers.
- Key power / GND and test-point labels.

Ordinary footprint outlines / designators may be accepted as a silkscreen-only waiver if Gerber inspection shows no assembly, polarity, connector, service, or user-error risk.

## Latest DRC Evidence

User-provided Altium Design Rule Verification Report:

```text
Date / time: 2026-08-25 21:31:22
Warnings: 0
Rule Violations: 213

Clearance                    0
Short Circuit                0
Un-Routed Nets               0
Modified Polygon             0
Width — POWER                0
Width — Default              0
Power Plane Connect          0
Hole Size                    4
Hole-To-Hole                 0
Minimum Solder Mask Sliver   0
Silk To Solder Mask          200
Silk To Silk                 9
Net Antennae                 0
Height                       0
```

The four Hole Size findings correspond to the intentional ≈3.20 mm M3 NPTH mounting holes. Remaining 209 findings are silkscreen-related.

## Evidence / Verification Status

| Evidence item | Current status |
| --- | --- |
| Documentation final rule baseline | `SYNCED TO CURRENT LOCAL ALTIUM / DRC EVIDENCE` |
| Placement | `COMPLETE / FROZEN` |
| Routing | `COMPLETE — B3 ACCEPTED` |
| GND Polygon / Repour | `COMPLETE — USER EVIDENCE REVIEWED` |
| Bottom GND continuity | `PASS BY VISUAL REVIEW; EXACT GEOMETRY REMAINS EDA AUTHORITY` |
| PCB Batch DRC | `RUN — ELECTRICAL / ROUTING CATEGORIES CLEAN; 4 HOLE-RULE + 209 SILK FINDINGS RETAINED` |
| Stage 6 B3 | `PASS — APPROVED TO ENTER STAGE 7 PCB RELEASE REVIEW` |
| Gerber / Drill output review | `PENDING` |
| Manufacturing release | `NOT YET AUTHORIZED` |
