# Stage 2 — Critical Component Selection Plan

Status: Stage 2 documentation closeout PASS — 2026-08-12；Stage 3 entry authorized

本文件记录影响架构、外围、封装、布局、散热和采购的关键器件决策。`Primary` 表示当前主选方向，`Alternate` / `Conditional fallback` 表示备选；均不是 Final BOM，也不表示已经完成 EDA、计算、仿真或硬件验证。

## Selection Summary

| Category | Primary / architecture | Alternate / fallback | Stage 2 status |
| --- | --- | --- | --- |
| Battery pack | 1S protected Li-Polymer pouch pack, 103450, 3.7V / 2000mAh | TBD only if requirements are reopened | Design Basis accepted; product-spec evidence only |
| Charger / Power Path | TI BQ25185DLHR (LCSC C19725033) | BQ24075RGTR + external functional UVLO / battery disconnect; MCP73871T-2CCI/ML secondary fallback | Primary selected; current availability Limited |
| Functional battery UVLO | BQ25185 integrated BUVLO / BATFET behavior | TPS3700 + external disconnect with BQ24075 architecture-level Alternate | Primary architecture selected |
| 3.3V Buck-Boost | TI TPS63802DLAR (LCSC C2845237) | TI TPS63070; TPS63001 only if 1A peak target is relaxed | Primary selected |
| External battery temperature | External NTC near pouch surface, about 10kΩ @25℃, β about 3435K | Different NTC only with compensation analysis | Architecture selected; MPN deferred |
| Output protection | TI TPS259474LRPWR (LCSC C2864845) | TPS259470LRPWR only with external latch/control qualification; not equivalent | Primary orderable MPN normalized |
| USB-C CC/current detection | TI TUSB320LAIRWBR (LCSC C132554) in UFP/Sink role | TBD | Exact Primary selected; implementation deferred |
| CC ESD | STMicroelectronics USBLC6-2SC6 (LCSC C7519) | TI TPD6S300A for stronger short-to-VBUS robustness | Primary functionally replaced; Stage 3 pinout/topology/layout review required |
| USB VBUS protection | 5V-appropriate TVS/transient protection before BQ25185 IN | Exact MPN TBD | Architecture selected; MPN deferred |
| USB-C receptacle | TYPE-C-31-M-12 | Reopen selection only if Stage 3 mechanical/footprint review finds a conflict | Primary selected; exact receptacle resolved for Stage 2 |
| Battery connector | 2.54mm-class, 2-wire red/black interface | Exact series TBD | Physical verification required |

## 1. Charger / Power Path

### Primary — TI BQ25185DLHR (LCSC C19725033)

- **Selection rationale:** BQ25185 仍最符合当前 Rev.A architecture；它集成单节电池线性 Charger、Power Path、SYS load priority、输入电流管理/DPPM、外部 TS/NTC、BATFET、无电池/低电池时由输入维持 SYS，以及 battery BUVLO，与 1S 4.20V、默认约 500mA 的 Rev.A 方向匹配，外围和状态机复杂度低于分立 Power Path。
- **Key verified parameters:** 官方 datasheet 支持 1-cell 应用；典型充电调节范围 3.6V～4.4V，包含 4.2V 设置；典型充电电流范围 5mA～1000mA；可选输入限流包含约 100mA、500mA、1050mA；Power Path 在 battery empty/charging 时可由 IN 维持 SYS，并在输入受限时优先系统负载、降低充电电流；TS 以 10kΩ @25℃、β(25/85)=3435K 为直接目标；集成 BUVLO 在 VBAT falling 典型 3.0V、恢复迟滞典型约 150mV，低于阈值时断开 BAT-to-SYS。
- **Risks:** 线性充电器的最坏工况功耗/温升尚未预算；2mm × 2mm WSON/thermal pad 对手工焊接和散热布局有挑战；固定 BUVLO 与商品标称 2.75V cutoff 的协调、恢复状态、无电池启动负载及漏电边界仍需设计分析和实测。
- **Procurement evidence:** LCSC 将 BQ25185DLHR 对应到 C19725033。当前 availability 仅按 point-in-time evidence 记录为 `Limited`，不把具体库存数量固化为长期 Project Fact；真正采购前必须重新检查 availability。
- **Pending verification:** 确认封装可制造性；核对所有 TS open/short/disconnected 情形与 USB-valid SYS continuity；核对 100/500/1050mA input-limit switching 的允许实现和上电默认状态。
- **Deferred to Stage 3:** ISET、ILIM/VSET、输入/输出/电池电容、TS 网络、CE/状态引脚、热预算、启动/终止/再充电、详细 BUVLO 工况与布局计算。

