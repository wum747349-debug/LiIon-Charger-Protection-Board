# Li-Ion Charger & Protection Board

面向 STM32、传感器及类似低压嵌入式负载的单节锂离子电池电源板，集成 USB-C 输入、单节锂电池充电、Power Path / Load Sharing、3.3 V Buck-Boost 稳压和输出保护。

**Hardware Revision:** Rev.A — Fabricated, assembled, and primary-function bring-up verified  
**Repository Design Files:** Latest maintained Altium Designer source  
**Current Stage:** Stage 8 — Basic Bring-up Complete / Extended Performance & Protection Validation Deferred

> The repository contains the latest maintained Altium project files. The PCB design includes minor post-fabrication layout refinements; measured results in this repository refer to the physical Rev.A board.

## Project Overview

Rev.A 以普通 5 V USB-C 输入和单节锂离子电池为两类输入源，目标是在 USB 接入时优先为系统供电并完成电池充电，在 USB 断开后由电池继续维持 3.3 V 系统输出。

项目范围聚焦硬件设计与验证，不包含固件需求。当前已完成原理图、PCB、PCBA、基础 Bring-up、实际充电测试和一次真实故障定位闭环。

## Key Features

- USB-C 5 V 输入；
- 单节锂离子电池充电；
- USB / Battery Power Path 与无电池运行；
- 3.3 V Buck-Boost 稳压输出；
- 3.3 V 输出保护；
- 外接 NTC 温度资格链；
- 原理图、PCB、BOM 与 Altium Designer 源文件；
- 实板 Bring-up、Debug 与测量记录。

## System Architecture

```text
USB-C 5 V
   │
   ├─ CC / input protection
   │
   └─ BQ25185 charger + power path ── BAT
                    │
                   SYS
                    │
             TPS63802 buck-boost
                    │
                   3V3
                    │
             TPS259474 protection
                    │
                3V3_PROT
                    │
              External Load
```

主要器件包括：

- `BQ25185DLHR` — Charger / Power Path；
- `TPS63802DLAR` — 3.3 V Buck-Boost；
- `TPS259474LRPWR` — 输出保护；
- `TUSB320LAIRWBR` — USB-C CC / current advertisement detection；
- `PESD5V0U2BT,215` — CC ESD protection；
- `MF52D 103F3435-100` — 外接 10 kΩ NTC。

## Rev.A Verification Snapshot

| Item | Rev.A result |
| --- | --- |
| USB-only | `VBUS_PROT≈5.07 V / SYS≈4.67 V / 3V3≈3.30 V / 3V3_PROT≈3.29 V` |
| Battery-only | `BAT≈SYS≈3.95 V / 3V3≈3.31 V / 3V3_PROT≈3.29 V` |
| USB + Battery | Power path and charging operation verified |
| Charging current | approximately `0.49 A` |
| Repeated USB insertion / removal | `3V3_PROT` steady state remained approximately `3.29 V` in repeated checks |
| 3.3 V ripple / noise observation | representative measurement `Vpp≈12.4 mV`, `Vrms≈3.53 mV` under the recorded probe / scope conditions |
| NTC debug | intermittent connector contact fault identified and corrected |

The ripple result above is a low-frequency ripple / noise observation under the recorded measurement setup. The displayed approximately 50 Hz component is not interpreted as the TPS63802 switching frequency.

## Bring-up & Debug Highlight

During charging bring-up, `NTC_TS` intermittently collapsed to approximately 0 V and charging was disabled. The external NTC and bias resistor values were checked first, followed by comparison tests with the battery and NTC connected / disconnected.

The fault was ultimately traced to poor contact at the external NTC connector. Replacing the connector restored stable operation. Under normal operation `NTC_TS≈0.46 V`, and the measured battery charging current was approximately `0.49 A`, consistent with the Rev.A approximately 500 mA fast-charge target.

Detailed execution evidence is retained in [`docs/bringup_log.md`](docs/bringup_log.md).

## Hardware Design & PCB

This repository intentionally retains the engineering records behind the board rather than reducing the project to a showcase-only summary. Requirements, architecture, component selection, schematic review, PCB rules, PCB review and Bring-up remain available for deeper inspection.

The Altium project is available under [`hardware/altium_project/`](hardware/altium_project/). Human-readable exports are under [`hardware/outputs/`](hardware/outputs/).

The current `.SchDoc` is consistent with the Rev.A design basis. The maintained `.PcbDoc` contains minor layout / copper / via / solder-mask refinements made after fabrication, so it is not presented as a byte-equivalent copy of the manufacturing source used for the physical Rev.A board.

## Repository Navigation

### Engineering definition

- [Requirements baseline](requirements.md)
- [System block diagram](block_diagram.md)
- [Design notes](design_notes.md)
- [Reference / evidence index](references.md)

### Design and review records

- [Critical component selection](docs/component_selection_plan.md)
- [System design overview](docs/project_design_overview.md)
- [Module-level circuit design](docs/module_design/README.md)
- [Formal schematic review](docs/schematic_review.md)
- [PCB design rules](docs/pcb_design_rules.md)
- [PCB layout / routing / release review](docs/pcb_review.md)
- [Assembly / Bring-up / Debug log](docs/bringup_log.md)

### Hardware files

- [Hardware source and evidence guide](hardware/README.md)
- [Altium Designer project](hardware/altium_project/)
- [Schematic / BOM outputs](hardware/outputs/)

### Project rules and workflow

- [Framework binding](FRAMEWORK.md)
- [Project rules](PROJECT_RULES.md)
- [Documentation guide](docs/README.md)
- [Project validator](scripts/validate_project_repository.py)

The project-level rules remain in this repository because they are part of the normal engineering workspace. The reusable workflow and broader methodology are maintained separately in the [Hardware Project Framework](https://github.com/wum747349-debug/Hardware-Project-Framework) repository.

## AI-assisted Engineering

AI tools were used as engineering assistants for tasks such as requirements analysis, datasheet review, component-selection comparison, schematic / PCB review support, test planning, debugging reasoning and documentation maintenance.

AI-generated suggestions are not treated as hardware evidence by themselves. Critical design decisions are checked against source material, EDA implementation and measured hardware results where applicable. The detailed reusable workflow is kept in the separate Hardware Project Framework repository rather than duplicated here.

## Current Limitations

Rev.A primary-function basic Bring-up is complete, but the following items remain deferred and are not claimed as verified performance:

- 3.3 V / 500 mA continuous-load validation and loaded ripple;
- 1 A / 100 ms transient response;
- precise BUVLO shutdown / recovery thresholds;
- controlled over-current / short-circuit / UV / OV protection testing;
- efficiency and quantitative thermal characterization;
- precise NTC hot / cold trip thresholds and thermal hysteresis;
- stricter high-frequency ripple characterization.

These deferred items do not represent unresolved basic-function failures; they define the current validation boundary of Rev.A.
