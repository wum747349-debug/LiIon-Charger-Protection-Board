# 单节锂电池充电与保护电源板

Project Identity: LiIon Charger Protection Board
Current Project Stage: Stage 8 — Assembly, Bring-up and Hardware Test
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
- 2026-08-25 21:31:22 对最终 Rev.A `.PcbDoc` 运行的 Final Full Batch DRC 已完成并复核：Warnings = 0，Rule Violations = 213；其中 Clearance = 0、Short Circuit = 0、Un-Routed Nets = 0、Modified Polygon = 0、Width_Power = 0、Width_Default = 0、Hole-To-Hole = 0、Minimum Solder Mask Sliver = 0、Net Antennae = 0。剩余为 4 个 M3 NPTH mounting-hole Hole Size rule hits、200 个 Silk To Solder Mask、9 个 Silk To Silk。
- Rev.A 最终 POWER width engineering baseline 为 `Min 8 mil / Preferred 20 mil / Max 100 mil`，Scope 为 `InNetClass('POWER')`。8 mil 只允许用于小封装 escape / 很短 neck-down；普通 power routing 目标仍为 20 mil 或更宽铜皮 / Polygon。
- Top / Bottom GND Polygon 当前采用 Solid Pour；`Remove Islands Less Than = 2500 sq.mil`、`Remove Necks Less Than = 10 mil`、`Remove Dead Copper = enabled`、`Pour Over Same Net Objects`。
- 四个安装孔为 M3 mechanical NPTH，实际孔径约 3.20 mm / 125.984 mil，`Plated = OFF`。最终 manufacturing viewer 已确认四个 M3 安装孔按预期表示为 NPTH / non-plated mechanical holes；RL-001 已关闭并接受。
- 最终 manufacturing viewer 已完成 critical marking review；所需 connector / polarity / Pin 1 / power / GND / critical TP 信息保持可理解。普通 silkscreen DRC findings 按此前批准的 waiver 接受；RL-002 已关闭。
- M1 当前 TUSB320 VBUS_DET baseline 为 `VBUS_PROT → R18 910kΩ / 1% → VBUS_DET`。USB shield baseline 为 EH1–EH4 并入 `USB_SHIELD`，再由 R19 1MΩ 与 C13 1nF/high-voltage/1206 两个并联支路接 GND；Rev.A 不增加 R24 0Ω DNP branch。
- `J_NTC1 = HX-XH2.54-2PZZ`，外接 `C5221417`；`J_BAT1` pin 1 → BAT、pin 2 → GND。Rev.A Bring-up 已实际接入所选电池并完成 Battery-only / charging 测试，当前线序与极性在本次实板上已被实际工作结果支持；首次接入前的独立极性读数未单独留档，准确 mating series / 批次一致性仍不扩展为普遍结论。
- M5 保持 `U2 = TPS259474LRPWR`，输出边界为 `M4 3V3 → U2 → 3V3_PROT → M6 / external load`。外部负载不得绕过 M5 使用 raw `3V3`。
- Stage 7 PCB Release Review 已完成。Final Full Batch DRC、最终 Gerber / Drill 制造数据、嘉立创 manufacturing viewer、NPTH / silkscreen findings、制造参数和 PCB / BOM / manufacturing-output 版本一致性均已复核；Manufacturing Release Decision = `批准制造`，Rev.A PCB 已提交打板。
- Stage 8 已完成 Rev.A 实板焊接并进入实际 Bring-up。当前已获得 USB-only、3.8V bench Battery-only、真实 Battery-only、USB + Battery 充电等实测证据；代表性实测包括 USB-only `VBUS_PROT≈5.07V / SYS≈4.67V / 3V3≈3.30V / 3V3_PROT≈3.29V`，真实 Battery-only `BAT≈SYS≈3.95V / 3V3≈3.31V / 3V3_PROT≈3.29V`。
- 充电 Bring-up 中曾出现 `NTC_TS` 间歇性掉到约 0V、充电被禁止的问题。排查确认外部 NTC 本体约 9kΩ、R15约2.7kΩ，最终定位为 NTC 端子 / 接触不良；更换端子后恢复稳定。正常时 `NTC_TS≈0.46V`，真实电池串联测得充电电流约 `0.49A`，与 Rev.A 约500mA快充设计目标一致。
- 重复 USB 插入 / 拔出后 `3V3_PROT` 稳态保持约3.29V；一次早期插入曾出现万用表约0.4V的瞬时读数，但未用示波器复现或定量，因此不宣称 zero-drop / seamless transfer。
- Stage 8 仍未完成完整 G3：`VBUS_PROT` bench first-power injection、示波器启动/纹波/切换波形、BUVLO扫阈值、3.3V/500mA连续负载、1A瞬态、过流/短路/UV/OV、效率、定量热测试及NTC精确温度阈值均保持 Deferred / Not Yet Verified。当前可声明的是主电源路径和基本约500mA充电功能已由实板测量支持，而不是整板性能验证全部通过。
- 本 Standalone Repository 是 Only Active Project Authority；Legacy Project 2 仅作为 Frozen Migration Source 保留。

## 项目事实

- [需求基线](requirements.md)
- [系统框图](block_diagram.md)
- [设计说明](design_notes.md)
- [资料索引](references.md)
- [Stage 2 关键器件选型计划](docs/component_selection_plan.md)
- [系统设计与器件选型说明](docs/project_design_overview.md)
- [Stage 3 模块设计入口](docs/module_design/README.md)
- [Stage 4 原理图审查记录](docs/schematic_review.md)
- [PCB 设计规则与最终工程基线](docs/pcb_design_rules.md)
- [PCB Layout / Routing / Release Review](docs/pcb_review.md)
- [Stage 8 装配与 Bring-up 记录 / 测试计划](docs/bringup_log.md)

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

完成 Rev.A 手工焊接后，按 `docs/bringup_log.md` 的 Gate-driven 执行计划开始 Stage 8 Bring-up：先完成 B0 目视检查和 B1 不上电静态筛查，通过 G1 后，以限流可调电源注入 `VBUS_PROT` 完成 B2A 受控首次上电，再用正常 USB-C Source 通过 `USBC1` 完成 B2B native USB-C 上电验证，并沿 `VBUS_PROT → SYS → 3V3 → 3V3_PROT` 逐级测量。

随后执行 B3 启动 / 基础纹波观察和 B4 Battery-only bench-supply emulation / BUVLO sweep；真实电池首次连接前必须通过 G2，完成 battery mating / polarity / wire-order 与 NTC sanity check。真实电池首次接入采用 USB absent 的 B5 Battery-only 模式；确认正常后再进行 B6 充电行为、B7 USB / Battery Power Path 切换和 B8 NTC / charging qualification。只有这些基础路径取得真实证据且无未解决的 safety-critical blocker 时，才可通过 G3 Basic Rev.A Bring-up Gate。

3.3V / 500mA 连续负载、1A / 100ms 瞬态、M5 精确过流 / 短路 / latch 行为、精确温升和 NTC actual trip temperature 等项目仍保持 `DEFERRED / NOT YET VERIFIED`，不得因 G3 通过而自动视为 PASS。