### Alternates

- **BQ24075RGTR + external functional UVLO / battery disconnect:** architecture-level Alternate，外部 UVLO 沿用 TPS3700 + external disconnect 思路。BQ24075 不是 BQ25185 的 drop-in replacement；切换必须重新核对 functional battery UVLO、hysteresis、battery disconnect、USB / battery transition、deep-discharge startup、quiescent current、thermal behavior 与 charger control。
- **MCP73871T-2CCI/ML:** secondary / cross-vendor fallback。官方资料确认 1-cell、integrated system load sharing / power-path management、可同时供系统和充电、温度监控；输入选择、热行为、状态机和外围与 BQ25185 不同。

## 2. Functional Battery UVLO

- **Primary:** 使用 BQ25185 集成 BUVLO / BATFET 行为。官方 datasheet 明确低于 BUVLO 时 BAT-to-SYS 断开，典型 falling threshold 3.0V、典型恢复迟滞约 150mV，并有约 60µs deglitch；该点早于商品标称 2.75V 放电截止，符合“内部保护只作最后后备”的方向。
- **Alternate architecture:** BQ24075RGTR + TPS3700 + external battery disconnect。该方案不与 BQ25185 pin-compatible 或 drop-in equivalent；只有切换 Alternate 或后续证据证明集成方案不足时才进入详细设计。
- **Pending / Stage 3:** 核对最坏阈值容差、负载压降、恢复条件、深放电启动、USB 插拔转换和 battery-only quiescent current；本阶段不设计外部 UVLO 电路。

## 3. 3.3V Buck-Boost

### Primary — TI TPS63802DLAR (LCSC C2845237)

- **Selection rationale:** 对 1S 电池跨越 3.3V 的输入范围提供真正 Buck-Boost；相比 TPS63001 对当前 1A/100ms Design Target 有更明确的电流余量。
- **Key verified parameters:** 官方 datasheet 给出 1.3V～5.5V 输入、启动需高于 1.8V、1.8V～5.2V 可调输出；在 VIN ≥2.3V、VOUT=3.3V 时标称 2A output-current capability；官方典型曲线包含 100mA→1A 的 buck、buck-boost 与 boost transient examples。
- **Risks:** 官方额定和典型波形不等于本项目在全部输入、温度、容差和 PCB 条件下已经保证 1A/100ms；小型 DFN、0.47µH power stage 和高 di/dt 布局需要严格实现。
- **Pending verification:** 采购/封装、最坏 battery/System Rail 输入范围、500mA 连续效率/热和 1A pulse 波形边界。
- **Deferred to Stage 3:** 电感、输入/输出电容、FB divider、MODE/EN/PG、纹波、瞬态、饱和/RMS 电流、开关应力、热与 Layout 计算。

### Alternates

- **TPS63070:** 官方资料确认 2V～16V 输入、2.5V～9V 输出、3.6A switch current 和最高约 2A output-current capability（依工况）；作为更高面积/静态功耗但更高裕量的 fallback。
- **TPS63001:** 官方资料给出 3.3V boost mode 在 VIN>2.4V 时最高约 800mA，而 step-down mode 可到 1.2A。它不能作为当前 1A peak target 的等价替代；仅在该目标被明确放宽并重新核对 requirements 后保留为 conditional fallback。

## 4. External Battery Temperature Sensing

- **Primary architecture:** 外部 NTC 紧贴 pouch 电池代表性表面位置，目标约 10kΩ @25℃、β≈3435K；充电窗口目标约 0℃～45℃。
- **Verified fit:** BQ25185 官方 TS 功能按 10kΩ @25℃、β(25/85)=3435K 配置；TS hot/cold fault 会暂停充电与 safety timer，TS fault 时 trickle charge 也被禁止。
- **Risk / pending:** 商品温度范围不是官方电芯 datasheet；NTC 表面温度不等于内部温度；必须确保 open/short/disconnected/abnormal 均禁止充电且不切断有效 USB 的 SYS 路径。
- **Deferred to Stage 3:** NTC MPN、容差、封装、粘贴/绝缘方式、补偿电阻网络与温度窗口容差分析。

