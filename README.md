# 单节锂电池充电与保护电源板

Project Identity: LiIon Charger Protection Board
Current Project Stage: Stage 3 — Schematic Module Design and Capture
Hardware Revision: TBD

## 项目目的

本项目是一块面向 STM32、普通传感器及类似低压实验负载的锂电池实验电源板。第一版以普通 5V USB-C 输入、单节锂离子电池充电、真实 Power Path / Load Sharing、无电池运行、板级功能保护及 3.3V 稳压输出为范围；充电是核心功能，当前没有 firmware 需求。

## 当前状态

- Initialization Status: Initialized
- Battery Requirements Baseline Amendment 已按绑定 Framework 重新执行 Gate 1.5 一致性核对并通过；该结论不证明硬件设计或验证完成。
- 自动 Validator 仅用于检查结构与需求入口完整性。
- Stage 2 — Critical Component Selection 已完成；关键器件及架构方向足以进入模块级详细设计，但这些是 selection baseline，不是 Final BOM。
- 当前 Rev.A primary baseline 包括 BQ25185DLHR、TPS63802DLAR、TPS259474LRPWR、TUSB320LAIRWBR、PESD5V0U2BT,215 和 TYPE-C-31-M-12；M2 Q1/Q3 exact MPN baseline 为 2N7002BK,215，M4 L1 exact baseline 为 Coilcraft XFL4015-471MEC / 0.47uH / LCSC C18221164。NTC 与 `SMF6.0A` TVS exact manufacturer/orderable MPN 仍 OPEN；battery connector 实物兼容性/极性仍为 Physical Verification Pending。该 selection baseline 不是 Final BOM。
- Stage 3 — Schematic Module Design and Capture 仍在进行。M1–M6 module design/capture baseline substantially complete；用户已报告在 Altium Designer 中完成当前 M1–M6 schematic draft，Cross-Module Integration 仍 pending。
- M1 当前 USB shield baseline 为 EH1–EH4 并入 `USB_SHIELD`，再由 R23 1MΩ 与 C15 1nF/1kV C0G 两个并联支路接 GND；R24 0Ω 为 default-DNP tuning/direct-ground provision。D2 当前为 `SMF6.0A`，保护拓扑保持 `VBUS_PROT → TVS → GND`；这些不表示 EMC/ESD verified。
- M4 保持 TPS63802DLAR Primary，L1 exact baseline 为 `XFL4015-471MEC / 0.47uH / C18221164`，连接为 `TPS63802 L1 pin ↔ L1 ↔ L2 pin`。该选择为 datasheet-qualified design selection，不是 thermal、transient、PCB-layout 或 hardware verification。shared open-drain `CHG_GATE` qualification 已形成当前 design baseline，但 TPS63802 低于自身 UVLO 时 PG 的 datasheet-defined `Undefined` 边界仍要求 Bring-up 示波器验证 earliest startup transient。
- M5 保持 TPS259474LRPWR Primary，当前边界为 `M4 3V3 → TPS259474LRPWR → 3V3_PROT → M6 / external load`。PGTH divider 已更新为 R20 75kΩ / 1% 与 R21 49.9kΩ / 1%，nominal threshold 仍约 3.0V，较高 divider current 用于降低 PGTH leakage-related error；threshold 未实测。
- M6 当前记录 `J_OUT1 pin 1 → 3V3_PROT`、`pin 2 → GND` 及关键 physical test points。外部负载不得绕过 M5 使用 raw `3V3`；`CHG_GATE` 是多 source shared qualification node，`TP_CHG_GATE` 不能单独区分哪个 source 正在拉低。当前没有完整 schematic PDF / BOM 或 ERC evidence，未声称实际 `.SchDoc` pin mapping、footprint mapping、PCB/DRC 或硬件已验证，Stage 3 尚未完成。
- 本 Standalone Repository 是 Only Active Project Authority；Legacy Project 2 仅作为 Frozen Migration Source 保留，用于历史迁移追溯。

## 项目事实

- [需求基线](requirements.md)
- [系统框图](block_diagram.md)
- [设计说明](design_notes.md)
- [资料索引](references.md)
- [Stage 2 关键器件选型计划](docs/component_selection_plan.md)
- [Stage 3 模块设计入口](docs/module_design/README.md)

## 仓库导航

- [Framework 绑定与初始化状态](FRAMEWORK.md)
- [项目运行规则](PROJECT_RULES.md)
- [阶段文档职责](docs/README.md)
- [硬件源文件与证据职责](hardware/README.md)
- [本地资料职责](references/README.md)
- [项目验证器](scripts/validate_project_repository.py)

## 迁移来源

Source Repository: wum747349-debug/Hardware-Practice-Projects

Legacy Project Path: projects/02_LiIon_Charger_Protection_Board

Migration Date: 2026-08-04

Initial Migration Framework Snapshot: development-v0.9 @ 4261bdfa3cf6be400be16722c27fca22ea1895ad

Current Framework binding: see `FRAMEWORK.md`.

Git History Strategy: Clean Import

Legacy Repository: wum747349-debug/Hardware-Practice-Projects

Authority Status: Standalone Project Repository — Only Active Project Authority

Legacy Project Status: Frozen Migration Source — Historical migration traceability only

## 下一步

当前处于 Stage 3。M1–M6 module design/capture baseline substantially complete；下一步是完成 Stage 3 Cross-Module Integration，并在 Bring-up 中验证 earliest startup transient 以及 M5 的 UV/OV、PGTH、过流、短路、热和 latch/reset behavior。建议同步观察 `VBUS / SYS / 3V3 / 3V3_PROT / PG / CHG_ENABLE_CMD / CHG_GATE / CE#`。完整 schematic PDF / BOM、ERC、PCB 与硬件验证均尚未完成。

Repository Authority Cutover 已完成。今后的 Project facts、hardware work、evidence 和 Stage progression 只能在本 Standalone Repository 中进行；Legacy Project 仅保留为 Frozen Migration Source，不再继续活动开发或双写。
