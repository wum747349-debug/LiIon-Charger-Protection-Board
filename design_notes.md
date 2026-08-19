# 设计说明

状态：Stage 2 closeout PASS；Stage 3 documentation closeout COMPLETE；Stage 4 Formal Schematic Review CLOSED；当前进入 Stage 5 — PCB Layout

## 整板架构意图

第一版围绕“USB 系统供电优先、剩余能力充电、电池可独立供电、无电池也能运行”建立电源路径。USB-C 仅作普通 5V Sink；Charger / Power Path 负责 CC-CV、输入与充电电流约束、负载优先和充电退让；System Rail 再经 3.3V 稳压与输出保护供给实验负载。

完整的模式、负载和验收边界以 [requirements.md](requirements.md) 为准。500mA 连续是 Required，1A/100ms 仍是尚未保证的 Design Target。Rev.A selection baseline 为 BQ25185DLHR Charger / Power Path、TPS63802DLAR Buck-Boost、TPS259474LRPWR 输出保护、TUSB320LAIRWBR Type-C current-advertisement detection、PESD5V0U2BT,215 CC ESD 和 TYPE-C-31-M-12 receptacle；这些是 Primary selection，不是 Final BOM。

## Power Path 与无电池运行

实现必须把 System Rail 与充电终止判断从外部负载隔离：USB 有效时负载优先且充电退让，无 USB 时切换到合格电池路径；普通充电器、电池与负载的简单并联不符合该意图。BQ25185DLHR（LCSC C19725033）仍是当前 primary，并承担 Charger / Power Path / BATFET / BUVLO 等职责。当前 M2 记录采用 `R1=600Ω / 1%` 设定约 500mA fast charge；IN/SYS/BAT 去耦分别为 `C1=10uF/25V/0805`、`C2=2.2uF/25V/0603`、1uF。ILIM/VSET 使用 13.0kΩ 与 4.99kΩ 两段网络及 MOSFET bypass：OFF 约 18kΩ 为 conservative state，ON 为 13kΩ high-current state，由 M1 的 `ILIM_HIGH_CMD` 控制。final charge-enable MOSFET 为 Q1，其 drain 与 R5、`CE_N / BQ25185 CE#` 是 common node，gate 接 `CHG_GATE`。热、PCB/layout 和 ERC verification 保持 OPEN。

## 三层保护意图

### 第一层：电池包内部保护板

商品宣称存在包内保护板，但其实际阈值、延时和恢复方式未知，只作为最后安全后备，不作为正常充电、欠压、输出或温度控制点。第一版当前不增加第二套完整电芯保护 IC；这不削弱板级充电管理、功能性 UVLO 与输出保护的职责。

### 第二层：板级充电与电池功能管理

覆盖专用单节锂离子 CC-CV、Power Path、输入和充电电流限制、终止控制、外部 NTC，以及功能性电池 UVLO、迟滞与延时。

Rev.A primary 使用 BQ25185 集成 BUVLO / BATFET 行为：官方资料给出典型 3.0V falling threshold、约 150mV 恢复迟滞，并在低于阈值时断开 BAT-to-SYS，方向上早于商品标称 2.75V cutoff。architecture-level Alternate 为 BQ24075RGTR + TPS3700 + external battery disconnect；它不是 BQ25185 的 drop-in replacement。切换时必须重新核对 functional battery UVLO、hysteresis、battery disconnect、USB / battery transition、deep-discharge startup、quiescent current、thermal behavior 与 charger control。最终全工况行为 Deferred to Stage 3；UVLO 不得关闭有效 USB 的 System Rail 路径。MCP73871T-2CCI/ML 仅保留为 secondary / cross-vendor fallback。

### 第三层：3.3V 输出保护

保护架构需要独立控制外部负载资格：稳压器有效输出建立前保持负载关闭，输出欠压/过压时断开负载，持续过流时关闭，硬短路时快速限制或关闭。Fault policy 已冻结为 latch-off / controlled recovery，primary 的 orderable MPN 规范为 TPS259474LRPWR（LCSC C2864845）；这是 orderable MPN normalization，不是功能性器件替换。官方资料确认 `474L` 是 persistent over-current 后 circuit-break 并 latch-off 的变体；TPS259470LRPWR 只在外部锁断/控制得到设计和验证时作为 conditional fallback，因为 `470x` 的 persistent-over-current 行为是 active current limiting，不能视为等价 Primary / Alternate。

