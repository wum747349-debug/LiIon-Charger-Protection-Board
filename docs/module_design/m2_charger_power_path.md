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
- Q1 is a small N-MOS with drain at the R2/R3 junction, source at GND, gate at `ILIM_HIGH_CMD`, and a 100kΩ gate pull-down. Q1 OFF presents approximately 18kΩ for the conservative input-current state; Q1 ON bypasses R3 and presents 13kΩ for the high-current state after M1 qualifies USB source capability.
- Q1 exact MPN remains OPEN and is deferred to M1 integration.

## CE# fail-safe and status access

`SYS → R5 47kΩ → CE#`. Q2 is a small N-MOS with drain at CE#, source at GND, gate at `CHG_GATE`, and `R6 = 100kΩ` from gate to GND. `CHG_ENABLE_CMD → R11 ≈4.7kΩ → CHG_GATE`, while M3 TLV7041 open-drain OUT also connects to `CHG_GATE`. Q2 can pull CE# low only when the M1 command permits charging and M3 temperature qualification does not pull the gate low. Q2 exact MPN remains OPEN, deferred to M1/M4 integration.

`STAT1` and `STAT2` connect to physical test-point components `TP_STAT1` and `TP_STAT2`. There are no onboard pull-ups or LEDs. A net label alone is not a physical PCB test pad; the physical test-point component and footprint remain subject to later EDA/footprint verification.

## Open integration and evidence boundary

- M1: `ILIM_HIGH_CMD`, `CHG_ENABLE_CMD`, source-capability qualification, attach/detach, logic levels, startup/default sequencing.
- M4: interaction with the downstream SYS load and startup sequence.
- Exact Q1/Q2 MPNs, thermal analysis, footprint mapping, full BOM and Cross-Module Integration remain OPEN.
- User-reported / screenshot-reviewed capture exists, but no complete schematic PDF/BOM/ERC evidence is present. ERC is NOT RUN / NOT CLAIMED; Stage 4 has not started.
