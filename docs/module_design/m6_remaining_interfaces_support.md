# M6 — Remaining Interfaces / Support

Status: `SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION`

## Responsibility and external output

M6 records the external output and physical bring-up access that complete the current Stage 3 interface/support baseline.

- `J_OUT1 = KF301-5.0-2P`.
- `J_OUT1 pin 1 → 3V3_PROT`.
- `J_OUT1 pin 2 → GND`.

External loads must use `3V3_PROT`; they must not bypass M5 by using raw `3V3`.

## Physical test-point baseline

| Test point | Electrical node |
| --- | --- |
| `TP_VBUS_PROT` | `VBUS_PROT` |
| `TP_SYS` | `SYS` |
| `TP_BAT` | `BAT` |
| `TP_3V3` | `3V3` |
| `TP_3V3_PROT` | `3V3_PROT` |
| `TP_CHG_GATE` | `CHG_GATE` |
| `TP_CE_N` | BQ25185 U1 pin 4 CE# = R5 / Q3 drain / CE# common node |
| `TP_GND` | `GND` |
| `TP_STAT1` | `STAT1` |
| `TP_STAT2` | `STAT2` |
| `TP_NTC_TS` | `NTC_TS` |
| `TP_M5_RESET` | TPS259474 U3 pin 1 EN/UVLO reset node |

`TP_CE_N` is on the R5 / Q3 drain / U1 CE# common node, not on Q3 gate. `TP_M5_RESET` remains normally isolated from `TP_GND`; it may be temporarily shorted to GND during Bring-up as documented in M5.

No independent `TP_PG_3V3` is present. TPS63802 PG connects directly to `CHG_GATE`, so such a point would duplicate `TP_CHG_GATE` on the same electrical node.

## Shared qualification-node diagnostic limit

`CHG_GATE` is a shared qualification node determined by the `CHG_ENABLE_CMD` authorization path, the TLV7041 open-drain veto and the TPS63802 PG open-drain veto. `TP_CHG_GATE` can show that the shared node is LOW, but cannot by itself identify which source is pulling it LOW.

## Evidence boundary

This record reflects the user-reported current schematic capture baseline. `.SchDoc` remains authoritative. Actual symbol-pin and footprint mapping are not fully verified; complete schematic PDF and BOM are not available; ERC is `NOT RUN / NOT CLAIMED`; PCB accessibility, silkscreen, placement, DRC and hardware behavior are not verified.