当前 M5 Stage 3 baseline 为 `M4 3V3 → TPS259474LRPWR → 3V3_PROT → M6 / external load`，后续外部负载不得绕过 M5 使用 raw `3V3`。R4/R17/R18 network 对应 nominal UVLO rising ≈3.10V、OVLO rising ≈3.57V；`R19=3.32kΩ / 1% / 1206` 对应约 1A nominal current-limit setting，reviewed ITIMER value 对应约 277ms typical over-current blanking。PG 与 ILM 已明确分离。PGTH divider 为 `R20=75kΩ / 1%` 与 `R21=49.9kΩ / 1%`，nominal threshold 仍约 3.0V，同时提高 divider current 以降低 PGTH leakage-related error；reviewed 2.2nF dV/dt capacitor 设定 startup dV/dt。`C11=10uF/0805` 是 `3V3_PROT` output capacitor。`TP_M5_RESET` 是 EN/UVLO node 上的普通 test point，正常工作时与独立 `TP_GND` 断开；仅在 Bring-up 时可临时短接两者以拉低 EN/UVLO 并方便 latch reset。上述为 datasheet-based / calculated design basis，PGTH threshold 未实测，也不保证严格 ±5% output window 或表示 short-circuit、thermal、1A/100ms transient、latch/reset behavior 已实测 PASS。

## 外部 NTC 意图与限制

Battery Design Basis 仍为商品规格充电温度约 0℃～45℃。Rev.A 外部 NTC 已冻结为 Nanjing Shiheng `MF52D 103F3435-100` / LCSC `C394023`：wired/epoxy、`10kΩ @25℃`、R25 `±1%`、B25/85 `3435K ±1%`、100mm insulated leads，贴近并电气绝缘于 pouch 代表性表面；最终安装方法保持 downstream mechanical input。保持 `J_NTC1 → R15=2.7kΩ / 1% → NTC_TS`；R15 主要缓解 NTC short 将复用的 TS/MR 直接硬拉近 GND 所带来的 push-button / long-low / factory-mode interaction，不作为精密校准声明。`J_NTC1 = HX-XH2.54-2PZZ`，使用 `C5221417` / HX 2.54-2P DT / 200mm / 24AWG 预压接母线束；维护线序为 red → pin 1 → R15 / `NTC_TS`，black → pin 2 → GND，NTC 本体无极性。

M3 保持 `TLV7041DBVR` open-drain comparator 作为高温 charging interlock；中途讨论/临时 capture 的 `TLV7031DBVR` 不是 current design。3V3 经 90.9kΩ/10kΩ divider 产生 nominal 约 0.327V `VHOT_REF`，与 `NTC_TS` 比较。最终 shared qualification 为 `CHG_ENABLE_CMD → R11 10kΩ → CHG_GATE`，同时 `R6 100kΩ` 将 `CHG_GATE` 下拉至 GND，TLV7041 OUT 与 TPS63802 PG 均以 open-drain LOW veto 接入，且 `CHG_GATE` 驱动 charge-enable MOSFET gate。`CHG_ENABLE_CMD` 是唯一 HIGH authorization source，逻辑意图为 `CHG_ENABLE_CMD AND temperature-valid AND 3V3-power-good → charging permitted`。不增加 TPS3808G33DBVR/U6，也不为 TPS63802 PG 增加独立 3V3 pull-up。当前 nominal conservative hardware window 约 4℃～39℃，是工程实现而非新的 manufacturer specification，也不是 verified exact trip range；电阻/comparator/NTC 容差、pouch thermal lag 与实际 trip test 均待后续验证。

## 3.3V Buck-Boost 决策

