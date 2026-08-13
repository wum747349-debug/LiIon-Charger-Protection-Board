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
- 当前 Rev.A primary baseline 包括 BQ25185DLHR、TPS63802DLAR、TPS259474LRPWR、TUSB320LAIRWBR、USBLC6-2SC6 和 TYPE-C-31-M-12；NTC、VBUS TVS、battery connector 与普通无源器件的准确选型/实物边界按计划 Deferred to Stage 3。该 selection baseline 不是 Final BOM。
- Stage 3 — Schematic Module Design and Capture 仍在进行。M2 Charger / Power Path 已达到 `COMPLETE FOR STAGE-3 MODULE INTEGRATION`，M3 Battery + External NTC 已基本完成当前 schematic capture / integration 所需设计记录；用户已在 Altium Designer 中手工 capture 当前 M2/M3 schematic draft。
- M1、M4、M5、M6 与 Cross-Module Integration 尚未完成；当前没有完整 schematic PDF / BOM 或 ERC evidence，未声称实际 `.SchDoc` pin mapping 已完全核对、ERC PASS、schematic complete、PCB 或硬件已验证，Stage 3 尚未完成。
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

当前处于 Stage 3。M2/M3 已进入模块集成边界，下一项硬件设计任务是 `M1 — USB-C Input / CC / Current Capability Control`；之后仍需 M4/M5/M6 与 Cross-Module Integration。完整 schematic PDF / BOM、ERC、PCB 与硬件验证均尚未完成。

Repository Authority Cutover 已完成。今后的 Project facts、hardware work、evidence 和 Stage progression 只能在本 Standalone Repository 中进行；Legacy Project 仅保留为 Frozen Migration Source，不再继续活动开发或双写。
