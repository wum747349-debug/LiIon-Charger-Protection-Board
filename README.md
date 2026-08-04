# 单节锂电池充电与保护电源板

Project Identity: LiIon Charger Protection Board
Current Project Stage: Bootstrap
Hardware Revision: TBD

## Purpose

建立一块以 USB-C 5V 供电的单节锂电池充电、保护与 3.3V 输出电源板；当前仅完成独立仓库 Bootstrap，不表示硬件设计已经开始。

## Current Status

- Framework binding and initialization status: [FRAMEWORK.md](FRAMEWORK.md)
- Project facts are Draft until confirmed in this repository.
- No component, EDA, ERC, DRC, manufacturing, bring-up, or test result is implied by Bootstrap.

## Bootstrap Facts

- USB-C 5V input is an intended project boundary.
- Intended functions include single-cell lithium-ion charging, battery protection, 3.3V output, charge-status indication, and key test points.
- Component choices, electrical parameters, manufacturing constraints, and acceptance criteria remain TBD for Phase 3B.
- No firmware is currently required.

## Project Facts

- [Requirements baseline](requirements.md)
- [Block diagram](block_diagram.md)
- [Design notes](design_notes.md)
- [Reference index](references.md)

## Repository Navigation

- [Project runtime rules](PROJECT_RULES.md)
- [Stage document responsibilities](docs/README.md)
- [Hardware source and evidence responsibilities](hardware/README.md)
- [Local reference responsibilities](references/README.md)
- [Project validator](scripts/validate_project_repository.py)

## Migration Provenance

Source Repository: wum747349-debug/Hardware-Practice-Projects

Legacy Project Path: projects/02_LiIon_Charger_Protection_Board

Migration Date: 2026-08-04

Framework Release: development-v0.9

Framework Commit: 4261bdfa3cf6be400be16722c27fca22ea1895ad

Git History Strategy: Clean Import

Legacy Repository: wum747349-debug/Hardware-Practice-Projects

Legacy Fact Import Commit: TBD

Authority Status: Migration Pilot — Legacy Project remains Current Authority

## Next Step

STOP — Await human review before Phase 3B.
