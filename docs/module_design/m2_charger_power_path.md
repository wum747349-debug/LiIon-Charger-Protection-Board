# M2 — BQ25185 Charger / Power Path

Status: `COMPLETE FOR STAGE-3 HANDOFF`

## Responsibility and interfaces

U1 is `BQ25185DLHR`. Current design-record pin mapping is pin 1 SYS, pin 2 BAT, pin 3 STAT2, pin 4 CE#, pin 5 GND, pin 6 TS/MR, pin 7 ILIM/VSET, pin 8 ISET, pin 9 STAT1, pin 10 IN, and pin 11 exposed thermal pad / EP; EP is connected to GND. This is the intended Stage 3 connection record, not a claim that the actual `.SchDoc` objects have been fully parsed or verified.

- `VBUS_PROT → pin 10 IN`; `C1 = 10uF / 25V / 0805` from IN to GND. `VBUS_PROT` is supplied by M1.
- `pin 1 SYS → SYS`; `C2 = 2.2uF / 25V / 0603` from SYS to GND. SYS feeds M4 TPS63802.
- `pin 2 BAT → BAT`; `C3 = 1uF` from BAT to GND. BAT also enters M3.
- `pin 6 TS/MR → NTC_TS`; M3 owns the complete temperature network.

## Charge and input-current programming

- `pin 8 ISET → R1 600Ω / 1% → GND`, targeting approximately 500mA fast charge.
- `pin 7 ILIM/VSET → R4 13.0kΩ / 1% → junction → R8 4.99kΩ / 1% → GND`.
- Q2 is `2N7002BK,215`, with drain at the R4/R8 junction, source at GND, gate at `ILIM_HIGH_CMD`, and `R12 = 100kΩ` from gate to GND. Switch OFF presents approximately 18kΩ for the conservative input-current state; switch ON bypasses R8 and presents 13kΩ for the high-current state after M1 qualifies USB source capability.

## CE# fail-safe and status access

`SYS → R3 47kΩ → CE_N / CE#`. Q1 is `2N7002BK,215`, with drain at the R3 / U1 pin 4 CE# common node, source at GND, and gate at `CHG_GATE`. `CHG_ENABLE_CMD → R6 10kΩ → CHG_GATE`; `R7 = 100kΩ` connects CHG_GATE to GND. M3 `TLV7041DBVR` open-drain OUT and M4 TPS63802 open-drain PG also connect to `CHG_GATE`. `CHG_ENABLE_CMD` is the only HIGH authorization source, while temperature-invalid or 3V3-not-power-good can veto LOW. Q1 can pull CE# low only when all three qualifications permit charging. `TP_CE_N` connects to the R3 / Q1 drain / U1 CE# common node, not Q1 gate.

`STAT1` and `STAT2` connect to physical test-point components `TP_STAT1` and `TP_STAT2`. There are no onboard pull-ups or LEDs. A net label alone is not a physical PCB test pad; the physical test-point component and footprint remain subject to later EDA/footprint verification.

## Open integration and evidence boundary

- M1 now defines `ILIM_HIGH_CMD = NOT(CC_OUT1)` and `CHG_ENABLE_CMD = NAND(CC_OUT1, CC_OUT2)`: unattached disables charge/conservatively limits input, Default Current uses the conservative approximately 500mA state, and 1.5A/3A advertisement selects the approximately 1.1A state.
- M3/M4: the shared open-drain veto architecture now implements `CHG_ENABLE_CMD AND temperature-valid AND 3V3-power-good → charging permitted`. Normal startup/brownout qualification is substantially resolved, but TPS63802 PG is datasheet-defined `Undefined` while input is below its own UVLO; the earliest startup transient remains open for Bring-up oscilloscope verification rather than a verified absolute fail-safe claim.
- Cross-Module Integration and schematic-to-BOM consistency review are complete; no new substantive cross-module electrical logic issue was found.
- The latest schematic/BOM are synchronized at the documented baseline. ERC remains `NOT YET VERIFIED`; thermal, PCB/layout and hardware behavior remain downstream evidence.
