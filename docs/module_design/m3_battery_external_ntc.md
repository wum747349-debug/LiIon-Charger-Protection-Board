# M3 — Battery + External NTC

Status: `SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION`

## Battery interface

`J_BAT1` schematic candidate is `HX-XH2.54-2PZZ`: pin 1 connects to BAT and pin 2 to GND; BAT also connects to the physical test-point component `TP_BAT`. Battery and NTC connectors may use the same candidate type only with clear silkscreen and physical-placement differentiation. Actual mating compatibility, polarity, wire order, wire gauge and current rating remain `Physical Verification Pending`; red/black wires or product images do not verify polarity.

## External NTC and NTC_TS network

Rev.A uses an external wired/epoxy NTC suitable for a representative pouch surface: `10kΩ @25℃`, `B ≈3435K`, `±1% preferred`. Exact orderable MPN remains `OPEN / Stage 3 procurement selection`. It must be electrically insulated and attached without puncturing, compressing or otherwise damaging the pouch; final placement and attachment remain mechanical inputs.

- `U1 pin 6 TS/MR → NTC_TS → TP_NTC_TS`.
- `NTC_TS → R8 2.7kΩ / 1% → J_NTC1 pin 1`; `J_NTC1 pin 2 → GND`.
- External `TH1 10k/B3435` connects between J_NTC1 pins 1 and 2.

R8 is primarily mitigation against an external NTC short pulling TS/MR directly near 0V, reducing TS/MR push-button / long-low / factory-mode interaction risk; it is not presented as a precision calibration resistor.

## High-temperature comparator interlock

U5 is the ordinary `TLV7041DBVR`, whose current design-record pin mapping is pin 1 OUT, pin 2 V-, pin 3 IN+, pin 4 IN-, pin 5 V+. It must not be confused with differently pinned TLV7041S / TLV7041L variants.

- Supply: `pin 5 V+ → 3V3`, `pin 2 V- → GND`; local `C3 = 100nF` from 3V3 to GND.
- Positive input: `pin 3 IN+ → NTC_TS`.
- Reference: `3V3 → R9 90.9kΩ / 1% → VHOT_REF → R10 10kΩ / 1% → GND`; `VHOT_REF → pin 4 IN-`. Nominal reference is approximately 0.327V.
- Open-drain output: `pin 1 OUT → CHG_GATE`; `CHG_ENABLE_CMD → R11 10kΩ → CHG_GATE`; `CHG_GATE → R6 100kΩ → GND`, TPS63802 open-drain PG, and Q2 gate. `CHG_ENABLE_CMD` is the only HIGH authorization source; TLV7041 OUT and TPS63802 PG are independent LOW veto sources. The required current design remains the ordinary open-drain `TLV7041DBVR`, not the push-pull `TLV7031DBVR` discussed during intermediate capture.

At normal temperature, `NTC_TS > VHOT_REF`, U5 OUT is high impedance, and a HIGH `CHG_ENABLE_CMD` may turn Q2 on and pull CE# low. For over-temperature or another relevant low-NTC_TS fault, U5 pulls `CHG_GATE` low, Q2 turns off, and R5 pulls CE# high to inhibit charging.

## Temperature interpretation and fault behavior

The Battery Design Basis remains the product-spec charging range of approximately 0℃–45℃. The present Stage 3 circuit is an engineering implementation with a conservative nominal hardware charge-temperature window of approximately 4℃–39℃, not a new manufacturer specification or a verified exact trip range. Resistor, comparator-threshold and NTC tolerances, pouch thermal lag and hardware testing remain unresolved; actual trip temperatures are `Planned for Bring-up / Test`.

- Too cold: BQ25185 native TS COLD behavior inhibits charging.
- Too hot: TLV7041 pulls `CHG_GATE` low, making Q2 OFF and CE# HIGH.
- NTC short: R8 avoids a direct hard short of TS/MR to GND; TLV7041 qualification is intended to revoke charge enable.
- NTC open/disconnected: BQ25185 native TS behavior owns abnormal handling.
- Temperature faults revoke charging qualification only; they do not disable the valid USB-to-SYS Power Path.

## Open integration and evidence boundary

M1 defines `CHG_ENABLE_CMD = NAND(CC_OUT1, CC_OUT2)`, so unattached state revokes charge enable and every qualified attached current-advertisement state may authorize the shared node. M3 TLV7041 and M4 TPS63802 PG then independently veto LOW, implementing `CHG_ENABLE_CMD AND temperature-valid AND 3V3-power-good → charging permitted`. The normal startup/brownout/temperature architecture is substantially resolved, but the earliest startup transient remains open because TPS63802 PG is datasheet-defined `Undefined` below its own UVLO; it requires Bring-up oscilloscope verification and is not an absolute fail-safe or measured PASS claim. Exact NTC MPN and attachment, connector physical verification, actual trip-temperature tests, footprint mapping and Cross-Module Integration remain OPEN. Current capture is user-reported / screenshot-reviewed; no claim is made of full `.SchDoc` pin verification, ERC PASS, complete BOM, footprint/PCB/hardware verification or experimental temperature verification.
