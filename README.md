# 单节锂电池充电与保护电源板

Project Identity: LiIon Charger Protection Board
Current Project Stage: Bootstrap
Hardware Revision: TBD

## 项目目的

建立一块以 USB-C 5V 供电的单节锂电池充电、保护与 3.3V 输出电源板；当前仅完成独立仓库 Bootstrap，不表示硬件设计已经开始。

## 当前状态

- Framework 绑定与初始化状态：[FRAMEWORK.md](FRAMEWORK.md)
- 项目事实经本仓库确认前均为草稿。
- Bootstrap 不表示器件、EDA、ERC、DRC、制造、上电调试或测试已有结果。

## 初始化事实

- 项目计划使用 USB-C 5V 输入。
- 计划功能包括单节锂电池充电、电池保护、3.3V 输出、充电状态指示和关键测试点。
- 器件选择、详细电气参数、制造约束和验收标准将在对应项目阶段中逐步确认，当前未决项继续保持 TBD。
- 当前没有 firmware 需求。

## 项目事实

- [需求基线](requirements.md)
- [系统框图](block_diagram.md)
- [设计说明](design_notes.md)
- [资料索引](references.md)

## 仓库导航

- [项目运行规则](PROJECT_RULES.md)
- [阶段文档职责](docs/README.md)
- [硬件源文件与证据职责](hardware/README.md)
- [本地资料职责](references/README.md)
- [项目验证器](scripts/validate_project_repository.py)

## 迁移来源

Source Repository: wum747349-debug/Hardware-Practice-Projects

Legacy Project Path: projects/02_LiIon_Charger_Protection_Board

Migration Date: 2026-08-04

Framework Release: development-v0.9

Framework Commit: 4261bdfa3cf6be400be16722c27fca22ea1895ad

Git History Strategy: Clean Import

Legacy Repository: wum747349-debug/Hardware-Practice-Projects

Authority Status: Migration Pilot — Legacy Project remains Current Authority

## 下一步

等待人工审查后再决定是否进入需求定义阶段。