TPS63802DLAR（LCSC C2845237）保持 primary。当前 Stage 3 exact inductor baseline 为 `L1 = Coilcraft XFL4015-471MEC / 470nH / LCSC C18221164`，连接为 `TPS63802 pin 9 L1 ↔ L1 ↔ pin 7 L2`；EN→SYS、MODE/AGND/GND→GND、FB→FB_3V3、PG→CHG_GATE、VOUT→3V3、VIN→SYS，输入 `C5=10uF/0805`、输出 `C14,C15=22uF/0805`，feedback 为 `3V3 → 511kΩ / 1% → FB_3V3 → 91kΩ / 1% → GND`。L1 footprint 已人工确认；该事实不是 thermal、transient、PCB-layout 或 hardware verification。官方资料中的能力与典型 transient examples 不保证本项目 1A/100ms 目标；纹波、瞬态、电流应力、热和 Layout 仍需后续验证。Normal 3V3 startup / brownout / temperature veto architecture 已形成，但 TPS63802 在输入低于自身 UVLO 的最早期区域存在 datasheet-defined PG `Undefined` 边界，因此 startup sequencing 不能声称 absolute fail-safe；Bring-up 应用示波器观察 `VBUS / SYS / 3V3 / PG / CHG_ENABLE_CMD / CHG_GATE / CE#`。

TPS63070 是 higher-margin fallback。TPS63001 在 3.3V boost mode 的官方额定仅到约 800mA，因此只在 1A peak Design Target 被明确放宽后作为 conditional fallback。

## USB-C 输入与保护决策

- Rev.A 为 5V Sink-only、no USB-PD、no data。TUSB320LAIRWBR（LCSC C132554）是精确 Primary，以 UFP/Sink 角色处理 CC1/CC2，并通过 GPIO OUT1/OUT2 检测 Source current advertisement。
- 当前 M1 capture record 为 `PORT → GND` 固定 UFP/Sink、`ADDR → NC` 选择 GPIO mode、`EN_N → GND`，以及 `VBUS_PROT → 900kΩ → VBUS_DET`。OUT1/OUT2 为 open-drain，各以约 100kΩ pull-up 到 `VDD_CC`。
- TI truth table 中 OUT1/OUT2 的 `H/H、H/L、L/H、L/L` 分别对应 unattached、attached Default、attached 1.5A、attached 3A。`SN74LVC2G132DCUR` 实现 `ILIM_HIGH_CMD = NOT(CC_OUT1)` 和 `CHG_ENABLE_CMD = NAND(CC_OUT1, CC_OUT2)`：unattached 禁止充电并保持 conservative ILIM；Default 使用约 500mA input-current-limit state；1.5A/3A 使用约 1.1A high-current state。3A Source 不表示本 Project 会吸取 3A。
- Legacy USB-A charger + A-to-C cable 按 Type-C Default / conservative policy 处理。Rev.A 不实现 BC1.2 / proprietary charger detection，不承诺利用标签标称的 1.5A / 2A / 3A capability，也不增加 legacy charger-detection IC。
- 接受 TUSB320 GPIO mode 仅基于 attach 时检测结果且无 firmware 动态跟踪后续 Rp capability change 的 Rev.A 限制。Unknown / invalid / unpowered / unqualified state 不得升级为 high-current 或 charge-enabled state；Cross-Module Integration 已完成，startup/transitions 仍须后续 Bring-up 验证。
- `VBUS_PROT → BAT54,215 → VDD_CC`，当前 symbol record 为 BAT54 pin 1/anode → `VBUS_PROT`、pin 3/cathode → `VDD_CC`、pin 2 NC；TUSB320 与 logic 各有 100nF local decoupling。
- CC ESD current Primary 为 Nexperia `PESD5V0U2BT,215`，当前 connection record 为 pin 1 → CC1、pin 2 → CC2、pin 3 → GND，不连接 VBUS。它替换旧 `USBLC6-2SC6`；旧器件只保留为 superseded historical selection，不再是 current Primary。实际 symbol-pin/footprint mapping、ground return 与 layout 仍须依据 Nexperia 官方资料和 EDA library 核对。
- TYPE-C-31-M-12 的全部 VBUS contacts 并到 `VBUS_PROT`，全部 GND contacts 并到 GND；D+/D−/SBU unused / NC。EH1/EH2/EH3/EH4 并到 `USB_SHIELD`；R23 1MΩ 与 `C13=1nF/high-voltage/1206` 是 `USB_SHIELD → GND` 的并联支路。Rev.A 不增加 R24 0-ohm DNP branch。该 EMI/ESD-oriented baseline 未经实验验证，不表示 EMC PASS 或 ESD PASS。
- `VBUS_PROT` 当前有 `D2 = Littelfuse SMF6.0A` / LCSC `C207260` TVS，拓扑为 `VBUS_PROT → TVS → GND`；不默认叠加 input eFuse，布局、回流路径与实际保护表现仍待 downstream verification。
- M1 当前状态为 `COMPLETE FOR STAGE-3 HANDOFF`；不等于 ERC PASS、PCB/layout verified 或 hardware verified。
- 1.6 mm FR-4 是 `Rev.A preliminary PCB thickness baseline`；drawing 未显示明确冲突，但它不是 Final fabrication stackup。层数、铜厚、表面处理、厂家和最终机械公差后续确定。

