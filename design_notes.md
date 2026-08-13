# 设计说明

状态：Stage 2 — Critical Component Selection closeout PASS；Stage 3 entry authorized

## 整板架构意图

第一版围绕“USB 系统供电优先、剩余能力充电、电池可独立供电、无电池也能运行”建立电源路径。USB-C 仅作普通 5V Sink；Charger / Power Path 负责 CC-CV、输入与充电电流约束、负载优先和充电退让；System Rail 再经 3.3V 稳压与输出保护供给实验负载。

完整的模式、负载和验收边界以 [requirements.md](requirements.md) 为准。500mA 连续是 Required，1A/100ms 仍是尚未保证的 Design Target。Rev.A selection baseline 为 BQ25185DLHR Charger / Power Path、TPS63802DLAR Buck-Boost、TPS259474LRPWR 输出保护、TUSB320LAIRWBR Type-C current-advertisement detection、USBLC6-2SC6 CC ESD 和 TYPE-C-31-M-12 receptacle；这些是 Primary selection，不是 Final BOM。

## Power Path 与无电池运行

实现必须把 System Rail 与充电终止判断从外部负载隔离：USB 有效时负载优先且充电退让，无 USB 时切换到合格电池路径；普通充电器、电池与负载的简单并联不符合该意图。BQ25185DLHR（LCSC C19725033）仍是当前 primary，最符合 Rev.A architecture，并同时承担 Charger / Power Path / BATFET / BUVLO 等职责。其官方资料确认 Power Path 可在电池 empty/charging 时由 IN 维持 SYS，并在输入受限时优先系统负载、降低充电电流。当前采购 availability 是 point-in-time evidence，仅记录为 `Limited`；真正采购前必须重新检查，不把具体库存数量固化为 Project Fact。Rev.A 默认快充目标约 500mA，商品规格允许约 1.0A；ISET、ILIM/VSET、热和状态机计算全部 Deferred to Stage 3。

## 三层保护意图

### 第一层：电池包内部保护板

商品宣称存在包内保护板，但其实际阈值、延时和恢复方式未知，只作为最后安全后备，不作为正常充电、欠压、输出或温度控制点。第一版当前不增加第二套完整电芯保护 IC；这不削弱板级充电管理、功能性 UVLO 与输出保护的职责。

### 第二层：板级充电与电池功能管理

覆盖专用单节锂离子 CC-CV、Power Path、输入和充电电流限制、终止控制、外部 NTC，以及功能性电池 UVLO、迟滞与延时。

Rev.A primary 使用 BQ25185 集成 BUVLO / BATFET 行为：官方资料给出典型 3.0V falling threshold、约 150mV 恢复迟滞，并在低于阈值时断开 BAT-to-SYS，方向上早于商品标称 2.75V cutoff。architecture-level Alternate 为 BQ24075RGTR + TPS3700 + external battery disconnect；它不是 BQ25185 的 drop-in replacement。切换时必须重新核对 functional battery UVLO、hysteresis、battery disconnect、USB / battery transition、deep-discharge startup、quiescent current、thermal behavior 与 charger control。最终全工况行为 Deferred to Stage 3；UVLO 不得关闭有效 USB 的 System Rail 路径。MCP73871T-2CCI/ML 仅保留为 secondary / cross-vendor fallback。

### 第三层：3.3V 输出保护

保护架构需要独立控制外部负载资格：稳压器有效输出建立前保持负载关闭，输出欠压/过压时断开负载，持续过流时关闭，硬短路时快速限制或关闭。Fault policy 已冻结为 latch-off / controlled recovery，primary 的 orderable MPN 规范为 TPS259474LRPWR（LCSC C2864845）；这是 orderable MPN normalization，不是功能性器件替换。官方资料确认 `474L` 是 persistent over-current 后 circuit-break 并 latch-off 的变体；TPS259470LRPWR 只在外部锁断/控制得到设计和验证时作为 conditional fallback，因为 `470x` 的 persistent-over-current 行为是 active current limiting，不能视为等价 Primary / Alternate。ILIM、timer、UV/OV 和 reset Deferred to Stage 3。

## 外部 NTC 意图与限制

