# M4 — 3.3V Buck-Boost

Status: `SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION`

## Responsibility and interfaces

M4 converts `SYS` to the regulated `3V3` rail with the Stage 2 Primary `TPS63802DLAR`. It accepts power from M2 at `SYS`, supplies the downstream 3.3V domain, and contributes the regulator power-good qualification to the shared `CHG_GATE` charging-permission node. The current connection record is user-reported / screenshot-reviewed schematic capture, not verified parsing of the authoritative `.SchDoc`.

## TPS63802 connection baseline

| Pin | Name | Current connection |
| --- | --- | --- |
| 1 | EN | `SYS` |
| 2 | MODE | `GND` |
| 3 | AGND | `GND` |
| 4 | FB | `FB_3V3` |
| 5 | PG | `CHG_GATE` |
| 6 | VOUT | `3V3` |
| 7 | L2 | one side of `L1 XFL4015-471MEC`, `0.47uH`; the other side connects to pin 9 L1 |
| 8 | GND | `GND` |
| 9 | L1 | one side of `L1 XFL4015-471MEC`, `0.47uH`; the other side connects to pin 7 L2 |
| 10 | VIN | `SYS` |

The current exact inductor baseline is `L1 = Coilcraft XFL4015-471MEC`, `0.47uH`, LCSC `C18221164`, connected `TPS63802 pin 9 L1 ↔ L1 0.47uH ↔ TPS63802 pin 7 L2`. Coilcraft official data identifies the selected part as 0.47uH, and the LCSC evidence maps the orderable MPN to `C18221164`.

The input capacitor is `10uF` from `SYS` to GND. The output network is `2 x 22uF` from `3V3` to GND. The feedback divider is `3V3 → 511kΩ / 1% → FB_3V3 → 91kΩ / 1% → GND`.

These values are the current Stage 3 schematic baseline. They do not constitute measured ripple, transient, current-stress, thermal or stability evidence.

## PG and CHG_GATE integration

The final simple charging qualification is:

`CHG_ENABLE_CMD → R11 10kΩ → CHG_GATE`

The same node also has `R6 100kΩ: CHG_GATE → GND`, `TLV7041DBVR OUT → CHG_GATE`, `TPS63802 PG → CHG_GATE`, and `CHG_GATE → charge-enable MOSFET gate`. `CHG_ENABLE_CMD` is the only HIGH authorization source. The ordinary `TLV7041DBVR` OUT and TPS63802 PG are open-drain LOW veto sources; neither authorizes charging HIGH. TPS63802 PG has no separate `3V3` pull-up, and this design does not add `TPS3808G33DBVR` / U6.

The intended qualification is:

`CHG_ENABLE_CMD AND temperature-valid AND 3V3-power-good → charging permitted`

Normal 3V3 startup, brownout and temperature-veto architecture is therefore substantially resolved. This is not an absolute fail-safe startup claim: while TPS63802 input is still below its own UVLO during the earliest startup interval, the datasheet defines a boundary in which PG is `Undefined`. Earliest startup transient behavior remains open for Bring-up oscilloscope verification. Recommended simultaneous observations are `VBUS / SYS / 3V3 / PG / CHG_ENABLE_CMD / CHG_GATE / CE#`.

## Layout-sensitive notes

- Keep the `0.47uH` inductor and the VIN/VOUT ceramic capacitors close to the TPS63802 power pins, with compact high-di/dt loops.
- Keep the `FB_3V3` node and 511kΩ/91kΩ divider away from the switch nodes and inductor; use a quiet ground reference for the lower divider resistor.
- Provide a low-impedance ground implementation that respects the separation and local return needs of AGND and power GND before their intended common reference.
- Route PG/`CHG_GATE` as a control node away from the L1/L2 switch nodes, and preserve access to the startup-observation nets for Bring-up.
- Final placement, copper, thermal behavior, footprint/pad mapping and manufacturer layout guidance compliance remain subject to later EDA and PCB review.

## Open evidence boundary

- Schematic capture basis: user-reported / screenshot-reviewed.
- Actual `.SchDoc` pin mapping: `NOT VERIFIED`.
- ERC: `NOT RUN / NOT CLAIMED`.
- Complete schematic PDF: `NOT AVAILABLE`.
- Complete BOM: `NOT AVAILABLE`.
- Hardware startup sequencing: `NOT MEASURED`.
- Inductor selection: `DATASHEET-QUALIFIED DESIGN SELECTION`; thermal, transient and PCB-layout behavior are `NOT VERIFIED`.
- 500mA continuous and 1A/100ms transient hardware verification: `NOT RUN`.

M4 is ready for ongoing Stage 3 module integration at the documented design-record level, but it is not `ERC PASS`, `SCHEMATIC PASS`, hardware verified, or ready to close Stage 3.