## 5. 3.3V Output Protection

### Primary — TI TPS259474LRPWR (LCSC C2864845)

`TPS259474LRPW` 到 `TPS259474LRPWR` 是 orderable MPN normalization，不是功能性器件替换。

- **Selection rationale:** 项目 fault policy 是 latch-off / controlled recovery。官方 datasheet 将 TPS259474x 定义为 circuit-breaker variant：超过 ILIM 的持续过流在可调 blanking interval 后关断；`L` variant 在该 fault 后保持 latched off，符合“不允许无限热自动重启”。
- **Key verified parameters:** TPS25947 family 的工作输入范围 2.7V～23V、集成 back-to-back FET、reverse-current blocking、可调 ILIM/ITIMER/UVLO/OVLO；RPW 为 10-pin VQFN-HR。
- **Risks:** 该器件能力远高于 3.3V/500mA 应用，低端 ILIM 容差、短路响应、SOA、压降和 reset method 必须核对；小封装焊接与热路径仍是风险。
- **Pending / Deferred to Stage 3:** ILIM、ITIMER、UV/OV、dVdt、reset/enable、PG、故障注入工况、热与 Layout。

### Conditional fallback — TI TPS259470LRPWR

官方资料把 TPS259470x 定义为 active-current-limit variant，而不是 TPS259474x 的 circuit-breaker variant；`L` suffix 的 thermal/fault latch behavior 不能把 persistent over-current current limiting 自动变成等价 circuit-break latch-off。它不是等价 Primary / Alternate；只有加入并验证外部 FLT/EN 锁断与受控 reset 方案后才可作为 conditional fallback，不能直接替代 primary。

## 6. USB-C Input and CC Current Detection

- **Architecture:** USB Type-C、5V Sink-only、no PD、no data；CC1/CC2 必须正确处理，不假设任意 source 可供 2A/3A。
- **Primary controller — TUSB320LAIRWBR (LCSC C132554):** 官方 datasheet 支持固定 UFP mode，在 CC1/CC2 呈现 Rd，并在 attach 后通过 GPIO OUT1/OUT2 或 I2C 报告 Source 的 Default / 1.5A / 3A advertisement。
- **Native USB-C / C-to-C policy:** `Default` advertisement 使用保守的 BQ25185 input-current setting，基线约 500mA；只有 attach 时合格检测到 `1.5A` 或 `3A` advertisement，才允许提升至 BQ25185 的约 1.05A～1.1A input-current-limit 档。3A Source 仅表示能力声明，本 Project 不吸取 3A，仍受 BQ25185 input limit 约束。
- **Legacy USB-A / A-to-C policy:** Rev.A 不实现 BC1.2 或 proprietary charger detection，不承诺利用 legacy charger 标签标称的 1.5A / 2A / 3A capability，并按 Type-C Default / conservative input policy 处理；不增加 legacy charger-detection IC。
- **Accepted Rev.A limitation:** TUSB320 GPIO mode 以 attach 时检测到的 Source current advertisement 为基础；当前无 firmware，不动态跟踪 Source 后续 Rp capability change。绑定 datasheet 支持在 GPIO mode 解码 attach 结果，且可用保守硬件默认实现合理 fail-safe，因此该限制不阻断 Stage 2 closeout。
- **Mandatory Stage 3 verification:** 设计并核对 power-up default、EN / PORT configuration、attach、detach、OUT1 / OUT2 decoding、resistor / switch control 和 fail-safe behavior。任何 unknown、invalid 或 unqualified state 必须回到 conservative input-current behavior，不得默认进入高电流档。本轮没有完成或声称 schematic verification。

## 7. USB-C CC Protection

- **Primary — STMicroelectronics USBLC6-2SC6 (LCSC C7519):** Rev.A CC1 / CC2 不承担 USB2.0 high-speed data；ST 官方资料确认该器件提供双线基础 ESD protection，且其 JLC/LCSC procurement availability 明显更现实，因此替换 TPD2E2U06 作为当前 Primary。USBLC6-2SC6 不是 CC short-to-VBUS OVP protector。
- **Replacement boundary:** 这是功能替换，不是 pin-to-pin replacement。pinout、package、connection topology、grounding 与 layout 必须在 Stage 3 schematic / layout 中依据 ST 官方 datasheet 重新核对。
- **Optional robust alternate — TPD6S300A:** 官方资料提供 CC/SBU 等通道的 24V short-to-VBUS protection 与多通道 IEC ESD；仅在接口误短接鲁棒性、空间和成本评估支持时采用，不是 Rev.A 强制项。
- **Deferred to Stage 3 / layout:** USBLC6-2SC6 的 pin mapping、连接拓扑、器件摆放、回流/接地、CC trace stub 与是否升级为 TPD6S300A。