外部 NTC 通过 BQ25185 TS/MR 只控制充电资格，异常不得切断 USB 系统路径。目标为约 10kΩ @25℃、β≈3435K，贴近 pouch 表面，充电窗口目标约 0℃～45℃。BQ25185 官方资料与该 NTC 类型直接匹配并在 TS hot/cold fault 时暂停充电；准确 NTC MPN、安装位置、补偿网络和容差分析 Deferred to Stage 3。

## 3.3V Buck-Boost 决策

TPS63802DLAR（LCSC C2845237）是 primary，理由是其 1.3V～5.5V 输入范围覆盖 1S 路径，官方资料在 VIN≥2.3V、VOUT=3.3V 时给出 2A output-current capability，并提供 100mA→1A 的典型 transient examples。上述资料支持它进入 primary，但不保证本项目 1A/100ms 目标；电感、电容、FB、纹波、瞬态、电流应力、热和 Layout 全部 Deferred to Stage 3。

TPS63070 是 higher-margin fallback。TPS63001 在 3.3V boost mode 的官方额定仅到约 800mA，因此只在 1A peak Design Target 被明确放宽后作为 conditional fallback。

## USB-C 输入与保护决策

- Rev.A 为 5V Sink-only、no USB-PD、no data。TUSB320LAIRWBR（LCSC C132554）是精确 Primary，以 UFP/Sink 角色处理 CC1/CC2，并通过 GPIO OUT1/OUT2 检测 Source current advertisement。
- Native USB-C / C-to-C 的 `Default` advertisement 使用保守 BQ25185 input-current setting，基线约 500mA；attach 时合格检测到 `1.5A` / `3A` 才允许提升至约 1.05A～1.1A input-current-limit 档。3A Source 不表示本 Project 会吸取 3A。
- Legacy USB-A charger + A-to-C cable 按 Type-C Default / conservative policy 处理。Rev.A 不实现 BC1.2 / proprietary charger detection，不承诺利用标签标称的 1.5A / 2A / 3A capability，也不增加 legacy charger-detection IC。
- 接受 TUSB320 GPIO mode 仅基于 attach 时检测结果且无 firmware 动态跟踪后续 Rp capability change 的 Rev.A 限制。Stage 3 必须设计并核对 power-up default、EN / PORT configuration、attach、detach、OUT1/OUT2 decoding、resistor/switch control 与 fail-safe；unknown / invalid / unqualified state 一律回到 conservative input-current behavior。本轮不声称已经完成 schematic verification。
- CC ESD primary 改为 STMicroelectronics USBLC6-2SC6（LCSC C7519），因为 Rev.A CC1 / CC2 不承载 USB2.0 high-speed data，基础 ESD protection 需求与更现实的 JLC/LCSC procurement availability 更匹配。它是功能替换，不是 TPD2E2U06 的 pin-to-pin replacement；pinout、package、connection topology、grounding 与 layout 必须在 Stage 3 依据 ST 官方 datasheet 重新核对。USBLC6-2SC6 不是 CC short-to-VBUS OVP protector；TPD6S300A 只在需要更强 short-to-VBUS robustness 时作为 optional robust alternate。
- VBUS 路径为 `USB-C VBUS → 5V-appropriate TVS/transient protection → BQ25185 IN`；准确 TVS MPN Deferred，不默认叠加 input eFuse。
- USB-C receptacle primary 为 TYPE-C-31-M-12。正式 drawing/spec 显示 16-pin、独立 CC1/CC2、VBUS/GND、D+/D−、SBU1/SBU2、recommended PCB layout 与机械固定结构，并满足当前最大约 1.1A input-current architecture；未进行连接器或 PCB 实物测试。
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
| DN-009 | USB source compatibility 与 TUSB320 initial-only limitation | 决定输入限流控制及故障安全边界 | Policy frozen；Stage 3 hardware decoding/fail-safe verification required |

## 当前证据声明

Stage 2 closeout PASS 仅表示关键架构器件及方向足以进入模块级详细设计，不是 Final BOM。没有详细外围计算、原理图、PCB、EDA、ERC、DRC、仿真、thermal/procurement/manufacturing verification、焊接、Bring-up 或 Test 结果；本文不能用于批准制造，也不能把候选器件描述为已测试。
