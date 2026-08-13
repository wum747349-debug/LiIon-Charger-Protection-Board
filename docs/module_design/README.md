# Stage 3 — Module Design Records

Status: Stage 3 active — M2/M3 module records synchronized; cross-module integration pending

## Current progress

| Module | Current status | Record |
| --- | --- | --- |
| M1 — USB-C Input / CC / Current Capability Control | NOT STARTED; next hardware design task | Pending |
| M2 — BQ25185 Charger / Power Path | COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m2_charger_power_path.md](m2_charger_power_path.md) |
| M3 — Battery + External NTC | SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m3_battery_external_ntc.md](m3_battery_external_ntc.md) |
| M4 — 3.3V Buck-Boost | NOT STARTED | Pending |
| M5 — 3.3V Output Protection | NOT STARTED | Pending |
| M6 — Remaining interfaces / support | NOT STARTED | Pending |

用户已在 Altium Designer 中手工 capture 当前 M2/M3 schematic draft。后续顺序为 M1，再完成 M4/M5/M6，最后执行 Stage 3 Cross-Module Integration Check；本记录不启动这些未完成模块。

## Evidence boundary

`.SchDoc` 仍是 authoritative implementation source。当前进度属于 user-reported / screenshot-reviewed Stage 3 schematic capture progress；仓库中没有与当前版本对应的完整 schematic PDF、完整 BOM 或 ERC evidence，因此不声称实际 `.SchDoc` pin mapping fully verified、schematic complete、ERC PASS、footprint verified、PCB/hardware verified 或 temperature trip experimentally verified。Stage 3 remains active and is not complete；Stage 4 has not started。
