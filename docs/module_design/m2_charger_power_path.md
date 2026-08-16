# M2 — BQ25185 Charger / Power Path

Status: `COMPLETE FOR STAGE-3 MODULE INTEGRATION`

## Responsibility and interfaces

U1 is `BQ25185DLHR`. Current design-record pin mapping is pin 1 SYS, pin 2 BAT, pin 3 STAT2, pin 4 CE#, pin 5 GND, pin 6 TS/MR, pin 7 ILIM/VSET, pin 8 ISET, pin 9 STAT1, pin 10 IN, and pin 11 exposed thermal pad / EP; EP is connected to GND. This is the intended Stage 3 connection record, not a claim that the actual `.SchDoc` objects have been fully parsed or verified.

- `VBUS_PROT → pin 10 IN`; `C1 = 2.2uF / 25V` from IN to GND. `VBUS_PROT` is supplied by future M1.
- `pin 1 SYS → SYS`; `C2 = 10uF / 25V` from SYS to GND. SYS later feeds M4 TPS63802.
- `pin 2 BAT → BAT`; `C4 = 1uF` from BAT to GND. BAT also enters M3.
- `pin 6 TS/MR → NTC_TS`; M3 owns the complete temperature network.

## Charge and input-current programming

- `pin 8 ISET → R1 600Ω / 1% → GND`, targeting approximately 500mA fast charge.
- `pin 7 ILIM/VSET → R2 13.0kΩ / 1% → junction → R3 4.99kΩ / 1% → GND`.
- Q1 is `2N7002BK,215`, with drain at the R2/R3 junction, source at GND, gate at `ILIM_HIGH_CMD`, and a 100kΩ gate pull-down. Q1 OFF presents approximately 18kΩ for the conservative input-current state; Q1 ON bypasses R3 and presents 13kΩ for the high-current state after M1 qualifies USB source capability. Nexperia's device pin identity is pin 1 gate, pin 2 source, pin 3 drain; actual EDA symbol-pin/footprint mapping remains to be verified.

## CE# fail-safe and status access

`SYS → R5 47kΩ → CE#`. Q3 is `2N7002BK,215`, with drain at the R5 / U1 pin 4 CE# common node, source at GND, gate at `CHG_GATE`, and `R6 = 100kΩ` from gate to GND. `CHG_ENABLE_CMD → R11 10kΩ → CHG_GATE`; M3 `TLV7041DBVR` open-drain OUT and M4 TPS63802 open-drain PG also connect to `CHG_GATE`. `CHG_ENABLE_CMD` is the only HIGH authorization source, while temperature-invalid or 3V3-not-power-good can veto LOW. Q3 can pull CE# low only when all three qualifications permit charging. `TP_CE_N` connects to the R5 / Q3 drain / U1 CE# common node, not Q3 gate. Actual EDA symbol-pin/footprint mapping remains to be verified.

`STAT1` and `STAT2` connect to physical test-point components `TP_STAT1` and `TP_STAT2`. There are no onboard pull-ups or LEDs. A net label alone is not a physical PCB test pad; the physical test-point component and footprint remain subject to later EDA/footprint verification.

## Open integration and evidence boundary

- M1 now defines `ILIM_HIGH_CMD = NOT(CC_OUT1)` and `CHG_ENABLE_CMD = NAND(CC_OUT1, CC_OUT2)`: unattached disables charge/conservatively limits input, Default Current uses the conservative approximately 500mA state, and 1.5A/3A advertisement selects the approximately 1.1A state.
- M3/M4: the shared open-drain veto architecture now implements `CHG_ENABLE_CMD AND temperature-valid AND 3V3-power-good → charging permitted`. Normal startup/brownout qualification is substantially resolved, but TPS63802 PG is datasheet-defined `Undefined` while input is below its own UVLO; the earliest startup transient remains open for Bring-up oscilloscope verification rather than a verified absolute fail-safe claim.
- Q1/Q3 exact MPNs are now `2N7002BK,215`; thermal analysis, actual symbol-pin/footprint mapping, full BOM and Cross-Module Integration remain OPEN.
- User-reported / screenshot-reviewed capture exists, but no complete schematic PDF/BOM/ERC evidence is present. ERC is NOT RUN / NOT CLAIMED; Stage 4 has not started.
