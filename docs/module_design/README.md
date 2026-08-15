# Stage 3 — Module Design Records

Status: Stage 3 active — M1/M2/M3/M4/M5 module records synchronized; cross-module integration pending

## Current progress

| Module | Current status | Record |
| --- | --- | --- |
| M1 — USB-C Input / CC / Current Capability Control | SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m1_usb_c_input_cc_current_control.md](m1_usb_c_input_cc_current_control.md) |
| M2 — BQ25185 Charger / Power Path | COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m2_charger_power_path.md](m2_charger_power_path.md) |
| M3 — Battery + External NTC | SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m3_battery_external_ntc.md](m3_battery_external_ntc.md) |
| M4 — 3.3V Buck-Boost | SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m4_3v3_buck_boost.md](m4_3v3_buck_boost.md) |
| M5 — 3.3V Output Protection | SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION | [m5_3v3_output_protection.md](m5_3v3_output_protection.md) |
| M6 — Remaining interfaces / support | NOT STARTED | Pending |

用户已在 Altium Designer 中手工 capture 当前 M1/M2/M3/M4/M5 schematic draft，并已报告完成 M1 主要审查接线修正。M4 的 `TPS63802DLAR` power stage、shared open-drain `CHG_GATE` qualification，以及 M5 的 `TPS259474LRPWR` output-protection network 已形成当前 Stage 3 integration baseline；后续仍需完成 M6 和 Stage 3 Cross-Module Integration Check。

## Evidence boundary

`.SchDoc` 仍是 authoritative implementation source。当前进度属于 user-reported / screenshot-reviewed Stage 3 schematic capture progress；仓库中没有与当前版本对应的完整 schematic PDF、完整 BOM 或 ERC evidence，因此不声称实际 `.SchDoc` pin mapping fully verified、schematic complete、ERC PASS、footprint verified、PCB/hardware verified 或 temperature trip experimentally verified。Normal 3V3 startup / brownout / temperature veto architecture 已基本形成，但 TPS63802 低于自身 UVLO 时 PG 的 datasheet-defined `Undefined` 边界使 earliest startup transient 仍须 Bring-up 示波器验证；M5 UV/OV threshold、1A/100ms transient、hard short、thermal 与 latch/reset behavior 也均未实测。Stage 3 remains active and is not complete；Stage 4 has not started。
