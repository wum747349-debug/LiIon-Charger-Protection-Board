# 资料索引

状态：Stage 2 evidence closeout PASS — 2026-08-12；Stage 3 active，M1/M2/M3 evidence index synchronized

关键硬件参数必须依据当前型号的官方 datasheet、reference manual、application note 或可追溯的供应商正式资料核对。商品页只能作为采购证据，不能单独证明关键参数。

## 商品宣传与采购证据

| Evidence item | 商品标称内容 | Allowed use | Evidence limit | Status |
| --- | --- | --- | --- | --- |
| 用户提供的所选电池商品规格 | 1S protected Li-Polymer pouch pack；103450；3.7V；2000mAh；约 7.4Wh | Rev.A Battery Design Basis 与采购身份 | 不是官方电芯制造商 PDF datasheet；容量测试条件与内部电芯身份未验证 | Accepted Design Basis / Not Independently Verified |
| 用户提供的所选电池商品规格 | 4.20V 充电；0.5C（约 1.0A）充电；1C（约 2.0A）放电；2.75V 放电截止 | Stage 2 候选筛选和后续 Stage 3 预算输入 | 商品规格证据；未独立验证测试条件、容差、持续/峰值定义 | Accepted Design Basis / Not Independently Verified |
| 用户提供的所选电池商品规格 | 充电 0℃～45℃；放电 -20℃～60℃ | 外部 NTC 目标窗口与后续热设计输入 | 商品规格证据；不是官方电芯制造商温度规格 | Accepted Design Basis / Not Independently Verified |
| 用户提供的所选电池商品规格 | 约 50 × 34 × 10mm；约 34g；2.54mm-class 两线红/黑连接器 | 机械包络、接口和首次接入检查项 | 准确 mating series、极性、线规和载流能力必须实物核对 | Accepted Design Basis / Physical Verification Pending |
| 当前采购电池包宣传图 | 声称“过充保护” | Procurement Evidence / 商品宣传转录 | 不是官方技术事实；功能、阈值、延时、恢复方式与一致性均未验证 | Not Verified |
| 当前采购电池包宣传图 | 声称“过放保护” | Procurement Evidence / 商品宣传转录 | 不是官方技术事实；功能、阈值、延时、恢复方式与一致性均未验证 | Not Verified |
| 当前采购电池包宣传图 | 声称“过流保护” | Procurement Evidence / 商品宣传转录 | 不是官方技术事实；功能、阈值、延时、恢复方式与一致性均未验证 | Not Verified |
| 当前采购电池包宣传图 | 声称“短路保护” | Procurement Evidence / 商品宣传转录 | 不是官方技术事实；功能、阈值、延时、恢复方式与一致性均未验证 | Not Verified |

上述四种保护类型来自用户提供的当前商品规格，仅作为商品宣称记录；包内保护仍只作为最后安全后备，不作为正常功能控制点。

## 非当前型号参考规格书

| Document | Relationship to current pack | Permitted use | Prohibited use | Status |
| --- | --- | --- | --- | --- |
| FLY18650-3350mAh 规格书 | 非当前采购电池包型号 | 仅作为“合格规格书应包含哪些参数”的格式和字段参考 | 不得把其中容量、电压、电流、温度、寿命或其他参数用于当前电池包 | Reference Only |

## Stage 2 / Stage 3 官方半导体资料

以下均为制造商官方资料。Stage 2 source set 访问日期为 2026-08-11；M1/M2 Stage 3 新增/替换资料访问日期为 2026-08-14。Verified 表示本文所记录的对应功能或参数已在匹配型号的官方资料中核对，不表示 Project 电路或硬件已经验证。

