# 单节锂电池充电与保护电源板

Project Identity: LiIon Charger Protection Board
Current Project Stage: Stage 1 — Requirements Definition
Hardware Revision: TBD

## 项目目的

本项目是一块面向 STM32、普通传感器及类似低压实验负载的锂电池实验电源板。第一版以普通 5V USB-C 输入、单节锂离子电池充电、真实 Power Path / Load Sharing、无电池运行、板级功能保护及 3.3V 稳压输出为范围；充电是核心功能，当前没有 firmware 需求。

## 当前状态

- Initialization Status: Initialized
- Gate 1.5 已获得明确人工 PASS；该决定只批准 Project Initialization 与 Stage 1 Requirements Baseline，不证明硬件设计或验证完成。
- 自动 Validator 仅用于检查结构与需求入口完整性。
- 尚未选择最终充电、Power Path、稳压、保护或 NTC 器件。
- 尚未开始 EDA、ERC、DRC、制造、焊接、Bring-up 或 Test；文档中的验收项均保持 Planned、Not Verified 或 Open。
- 本 Standalone Repository 是 Only Active Project Authority；Legacy Project 2 仅作为 Frozen Migration Source 保留，用于历史迁移追溯。

## 项目事实

- [需求基线](requirements.md)
- [系统框图](block_diagram.md)
- [设计说明](design_notes.md)
- [资料索引](references.md)

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

当前仍处于 Stage 1，Stage 2 尚未授权；在获得独立人工授权前，不得创建 Stage 2 文件或开始器件选择。

Repository Authority Cutover 已完成，但不构成 Stage 2 授权。今后的 Project facts、hardware work、evidence 和 Stage progression 只能在本 Standalone Repository 中进行；Legacy Project 仅保留为 Frozen Migration Source，不再继续活动开发或双写。