## 外部接口与测试支持

M6 current baseline 使用 `J_OUT1 = KF301-5.0-2P`，pin 1 接 `3V3_PROT`、pin 2 接 GND；外部负载必须使用 `3V3_PROT`，不得绕过 M5 使用 raw `3V3`。关键 physical test points 覆盖 `VBUS_PROT / SYS / BAT / 3V3 / 3V3_PROT / CHG_GATE / CE# / GND`，并保留 `STAT1 / STAT2 / NTC_TS / M5_RESET`。`TP_CE_N` 位于 U1 pin 4 CE#、R5 与 Q1 drain 的 common node，不是 Q1 gate。TPS63802 PG 已与 `CHG_GATE` 共网，因此不增加重复的 `TP_PG_3V3`；`TP_CHG_GATE` 只能观察 shared qualification node，不能单独识别 `CHG_ENABLE_CMD` path、TLV7041 veto 或 TPS63802 PG veto 中哪个 source 正在拉低。

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
| DN-004 | 0℃～45℃、约 10kΩ / 3435K、NTC fault inhibits charge | 决定温度输入与容差要求 | MF52D 103F3435-100 / C394023 selected；network frozen；trip/thermal-lag hardware verification downstream |
| DN-005 | requirements OPEN-005、OPEN-010～OPEN-013 的输出波形、功耗、热与电流余量边界 | 决定稳压、储能和降额目标 | TPS63802 selected；calculations Deferred to Stage 3 |
| DN-006 | 集成 BUVLO primary 与 output latch-off policy | 决定阈值、迟滞、延时和恢复行为 | Architecture selected；exact behavior Deferred to Stage 3 |
| DN-007 | 关键候选官方 datasheet 集合 | Stage 2 决策必须有官方依据 | Verified sources indexed in references.md |
| DN-008 | TYPE-C-31-M-12 formal drawing/spec 与 1.6 mm preliminary baseline | 决定 receptacle 和机械输入是否足以进入 Stage 3 | Receptacle resolved for Stage 2；无明确板厚冲突；footprint/mechanical finalization Deferred |
| DN-009 | USB source compatibility 与 TUSB320 initial-only limitation | 决定输入限流控制及故障安全边界 | Policy/decode recorded；startup/transitions/mapping verification open |
| DN-010 | M1 CC/VBUS protection、TUSB320 GPIO decode 与 M2/M3 command interfaces | 决定 attach/default/high-current 与 charge-enable behavior | Stage 3 integration complete；ERC and Bring-up sequencing evidence open |

## 当前证据声明

最新版完整 Schematic PDF + BOM 的 Formal Schematic Review、Cross-Module Integration Review 与 BOM Consistency Review 已完成；未发现影响 board-level safety、schematic correctness、package/footprint decision、interface definition 或 critical PCB layout input 的未关闭 blocker。Stage 4 已关闭并显式批准进入 Layout Preflight。普通 commodity R/C 的 exact Manufacturer / Supplier / LCSC sourcing 可延后到 Manufacturing Preparation；BOM placeholder/garbage metadata 属于 manufacturing cleanup，不是 schematic electrical blocker。`.SchDoc` 仍是 authoritative implementation source；仓库当前没有 ERC report，因此 `ERC = NOT YET VERIFIED`。不声称 ERC PASS、PCB DRC、simulation、layout verification、manufacturing、hardware measurement、bring-up、EMI/ESD 或 temperature trip experimentally verified。Battery actual mating/polarity 必须在第一次实际连接前核对；earliest startup transient、M5 UV/OV/PGTH threshold、1A/100ms transient、hard short、thermal 与 latch/reset behavior 均未实测。本文不能用于批准制造。
