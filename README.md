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
- 当前 Rev.A primary baseline 包括 BQ25185DLHR、TPS63802DLAR、TPS259474LRPWR、TUSB320LAIRWBR、PESD5V0U2BT,215 和 TYPE-C-31-M-12；M2 Q1/Q2 exact MPN baseline 为 2N7002BK,215。NTC、`SMF5.0A` TVS exact manufacturer/orderable MPN、battery connector 与普通无源器件的准确选型/实物边界仍按计划 OPEN / Deferred。该 selection baseline 不是 Final BOM。
- Stage 3 — Schematic Module Design and Capture 仍在进行。M1、M3、M4 与 M5 已达到 `SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION`，M2 已达到 `COMPLETE FOR STAGE-3 MODULE INTEGRATION`；用户已在 Altium Designer 中手工 capture 当前 M1/M2/M3/M4/M5 schematic draft。
- M4 保持 TPS63802DLAR Primary，其 power stage 与 `CHG_ENABLE_CMD AND temperature-valid AND 3V3-power-good` shared open-drain `CHG_GATE` qualification 已形成当前 design baseline。Normal 3V3 startup / brownout / temperature veto architecture 已基本解决，但 TPS63802 低于自身 UVLO 时 PG 的 datasheet-defined `Undefined` 边界仍要求 Bring-up 示波器验证 earliest startup transient，不能声称 absolute fail-safe 或 measured PASS。
- M5 保持 TPS259474LRPWR Primary，当前边界为 `M4 3V3 → TPS259474LRPWR → 3V3_PROT → future M6 / external load`；后续外部负载不得绕过 M5 使用 raw `3V3`。UVLO / OVLO、ILM、ITIMER、PGTH、dV/dt 与 `TP_M5_RESET` bring-up reset path 已形成 screenshot-reviewed connection baseline；这些阈值与保护行为是 datasheet-based / calculated design basis，不是 measured evidence。M6 与 Cross-Module Integration 尚未完成；当前没有完整 schematic PDF / BOM 或 ERC evidence，未声称实际 `.SchDoc` pin mapping 已完全核对、ERC PASS、schematic complete、PCB 或硬件已验证，Stage 3 尚未完成。
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

当前处于 Stage 3。M1/M2/M3/M4/M5 已进入模块集成边界；下一步是完成 M6 与 Stage 3 Cross-Module Integration，并在 Bring-up 中验证 earliest startup transient 以及 M5 的 UV/OV、过流、短路、热和 latch/reset behavior。建议同步观察 `VBUS / SYS / 3V3 / 3V3_PROT / PG / CHG_ENABLE_CMD / CHG_GATE / CE#`。完整 schematic PDF / BOM、ERC、PCB 与硬件验证均尚未完成。

Repository Authority Cutover 已完成。今后的 Project facts、hardware work、evidence 和 Stage progression 只能在本 Standalone Repository 中进行；Legacy Project 仅保留为 Frozen Migration Source，不再继续活动开发或双写。
