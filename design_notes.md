# 设计说明

状态：Stage 2 — Critical Component Selection closeout PASS；Stage 3 active，M1/M2/M3 module design synchronized

## 整板架构意图

第一版围绕“USB 系统供电优先、剩余能力充电、电池可独立供电、无电池也能运行”建立电源路径。USB-C 仅作普通 5V Sink；Charger / Power Path 负责 CC-CV、输入与充电电流约束、负载优先和充电退让；System Rail 再经 3.3V 稳压与输出保护供给实验负载。

完整的模式、负载和验收边界以 [requirements.md](requirements.md) 为准。500mA 连续是 Required，1A/100ms 仍是尚未保证的 Design Target。Rev.A selection baseline 为 BQ25185DLHR Charger / Power Path、TPS63802DLAR Buck-Boost、TPS259474LRPWR 输出保护、TUSB320LAIRWBR Type-C current-advertisement detection、PESD5V0U2BT,215 CC ESD 和 TYPE-C-31-M-12 receptacle；这些是 Primary selection，不是 Final BOM。

## Power Path 与无电池运行

实现必须把 System Rail 与充电终止判断从外部负载隔离：USB 有效时负载优先且充电退让，无 USB 时切换到合格电池路径；普通充电器、电池与负载的简单并联不符合该意图。BQ25185DLHR（LCSC C19725033）仍是当前 primary，并承担 Charger / Power Path / BATFET / BUVLO 等职责。当前 M2 记录采用 `R1=600Ω / 1%` 设定约 500mA fast charge；IN/SYS/BAT 去耦分别为 2.2uF/25V、10uF/25V、1uF。ILIM/VSET 使用 13.0kΩ 与 4.99kΩ 两段网络及 Q1 bypass：Q1 OFF 约 18kΩ 为 conservative state，Q1 ON 为 13kΩ high-current state，由 M1 的 `ILIM_HIGH_CMD` 控制。Q1/Q2 exact MPN baseline 均为 `2N7002BK,215`；热、实际 symbol-pin/footprint mapping 和 EDA/ERC verification 保持 OPEN。

## 三层保护意图

### 第一层：电池包内部保护板

商品宣称存在包内保护板，但其实际阈值、延时和恢复方式未知，只作为最后安全后备，不作为正常充电、欠压、输出或温度控制点。第一版当前不增加第二套完整电芯保护 IC；这不削弱板级充电管理、功能性 UVLO 与输出保护的职责。

### 第二层：板级充电与电池功能管理

覆盖专用单节锂离子 CC-CV、Power Path、输入和充电电流限制、终止控制、外部 NTC，以及功能性电池 UVLO、迟滞与延时。

Rev.A primary 使用 BQ25185 集成 BUVLO / BATFET 行为：官方资料给出典型 3.0V falling threshold、约 150mV 恢复迟滞，并在低于阈值时断开 BAT-to-SYS，方向上早于商品标称 2.75V cutoff。architecture-level Alternate 为 BQ24075RGTR + TPS3700 + external battery disconnect；它不是 BQ25185 的 drop-in replacement。切换时必须重新核对 functional battery UVLO、hysteresis、battery disconnect、USB / battery transition、deep-discharge startup、quiescent current、thermal behavior 与 charger control。最终全工况行为 Deferred to Stage 3；UVLO 不得关闭有效 USB 的 System Rail 路径。MCP73871T-2CCI/ML 仅保留为 secondary / cross-vendor fallback。

### 第三层：3.3V 输出保护

保护架构需要独立控制外部负载资格：稳压器有效输出建立前保持负载关闭，输出欠压/过压时断开负载，持续过流时关闭，硬短路时快速限制或关闭。Fault policy 已冻结为 latch-off / controlled recovery，primary 的 orderable MPN 规范为 TPS259474LRPWR（LCSC C2864845）；这是 orderable MPN normalization，不是功能性器件替换。官方资料确认 `474L` 是 persistent over-current 后 circuit-break 并 latch-off 的变体；TPS259470LRPWR 只在外部锁断/控制得到设计和验证时作为 conditional fallback，因为 `470x` 的 persistent-over-current 行为是 active current limiting，不能视为等价 Primary / Alternate。ILIM、timer、UV/OV 和 reset Deferred to Stage 3。

## 外部 NTC 意图与限制

Battery Design Basis 仍为商品规格充电温度约 0℃～45℃。Rev.A 外部 NTC baseline 为容易采购的 wired/epoxy `10kΩ @25℃`、`β≈3435K`、`±1% preferred`，贴近并电气绝缘于 pouch 代表性表面；exact MPN 与最终安装方法保持 OPEN。`NTC_TS` 经 `R8=2.7kΩ / 1%` 接外部 NTC，R8 主要缓解 NTC short 将复用的 TS/MR 直接硬拉近 GND 所带来的 push-button / long-low / factory-mode interaction，不作为精密校准声明。