## 8. USB VBUS Protection

- **Selected architecture:** `USB-C VBUS → 5V-appropriate TVS/transient protection → BQ25185 IN`。
- **Decision:** Stage 2 不增加另一颗 input eFuse；目前 BQ25185 官方资料未显示本项目必须叠加 eFuse 的明确理由。
- **Deferred:** 精确 TVS MPN、VRWM、clamping、surge/ESD target、connector-to-TVS placement 与回流路径在 Stage 3 / layout-related evaluation 确定。

## 9. Connectors

### USB-C receptacle

- **Primary — TYPE-C-31-M-12:** Rev.A exact receptacle 已为 Stage 2 解决。正式 drawing/spec 显示 USB Type-C 16-pin receptacle，提供独立 CC1 / CC2、VBUS / GND、D+ / D−、SBU1 / SBU2、recommended PCB layout 与机械固定结构；其额定能力满足当前最大约 1.1A input-current architecture。该结论是 selection/document review，不是连接器或 PCB 实物测试结果。
- **PCB thickness baseline:** 1.6 mm FR-4 记录为 `Rev.A preliminary PCB thickness baseline`。已核对 drawing 未发现与 1.6 mm 的明确机械冲突；它不是 Final fabrication stackup，层数、铜厚、表面处理、厂家和最终机械公差仍在后续阶段确定。
- Stage 3 仍须完成 symbol/pin mapping、recommended footprint 转录核对、板边/外壳配合和装配/返修风险审查；不得把 drawing review 写成 manufacturability verified。

### Battery connector

- 当前接口为 2.54mm-class、2-wire、red/black。
- 准确 series、mating housing/header、键位、极性、线规和载流能力均为 Pending physical verification；首次电池连接前必须逐项核对并测量极性。
- 不使用 PH2.0 作为当前 Project fact。

## Stage 2 Closeout Disposition

- TYPE-C-31-M-12 exact receptacle：**Resolved for Stage 2**。
- Battery connector exact series / polarity / wire gauge：**Physical Verification Pending**；首次连接前必须实物核对，不得推断。
- VBUS TVS exact MPN：**Deferred to Stage 3**。
- BQ25185、TPS63802、TPS259474、NTC 与 TUSB320 的详细外围、参数、状态机、热、瞬态和 fail-safe 实现：**Deferred to Stage 3**。
- 1A / 100ms output target 的 waveform、droop、overshoot、repetition 与 recovery definition：继续是 **Design Target**，作为 Stage 3 design/test-definition input，未验证。
- Battery product-spec evidence、pack internal protection thresholds/recovery 与 battery connector polarity：保持 **Not Verified / Physical Verification Pending**。
- 小型 leadless package 的 assembly / rework 风险：保留为后续制造设计输入；没有 manufacturability verified 结论。

Stage 2 closeout 仅表示关键架构器件及方向已经足以进入模块级详细设计；不表示 Final BOM、EDA、thermal verification、procurement verification 或 hardware validation 完成。

## Explicit Stage 3 Deferred Work

- BQ25185 ISET / ILIM-VSET / capacitors / TS / state-machine / thermal / layout calculations。
- TPS63802 inductor / capacitors / FB / ripple / transient / current-stress / thermal calculations。
- NTC exact MPN、compensation network 和 tolerance analysis。
- TPS259474L ILIM / ITIMER / UV-OV / dVdt / reset implementation。
- TUSB320 power-up default、attach/detach、OUT1/OUT2 decoding、GPIO/resistor switching 与 conservative fail-safe behavior。
- USB VBUS TVS exact MPN、battery connector physical verification 和 TYPE-C-31-M-12 symbol/footprint/mechanical finalization。
- 任何 schematic capture、BOM draft、EDA、ERC、DRC、simulation、manufacturing、bring-up 或 test activity。