| Function | Manufacturer / device | Official source | Project use | Status |
| --- | --- | --- | --- | --- |
| Charger / Power Path / integrated BUVLO / TS | TI BQ25185DLHR | [BQ25185 datasheet, SLUSF65A, Rev. A](https://www.ti.com/lit/ds/symlink/bq25185.pdf) | Primary；核对 1-cell、Power Path、SYS priority/batteryless behavior、BATFET、4.2V、charge/input current settings、TS 与 BUVLO | Verified for Stage 2 selection; orderable MPN verified |
| High-temperature charge interlock | TI TLV7041DBVR | [TLV7041 datasheet, SLVSE13J, Rev. J](https://www.ti.com/lit/ds/symlink/tlv7041.pdf) | Stage 3 auxiliary support component；核对普通 TLV7041DBVR pinout、open-drain OUT、input/supply/POR behavior；不得与 TLV7041S/L pinout 混淆 | Official technical source indexed; Project circuit/hardware not verified |
| Charger architecture-level alternate | TI BQ24075RGTR | [BQ2407x datasheet, Rev. N](https://www.ti.com/lit/ds/symlink/bq24075.pdf) | 与 TPS3700 + external disconnect 配合；不是 BQ25185 drop-in replacement | Verified for alternate; full architecture requalification required |
| Charger secondary fallback | Microchip MCP73871T-2CCI/ML | [MCP73871 product page and official datasheet DS20002090F](https://www.microchip.com/en-us/product/MCP73871) | Cross-vendor fallback；核对 1-cell、system load sharing / power-path 与 temperature monitoring | Verified for secondary fallback |
| 3.3V Buck-Boost | TI TPS63802DLAR | [TPS63802 datasheet, SLVSEU9D, Rev. D](https://www.ti.com/lit/ds/symlink/tps63802.pdf) | Primary；核对输入/输出范围、3.3V current capability、typical 1A transient examples | Verified for Stage 2 selection; orderable MPN verified |
| Buck-Boost alternate | TI TPS63070 | [TPS63070 datasheet, SLVSC58B, Rev. B](https://www.ti.com/lit/ds/symlink/tps63070.pdf) | Higher-margin fallback | Verified for Stage 2 alternate |
| Buck-Boost conditional fallback | TI TPS63001 | [TPS6300x datasheet, SLVS520C, Rev. C](https://www.ti.com/lit/ds/symlink/tps63001.pdf) | 仅在 1A peak target 放宽后；3.3V boost-mode capability 不与 TPS63802 等价 | Verified limitation |
| Output protection | TI TPS259474LRPWR / TPS259470LRPWR | [TPS25947 datasheet, SLVSFC9C, Rev. C](https://www.ti.com/lit/ds/symlink/tps25947.pdf) | `474L` primary circuit-break/latch-off；`470L` conditional active-current-limit fallback only with external latch/control | Primary orderable MPN verified; alternate is not equivalent |
| USB-C CC/current detection | TI TUSB320LAIRWBR | [TUSB320LAI / TUSB320HAI datasheet, Rev. D](https://www.ti.com/lit/ds/symlink/tusb320lai.pdf) | UFP/Sink Rd 与 Source current-advertisement detection；GPIO initial-only limitation | Verified for Stage 2 selection; exact orderable MPN verified |
| CC ESD | Nexperia PESD5V0U2BT,215 | [PESD5V0U2BT product datasheet](https://assets.nexperia.com/documents/data-sheet/PESD5V0U2BT.pdf) | Current Primary；双线 bidirectional ESD device；M1 current record pin 1 → CC1、pin 2 → CC2、pin 3 → GND、no VBUS | Official device/pin identity verified; Project symbol-pin/footprint/layout not verified |
| M1 logic decode | TI SN74LVC2G132DCUR | [SN74LVC2G132 datasheet, SCES547D, Rev. D](https://www.ti.com/lit/ds/symlink/sn74lvc2g132.pdf) | Dual 2-input NAND with Schmitt-trigger inputs；实现 `NOT(CC_OUT1)` 与 `NAND(CC_OUT1, CC_OUT2)` | Official function/orderable MPN indexed; Project mapping not verified |
| M1 VDD_CC isolation diode | Nexperia BAT54,215 | [BAT54 product datasheet](https://assets.nexperia.com/documents/data-sheet/BAT54.pdf) | Current record pin 1/anode → VBUS_PROT、pin 2 NC、pin 3/cathode → VDD_CC | Official pin identity verified; Project symbol-pin/footprint not verified |
| M2 small-signal MOSFET | Nexperia 2N7002BK,215 | [2N7002BK product datasheet](https://assets.nexperia.com/documents/data-sheet/2N7002BK.pdf) | Q1/Q2 exact MPN baseline；official pin identity 1 G、2 S、3 D | Official device/pin identity verified; Project symbol-pin/footprint not verified |
| Superseded CC ESD selection | STMicroelectronics USBLC6-2SC6 | [USBLC6-2 datasheet, DS4260, Rev. 7](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) | Historical former Primary only；不再描述为 current Primary | Superseded by PESD5V0U2BT,215 |
| Robust Type-C protection | TI TPD6S300A | [TPD6S300A datasheet, SLVSEL8](https://www.ti.com/lit/ds/symlink/tpd6s300a.pdf) | Optional short-to-VBUS / multi-channel ESD robustness | Verified for optional alternate |

## JLCPCB / LCSC point-in-time procurement evidence

以下证据只用于 C-number、orderable MPN、package listing、采购便利性与当前 availability；库存和价格会变化，不是长期 Project Fact，真正采购前必须重新检查。

| Function | Orderable MPN | JLC/LCSC | Package listing | Availability statement | Status |
| --- | --- | --- | --- | --- | --- |
| Charger / Power Path | BQ25185DLHR | C19725033 | WSON-10-EP (2.2mm × 2mm) | Limited | Verified point-in-time mapping; recheck before purchase |
| 3.3V Buck-Boost | TPS63802DLAR | C2845237 | VSON-10 (2mm × 3mm) | Available at evidence check | Verified point-in-time mapping; recheck before purchase |
| Output protection | TPS259474LRPWR | C2864845 | VQFN-10-HR (2mm × 2mm) | Available at evidence check | Verified point-in-time mapping; recheck before purchase |
| USB-C CC controller | TUSB320LAIRWBR | C132554 | X2-QFN-12 (1.6mm × 1.6mm) | Available at evidence check | Verified point-in-time mapping; recheck before purchase |
| USB-C CC ESD (superseded historical selection) | USBLC6-2SC6 | C7519 | SOT-23-6L | Historical point-in-time evidence only | Procurement mapping retained for history; not current Primary |

## TYPE-C-31-M-12 正式 drawing/spec 证据

| Evidence item | Verified project-relevant facts | Allowed use | Evidence boundary | Status |
| --- | --- | --- | --- | --- |
| User-provided [TYPE-C-31-M-12 formal drawing/spec](references/C165948_USB连接器_TYPE-C-31-M-12_规格书_WJ310728%20%281%29.PDF)；HRO Electronics Co., Ltd.，Part No. TYPE-C-31-M-12，drawing date 2020-12-08 | USB Type-C 16-pin receptacle；独立 CC1 / CC2；VBUS / GND；D+ / D−；SBU1 / SBU2；recommended PCB layout；机械固定结构；drawing 标称 5A current rating | Rev.A receptacle Stage 2 Primary selection；确认当前最大约 1.1A input-current architecture 的 connector capability；Stage 3 footprint/mechanical input | Repository 已归档用户提供的 PDF；drawing review 不证明采购批次、焊接、PCB、机械配合或硬件测试通过 | Verified for Stage 2 selection / Locally archived |
| 同一 TYPE-C-31-M-12 formal drawing/spec | drawing 未规定强制 PCB thickness，未发现与 1.6 mm FR-4 的明确机械冲突 | 允许将 1.6 mm FR-4 记录为 Rev.A preliminary PCB thickness baseline | 不是 Final fabrication stackup；层数、铜厚、表面处理、厂家与最终公差未定 | No explicit conflict found / Stage 3 finalization required |

商品页、分销商页、论坛、教程和开源项目未用于证明上述关键参数。官方 datasheet 中的 typical application / typical curve 也不等同于本 Project 已计算或已验证。

External NTC exact MPN 尚未冻结。采购前需取得匹配 orderable MPN 的 manufacturer official evidence，确认 `10kΩ @25℃`、`B≈3435K`、目标容差、工作温度、wired/epoxy construction 与适合 pouch surface attachment 的机械/绝缘条件；LCSC/marketplace 资料只能支持采购身份与 availability，不能替代技术资格证据。

## Stage 2 所需官方 datasheet

本表定义 Stage 2 候选必须取得的资料类别；Battery amendment 完成时尚不构成器件选择。

| Function | Required official source | Parameters / behavior to verify | Status |
| --- | --- | --- | --- |
| USB-C Sink 与输入保护 | TUSB320LAIRWBR / PESD5V0U2BT,215 / TPD6S300A 官方 datasheet；Type-C 适用要求 | CC1/CC2、Sink-only、advertisement detection、ESD、short-to-VBUS option | Current Primary sources indexed；actual mapping/layout 与 VBUS TVS exact MPN OPEN |
| 单节锂离子 Charger / Power Path | BQ25185 官方 datasheet；BQ24075 / MCP73871 official alternate sources | CC-CV、输入/充电限流、负载优先、终止隔离、无电池、深放电启动、NTC 故障行为 | Primary / alternates verified for Stage 2 |
| 3.3V Buck-Boost | TPS63802 / TPS63070 / TPS63001 官方 datasheet | 输入范围、500mA 连续、1A/100ms 目标、纹波、瞬态、功耗、热、电感/开关电流与布局 | Primary / alternate limits verified；calculations Deferred |
| Functional battery UVLO | BQ25185 官方 datasheet | 阈值、迟滞、延时、USB 与电池路径隔离 | Integrated primary verified；full behavior Deferred |
| 3.3V 输出监控与保护 | TPS25947 官方 datasheet | UV/OV、启动禁止、过流/短路、锁断/复位与热行为 | `TPS259474L` primary verified；exact network Deferred |
| External NTC network | 电池商品温度规格与 BQ25185 官方 datasheet | 允许温度窗、阻值、Beta、容差、封装、偏置及开短路行为 | Architecture verified；MPN/network Deferred |
| USB-C receptacle | TYPE-C-31-M-12 formal drawing/spec | 引脚、recommended PCB layout、机械固定、额定能力与板厚冲突检查 | Primary verified for Stage 2；footprint/mechanical finalization Deferred |
| Battery connector 与高电流互连 | 制造商正式图纸、实物和额定规格 | 极性/键位、接触电阻、额定电流、线规与温升 | Physical Verification Pending |

## 当前缺失的电池包正式参数

| Missing parameter / evidence | Why required | Resolution plan | Status |
| --- | --- | --- | --- |
| 可追溯的内部电芯型号与批次 | 确认化学体系与规格适用性 | 向供应商取得正式型号、规格书或批次证据 | Open |
| 标称/最小容量及测试条件 | 功率预算与续航估算 | 商品标称 2000mAh；取得正式测试条件并在后续按批准条件受控实测 | Partially Resolved |
| 最大持续/峰值放电电流及热条件 | 500mA 连续与 1A 瞬态边界 | 商品标称 1C / 约 2.0A；取得正式定义并完成热预算 | Partially Resolved |
| 充电终止电压、标准/最大充电电流 | 充电安全和寿命 | 商品标称 4.20V、0.5C / 约 1.0A；保持证据边界并继续取得正式资料 | Partially Resolved |
| 预充电、终止、再充电要求 | 深放电启动与终止控制 | 取得当前型号正式规格并与候选方案核对 | Open |
| 充电/放电允许温度范围 | 确定外部 NTC 温度窗 | 商品标称充电 0℃～45℃、放电 -20℃～60℃；继续取得正式资料 | Partially Resolved |
| 包内保护类型、阈值、延时和恢复方式 | 三层保护协调与异常恢复 | 取得保护板规格或可追溯供应商说明；必要时制定受控验证 | Open |
| 循环寿命与条件 | 明确使用和维护边界 | 取得正式寿命规格 | Open |
| 2.54mm-class 两线连接器的准确系列、极性、线规和载流能力 | 防反接与互连热安全 | 实物核对 mating series/键位/极性，再查连接器和线材官方规格 | Open |
| 包内是否存在温度检测及其位置 | 避免误解外部 NTC 的保护覆盖 | 取得结构或保护板正式资料 | Open |

## 证据结论限制

- 当前商品规格可作为 Rev.A Design Basis，但不能独立证明容量/电流/温度测试条件、保护阈值与恢复方式、循环寿命、电芯身份、连接器载流能力或内部温度检测。
- 商品资料、非当前型号规格书和 Framework 文档均不能证明本项目的 EDA 实现或测试状态。
- Stage 2 已依据官方制造商资料及用户提供的 TYPE-C-31-M-12 formal drawing/spec 完成当时的 Primary / Alternate 决策；Stage 3 已将真实 selection change `USBLC6-2SC6 → PESD5V0U2BT,215` 及 M1/M2 support-device sources 同步到本索引，不回退或重新开启 Stage 2。
- M1 当前连接与 decode 记录不等于实际 `.SchDoc`、symbol-pin、footprint 或 ERC evidence；`SMF5.0A` 只有 topology/value identity，exact manufacturer/orderable MPN 尚未冻结。