M3 增加 `TLV7041DBVR` open-drain comparator 作为高温 charging interlock：3V3 经 90.9kΩ/10kΩ divider 产生 nominal 约 0.327V `VHOT_REF`，与 `NTC_TS` 比较；OUT 与 `CHG_ENABLE_CMD` 经约 4.7kΩ 汇合于 `CHG_GATE`。温度合格且 command 为 HIGH 时 Q2 才能拉低 CE#；低 `NTC_TS` fault 时 comparator 拉低 `CHG_GATE`，R5 将 CE# 拉高。当前 nominal conservative hardware window 约 4℃～39℃，是工程实现而非新的 manufacturer specification，也不是 verified exact trip range；电阻/comparator/NTC 容差、pouch thermal lag 与实际 trip test 均待后续验证。温度故障只撤销 charging qualification，不关闭有效 USB-to-SYS Power Path；M1 command/default decode 已记录，但 3V3 supply/startup 依赖 M4，两者之间的 sequencing 仍待 Cross-Module Integration。

## 3.3V Buck-Boost 决策

TPS63802DLAR（LCSC C2845237）是 primary，理由是其 1.3V～5.5V 输入范围覆盖 1S 路径，官方资料在 VIN≥2.3V、VOUT=3.3V 时给出 2A output-current capability，并提供 100mA→1A 的典型 transient examples。上述资料支持它进入 primary，但不保证本项目 1A/100ms 目标；电感、电容、FB、纹波、瞬态、电流应力、热和 Layout 全部 Deferred to Stage 3。

TPS63070 是 higher-margin fallback。TPS63001 在 3.3V boost mode 的官方额定仅到约 800mA，因此只在 1A peak Design Target 被明确放宽后作为 conditional fallback。

## USB-C 输入与保护决策

- Rev.A 为 5V Sink-only、no USB-PD、no data。TUSB320LAIRWBR（LCSC C132554）是精确 Primary，以 UFP/Sink 角色处理 CC1/CC2，并通过 GPIO OUT1/OUT2 检测 Source current advertisement。
- 当前 M1 capture record 为 `PORT → GND` 固定 UFP/Sink、`ADDR → NC` 选择 GPIO mode、`EN_N → GND`，以及 `VBUS_PROT → 900kΩ → VBUS_DET`。OUT1/OUT2 为 open-drain，各以约 100kΩ pull-up 到 `VDD_CC`。
- TI truth table 中 OUT1/OUT2 的 `H/H、H/L、L/H、L/L` 分别对应 unattached、attached Default、attached 1.5A、attached 3A。`SN74LVC2G132DCUR` 实现 `ILIM_HIGH_CMD = NOT(CC_OUT1)` 和 `CHG_ENABLE_CMD = NAND(CC_OUT1, CC_OUT2)`：unattached 禁止充电并保持 conservative ILIM；Default 使用约 500mA input-current-limit state；1.5A/3A 使用约 1.1A high-current state。3A Source 不表示本 Project 会吸取 3A。
- Legacy USB-A charger + A-to-C cable 按 Type-C Default / conservative policy 处理。Rev.A 不实现 BC1.2 / proprietary charger detection，不承诺利用标签标称的 1.5A / 2A / 3A capability，也不增加 legacy charger-detection IC。
- 接受 TUSB320 GPIO mode 仅基于 attach 时检测结果且无 firmware 动态跟踪后续 Rp capability change 的 Rev.A 限制。Unknown / invalid / unpowered / unqualified state 不得升级为 high-current 或 charge-enabled state；实际 startup/transitions 仍须 Cross-Module Integration 核对。
- `VBUS_PROT → BAT54,215 → VDD_CC`，当前 symbol record 为 BAT54 pin 1/anode → `VBUS_PROT`、pin 3/cathode → `VDD_CC`、pin 2 NC；TUSB320 与 logic 各有 100nF local decoupling。
- CC ESD current Primary 为 Nexperia `PESD5V0U2BT,215`，当前 connection record 为 pin 1 → CC1、pin 2 → CC2、pin 3 → GND，不连接 VBUS。它替换旧 `USBLC6-2SC6`；旧器件只保留为 superseded historical selection，不再是 current Primary。实际 symbol-pin/footprint mapping、ground return 与 layout 仍须依据 Nexperia 官方资料和 EDA library 核对。
- TYPE-C-31-M-12 的全部 VBUS contacts 并到 `VBUS_PROT`，全部 GND contacts 并到 GND；D+/D−/SBU unused / NC。Connector shield / EH 最终接地策略仍 OPEN。
- `VBUS_PROT` 当前有 `SMF5.0A` TVS topology；准确 manufacturer/orderable MPN 尚未冻结，不默认叠加 input eFuse，也不把 generic symbol 升级成新的精确采购事实。
- M1 当前状态为 `SUBSTANTIALLY COMPLETE FOR STAGE-3 MODULE INTEGRATION`，基于用户报告的 Altium capture 与主要接线修正；不等于实际 `.SchDoc` mapping verified、ERC PASS 或 schematic complete。
- 1.6 mm FR-4 是 `Rev.A preliminary PCB thickness baseline`；drawing 未显示明确冲突，但它不是 Final fabrication stackup。层数、铜厚、表面处理、厂家和最终机械公差后续确定。

