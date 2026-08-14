# M1 — USB-C Input / CC / Current Capability Control

Status: `SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION`

## Responsibility and interfaces

M1 accepts ordinary 5V USB-C input, presents the Rev.A Sink/UFP CC behavior, protects the CC and VBUS entry points, detects the Source current advertisement, and generates `VBUS_PROT`, `ILIM_HIGH_CMD` and `CHG_ENABLE_CMD`. It interfaces with M2 at `VBUS_PROT`, `ILIM_HIGH_CMD` and `CHG_ENABLE_CMD`; M3 may revoke charging downstream at `CHG_GATE`. USB-PD, USB data and Source/DRP operation are out of scope.

## Connector and protection topology

- Receptacle: `TYPE-C-31-M-12`. All VBUS contacts join `VBUS_PROT`; all GND contacts join GND. D+/D−/SBU are unused / NC. Connector shield / EH grounding remains OPEN.
- CC ESD Primary: `PESD5V0U2BT,215`, current connection record pin 1 → CC1, pin 2 → CC2, pin 3 → GND; it is not connected to VBUS. This replaces `USBLC6-2SC6` as current Primary.
- VBUS uses the current `SMF5.0A` TVS topology on `VBUS_PROT`. `SMF5.0A` is not promoted to an exact manufacturer/orderable procurement fact without matching evidence.

## TUSB320 configuration and VDD_CC

`TUSB320LAIRWBR` CC1/CC2 connect to the receptacle CC lines. `PORT → GND` fixes UFP/Sink mode, `ADDR → NC` selects GPIO mode, `EN_N → GND` enables the device, and `VBUS_PROT → 900kΩ → VBUS_DET`. OUT1/OUT2 are open-drain and each uses an approximately 100kΩ pull-up to `VDD_CC`.

`VBUS_PROT → BAT54,215 → VDD_CC`; the current BAT54 symbol connection is pin 1/anode → `VBUS_PROT`, pin 3/cathode → `VDD_CC`, pin 2 NC. TUSB320 and the logic device each have 100nF local decoupling from `VDD_CC` to GND.

## OUT1/OUT2 and decode

| Attach/current state | CC_OUT1 | CC_OUT2 | ILIM_HIGH_CMD = NOT(CC_OUT1) | CHG_ENABLE_CMD = NAND(CC_OUT1, CC_OUT2) | Policy |
| --- | --- | --- | --- | --- | --- |
| Unattached | H | H | L | L | Charging disabled; conservative ILIM |
| Attached, Default Current | H | L | L | H | Charging enabled; conservative ≈500mA input-current-limit state |
| Attached, 1.5A advertisement | L | H | H | H | Charging enabled; high-current ≈1.1A input-current-limit state |
| Attached, 3A advertisement | L | L | H | H | Charging enabled; high-current ≈1.1A input-current-limit state; Project does not sink 3A |

`SN74LVC2G132DCUR` implements the decode. One NAND gate is used as the inverter for `ILIM_HIGH_CMD`; the other generates `CHG_ENABLE_CMD`.

## Interaction, default and fail-safe behavior

- M2 Q1 uses `ILIM_HIGH_CMD` to select the approximately 18kΩ conservative or 13kΩ high-current ILIM/VSET network. M2 Q2 uses the qualified `CHG_GATE` path to pull BQ25185 CE# low.
- M3 TLV7041 can pull `CHG_GATE` low and inhibit charging independently of M1. Its supply is M4 `3V3`, so `3V3` startup versus `CHG_ENABLE_CMD` remains a Cross-Module Integration sequencing issue.
- Unattached, unpowered, unknown, invalid or unqualified states must not select high current or authorize charging. The resistor defaults in M2 keep Q1/Q2 off when their command/gate drive is absent.

## Remaining open issues

- Final USB-C connector shield / EH grounding strategy.
- Exact manufacturer/orderable MPN and protection qualification for the current `SMF5.0A` TVS topology.
- Actual TUSB320, PESD, connector, BAT54, SN74LVC2G132 and 2N7002BK symbol-pin and footprint mapping verification.
- `3V3` startup / M3 TLV7041 / `CHG_ENABLE_CMD` sequencing during power-up and transitions.
- M4/M5/M6 and the Stage 3 Cross-Module Integration Check.
- Complete schematic PDF, complete BOM and real ERC evidence.

## Evidence boundary

Technical behavior is bounded by the Nexperia and TI manufacturer sources indexed in [references.md](../../references.md). The connection/capture state is user-reported and reflects the current reviewed design baseline; `.SchDoc` remains authoritative. No claim is made of full `.SchDoc` parsing, actual symbol-pin/footprint verification, `ERC PASS`, `SCHEMATIC PASS`, complete schematic/BOM, PCB verification or hardware test.
