# M1 — USB-C Input / CC / Current Capability Control

Status: `COMPLETE FOR STAGE-3 HANDOFF`

## Responsibility and interfaces

M1 accepts ordinary 5V USB-C input, presents the Rev.A Sink/UFP CC behavior, protects the CC and VBUS entry points, detects the Source current advertisement, and generates `VBUS_PROT`, `ILIM_HIGH_CMD` and `CHG_ENABLE_CMD`. It interfaces with M2 at `VBUS_PROT`, `ILIM_HIGH_CMD` and `CHG_ENABLE_CMD`; M3 may revoke charging downstream at `CHG_GATE`. USB-PD, USB data and Source/DRP operation are out of scope.

## Connector and protection topology

- Receptacle: `TYPE-C-31-M-12`. All VBUS contacts join `VBUS_PROT`; all GND contacts join GND. D+/D−/SBU are unused / NC. EH1/EH2/EH3/EH4 join `USB_SHIELD`.
- Current shield baseline uses parallel branches: `USB_SHIELD → R19 1MΩ → GND` and `USB_SHIELD → C13 1nF / high-voltage / 1206 → GND`. `R23 = 100kΩ` and is not the shield resistor. Rev.A does not include an R24 0-ohm DNP branch. This EMI/ESD-oriented engineering baseline is not `EMC PASS`, `ESD PASS` or experimentally verified shield behavior.
- CC ESD Primary: `PESD5V0U2BT,215`, current connection record pin 1 → CC1, pin 2 → CC2, pin 3 → GND; it is not connected to VBUS. This replaces `USBLC6-2SC6` as current Primary.
- `D2 = Littelfuse SMF6.0A` / LCSC `C207260` connects `VBUS_PROT → TVS → GND`. Exact sourcing is synchronized to the current BOM; placement, return path and hardware protection performance remain downstream verification.

## TUSB320 configuration and VDD_CC

`TUSB320LAIRWBR` CC1/CC2 connect to the receptacle CC lines. `PORT → GND` fixes UFP/Sink mode, `ADDR → NC` selects GPIO mode, `EN_N → GND` enables the device, and `VBUS_PROT → R18 910kΩ / 1% → VBUS_DET`. OUT1/OUT2 are open-drain and each uses an approximately 100kΩ pull-up to `VDD_CC`.

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

- M2 uses `ILIM_HIGH_CMD` to select the approximately 18kΩ conservative or 13kΩ high-current ILIM/VSET network. The final charge-enable MOSFET Q1 uses the qualified `CHG_GATE` path to pull `CE_N / BQ25185 CE#` low.
- M3 `TLV7041DBVR` and M4 TPS63802 PG can independently pull the shared `CHG_GATE` low. With `CHG_ENABLE_CMD` as the only HIGH authorization source, the normal startup/brownout/temperature qualification architecture is substantially resolved; only the earliest startup transient remains open because TPS63802 PG is datasheet-defined `Undefined` below its own UVLO.
- Unattached, unpowered, unknown, invalid or unqualified states must not select high current or authorize charging. The resistor defaults in M2 keep Q1/Q3 off when their command/gate drive is absent.

## Remaining open issues

- USB shield RC placement, return path and experimental EMI/ESD validation.
- Placement, return-path and hardware protection qualification for the selected Littelfuse `SMF6.0A` TVS.
- Actual TUSB320, PESD, connector, BAT54, SN74LVC2G132 and 2N7002BK symbol-pin and footprint mapping verification.
- Bring-up oscilloscope verification of the earliest startup transient across `VBUS / SYS / 3V3 / PG / CHG_ENABLE_CMD / CHG_GATE / CE#`; no absolute fail-safe or measured PASS claim is made.
- ERC report remains unavailable; `ERC = NOT YET VERIFIED`.

## Evidence boundary

Technical behavior is bounded by the Nexperia and TI manufacturer sources indexed in [references.md](../../references.md). The latest schematic and BOM were included in the completed review and consistency review; `.SchDoc` remains authoritative. No claim is made of automated `.SchDoc` parsing, `ERC PASS`, PCB/layout verification or hardware test.