## 电池包证据边界

Rev.A Battery Design Basis 已改为 1S protected Li-Polymer pouch pack、103450、3.7V / 2000mAh / 约 7.4Wh。容量、电流、温度、尺寸、重量和保护类型来自用户提供的商品规格，不是官方电芯制造商 PDF datasheet；包内保护阈值、延时、恢复方式与一致性保持 Not Verified。包内保护只作为最后安全后备。

## 制造与调试原则

- 第一版优先手工焊接、调试便利性和可测性。
- 高电流路径需在后续阶段依据真实电流、铜厚、允许压降和温升核对铜宽、过孔与散热。
- 电池接口需降低极性误接风险；2.54mm-class 两线连接器的准确 mating series、键位、线序和极性在首次接入前必须实物核对。
- 软包电池固定、绝缘、引线应力释放、膨胀/公差空间以及防刺穿、防折弯、防挤压和防锐边接触要求以 requirements 为准。
- 板框、接口位置、层数、铜厚、表面处理、PCB 厂家和装配方式保持 Open；板厚仅有 1.6 mm FR-4 preliminary baseline。

## Stage 2 决策与剩余输入

| ID | Needed decision / evidence | Why it matters | Current state |
| --- | --- | --- | --- |
| DN-001 | 当前电池包充电/放电/温度商品规格和保护行为 | 约束充电、UVLO、热与保护协调 | Design Basis accepted；内部保护行为 Not Verified |
| DN-002 | 4.20V、默认约 500mA、商品允许约 1.0A | 决定充电方案筛选边界 | Resolved for Stage 2；passive values Deferred to Stage 3 |
| DN-003 | 5V 输入预算与各运行模式功率预算 | 决定负载优先、充电退让与热边界 | Architecture selected；detailed budget Deferred to Stage 3 |
| DN-004 | 0℃～45℃、约 10kΩ / 3435K、NTC fault inhibits charge | 决定温度输入与容差要求 | Architecture selected；MPN/network Deferred to Stage 3 |
| DN-005 | requirements OPEN-005、OPEN-010～OPEN-013 的输出波形、功耗、热与电流余量边界 | 决定稳压、储能和降额目标 | TPS63802 selected；calculations Deferred to Stage 3 |
| DN-006 | 集成 BUVLO primary 与 output latch-off policy | 决定阈值、迟滞、延时和恢复行为 | Architecture selected；exact behavior Deferred to Stage 3 |
| DN-007 | 关键候选官方 datasheet 集合 | Stage 2 决策必须有官方依据 | Verified sources indexed in references.md |
| DN-008 | TYPE-C-31-M-12 formal drawing/spec 与 1.6 mm preliminary baseline | 决定 receptacle 和机械输入是否足以进入 Stage 3 | Receptacle resolved for Stage 2；无明确板厚冲突；footprint/mechanical finalization Deferred |
| DN-009 | USB source compatibility 与 TUSB320 initial-only limitation | 决定输入限流控制及故障安全边界 | Policy/decode recorded；startup/transitions/mapping verification open |
| DN-010 | M1 CC/VBUS protection、TUSB320 GPIO decode 与 M2/M3 command interfaces | 决定 attach/default/high-current 与 charge-enable behavior | M1 substantially complete for integration；mapping/ERC and cross-module sequencing open |

## 当前证据声明

Stage 2 closeout PASS 仅表示关键架构器件及方向足以进入模块级详细设计，不是 Final BOM。用户已手工 capture 当前 M1/M2/M3 schematic draft；`.SchDoc` 仍是 authoritative implementation source，且当前没有完整 schematic PDF/BOM/ERC evidence，因此不声称实际 pin mapping fully verified、schematic complete、ERC PASS、footprint/PCB/hardware verified 或 temperature trip experimentally verified。M4/M5/M6、Cross-Module Integration 以及 M3 `3V3 startup` 与 `CHG_ENABLE_CMD` sequencing 仍未完成。本文不能用于批准制造。
