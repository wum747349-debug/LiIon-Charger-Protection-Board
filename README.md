# 单节锂电池充电与保护电源板

Project Identity: LiIon Charger Protection Board
Current Project Stage: Stage 7 — PCB Release Review
Hardware Revision: Rev.A

## 项目目的

本项目是一块面向 STM32、普通传感器及类似低压实验负载的锂电池实验电源板。第一版以普通 5V USB-C 输入、单节锂离子电池充电、真实 Power Path / Load Sharing、无电池运行、板级功能保护及 3.3V 稳压输出为范围；充电是核心功能，当前没有 firmware 需求。

## 当前状态

- Initialization Status: Initialized
- Battery Requirements Baseline Amendment 已按绑定 Framework 重新执行 Gate 1.5 一致性核对并通过；该结论不证明硬件设计或验证完成。
- Stage 2 — Critical Component Selection 已完成；当前 Rev.A primary baseline 包括 BQ25185DLHR、TPS63802DLAR、TPS259474LRPWR、TUSB320LAIRWBR、PESD5V0U2BT,215 和 TYPE-C-31-M-12；M2 switching MOSFET exact MPN baseline 为 2N7002BK,215，M4 L1 exact baseline 为 Murata DFE201612E-R47M=P2 / 470nH / LCSC C668312。External NTC 已冻结为 Nanjing Shiheng `MF52D 103F3435-100` / LCSC `C394023`，D2 已冻结为 Littelfuse `SMF6.0A` / LCSC `C207260`。
- Stage 4 Formal Schematic Review 已关闭并批准进入 PCB Layout；ERC 仍为 `NOT YET VERIFIED`，没有被提升为 PASS。
- Stage 5 Layout Preflight / Placement Closeout 已完成，Rev.A placement 已冻结。
- Stage 6 Routing and Copper 已完成 B3 Layout / Routing Review。当前用户提供的最终本地 PCB 证据显示：主要 routing 完成、Top / Bottom GND Polygon 已 Repour、Bottom GND plane 基本连续、关键 GND via / stitching 已收尾，`Un-Routed Nets = 0`，未发现需要重新 Placement 或大规模 reroute 的 blocker。
- 2026-08-25 21:31:22 对当前本地 `.PcbDoc` 运行完整 Batch DRC：Warnings = 0，Rule Violations = 213；其中 Clearance = 0、Short Circuit = 0、Un-Routed Nets = 0、Modified Polygon = 0、Width_Power = 0、Width_Default = 0、Hole-To-Hole = 0、Minimum Solder Mask Sliver = 0、Net Antennae = 0。剩余为 4 个 M3 NPTH mounting-hole Hole Size rule hits、200 个 Silk To Solder Mask、9 个 Silk To Silk。
- Rev.A 最终 POWER width engineering baseline 为 `Min 8 mil / Preferred 20 mil / Max 100 mil`，Scope 为 `InNetClass('POWER')`。8 mil 只允许用于小封装 escape / 很短 neck-down；普通 power routing 目标仍为 20 mil 或更宽铜皮 / Polygon。
- Top / Bottom GND Polygon 当前采用 Solid Pour；`Remove Islands Less Than = 2500 sq.mil`、`Remove Necks Less Than = 10 mil`、`Remove Dead Copper = enabled`、`Pour Over Same Net Objects`。
- 四个安装孔为 M3 mechanical NPTH，实际孔径约 3.20 mm / 125.984 mil，`Plated = OFF`。用户明确选择不再为这四个机械孔新增 Hole Size Rule exception；当前 4 个 Hole Size DRC hits 因此作为已知 Release finding 保留，不代表安装孔尺寸错误。
- 用户明确选择不再批量清理当前普通 silkscreen DRC hits。Silkscreen findings 仅允许在 Gerber Viewer 中确认关键接口、极性、Pin 1、测试点和用户可见文字仍清晰后作为 release-stage waiver 接受；若关键文字被裁切或不可读，则仍需修正。
- M1 当前 TUSB320 VBUS_DET baseline 为 `VBUS_PROT → R18 910kΩ / 1% → VBUS_DET`。USB shield baseline 为 EH1–EH4 并入 `USB_SHIELD`，再由 R19 1MΩ 与 C13 1nF/high-voltage/1206 两个并联支路接 GND；Rev.A 不增加 R24 0Ω DNP branch。
- `J_NTC1 = HX-XH2.54-2PZZ`，外接 `C5221417`；`J_BAT1` pin 1 → BAT、pin 2 → GND。实际电池 mating 与 polarity/wire order 仍为 `PHYSICAL VERIFICATION PENDING`，必须在第一次实际连接电池前完成。
- M5 保持 `U2 = TPS259474LRPWR`，输出边界为 `M4 3V3 → U2 → 3V3_PROT → M6 / external load`。外部负载不得绕过 M5 使用 raw `3V3`。
- 当前已进入 Stage 7 — PCB Release Review / Manufacturing Preparation。制造放行仍为 `NOT YET AUTHORIZED`，必须完成 Gerber / Drill / Board Outline / Solder Mask / Silkscreen / PTH-NPTH / manufacturing-parameter viewer review，并确认制造输出与当前 PCB / DRC / BOM 版本一致后才能批准下单。
- 本 Standalone Repository 是 Only Active Project Authority；Legacy Project 2 仅作为 Frozen Migration Source 保留。

## 项目事实

- [需求基线](requirements.md)
- [系统框图](block_diagram.md)
- [设计说明](design_notes.md)
- [资料索引](references.md)
- [Stage 2 关键器件选型计划](docs/component_selection_plan.md)
- [Stage 3 模块设计入口](docs/module_design/README.md)
- [Stage 4 原理图审查记录](docs/schematic_review.md)
- [PCB 设计规则与最终工程基线](docs/pcb_design_rules.md)
- [PCB Layout / Routing / Release Review](docs/pcb_review.md)

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

Stage 6 已关闭并批准进入 Stage 7。下一步只做 PCB Release / Manufacturing Preparation：冻结当前 PCB 设计、导出当前版本 Gerber + NC Drill，在 Gerber Viewer 中逐层核对铜层、阻焊、丝印、Board Outline、PTH / NPTH、安装孔和关键接口标识；确认输出包与当前 PCB / BOM / DRC 版本一致后，再给出 `批准制造 / 有条件批准 / 不批准制造` 的显式结论。

Bring-up 仍保留 NTC actual trip temperature / pouch thermal lag、battery pre-first-connection physical verification、earliest startup transient，以及 M5 的 UV/OV、PGTH、过流、短路、热和 latch/reset behavior 等 downstream verification。
