# 单节锂电池充电与保护电源板——系统设计与器件选型说明

本文面向第一次接触本项目的工程师、项目评审、项目展示和面试复习，重点解释整块电源板为什么这样设计，以及各模块如何协同。本文描述的是 Rev.A 的设计结论；除非明确标注为实测，否则文中的阈值、能力和行为均属于器件资料、设计计算或工程预期。

## 文档定位 / 职责边界

本文负责说明 system architecture、design rationale、module interaction、关键器件选型理由、Bring-up 思路，以及 design result 与 measured result 的边界。本文不作为正式需求、完整选型证据、详细原理图连接、PCB 制造放行或硬件实测结果的权威记录。

- [requirements.md](../requirements.md)：需求与验收边界；
- [docs/component_selection_plan.md](component_selection_plan.md)：器件选型证据与备选方案；
- [docs/module_design/](module_design/README.md)：模块级电路设计与连接细节；
- [docs/pcb_review.md](pcb_review.md)：PCB、DRC 与制造放行记录；
- 后续 Stage 8 Bring-up / Test 记录：实际硬件测量结果。

## 1. 项目目标与核心设计挑战

本项目是一块面向 STM32、普通传感器及类似低压实验负载的单节锂电池实验电源板。Rev.A 使用普通 5V USB-C 输入和 1S protected Li-Polymer pouch battery，在不依赖 MCU 或 firmware 的前提下，提供充电、边充边用、无电池运行、电池独立供电和受保护的 3.3V 输出。

确定的连续输出需求是 3.3V / 500mA；3.3V / 1A、单次不超过 100ms 仍是必须由 Stage 8 验证的 Design Target。

真正的设计难点不是把 Charger、Regulator 和 Protection IC 连接起来，而是为不同状态建立清楚的责任和优先级：USB 输入先保证系统供电；电池是否能继续放电单独判断；充电还要经过输入能力、温度和系统电源状态的资格判断；外部负载则由独立的输出保护边界决定是否获得电力。

## 2. 整机系统架构与运行模式

### 2.1 主能量流

```text
USB-C → M1 检测/保护 → VBUS_PROT → BQ25185 Charger / Power Path → SYS
                                                    ↕ BAT
                                              1S Battery Pack

SYS → TPS63802 Buck-Boost → 3V3 → TPS259474 eFuse → 3V3_PROT → External Load
```

`SYS` 是 Power Path 管理后的系统电源；`3V3` 是稳压器产生的 raw rail；`3V3_PROT` 才是经 downstream eFuse 资格判断后的外部输出。外部负载不得绕过 M5 使用 raw `3V3`。

### 2.2 三种主要运行模式

```text
USB-only:     USB → VBUS_PROT → BQ25185 → SYS → TPS63802 → TPS259474 → Load
Battery-only: Battery → BATFET / Power Path → SYS → TPS63802 → TPS259474 → Load
USB+Battery:  USB → Power Path → SYS → Load；剩余输入能力 → Battery charging
```

- **USB-only：** 即使电池未连接，合格 USB 输入仍可建立 `SYS`，适合先不接电池进行受控 Bring-up。
- **Battery-only：** 无 USB 时，电池通过 BQ25185 的 BATFET / Power Path 为 `SYS` 供能，前提是电池仍具有放电资格。
- **USB + Battery：** USB 输入优先满足系统负载，剩余能力才用于充电；输入预算不足时应降低或暂停充电，而不是拖垮 `SYS`。

### 2.3 四个资格边界

本系统不存在一个包办所有状态的总 Enable。四个 qualification 分别回答不同问题：

| Qualification boundary | 回答的问题 | 主要实现 |
| --- | --- | --- |
| Power-source / Power-path qualification | 当前 `SYS` 的能量来自 USB 还是 Battery，系统负载如何得到供电？ | BQ25185 Power Path / Load Sharing |
| Battery-discharge qualification | Battery 是否允许继续通过 BATFET / Power Path 向 `SYS` 提供能量？ | BQ25185 BATFET / BUVLO |
| Charging qualification | 当前是否允许给 Battery 充电？ | USB authorization、BQ25185 native TS、External NTC / TLV7041、TPS63802 PG、`CHG_GATE` / CE# |
| External-load qualification | raw `3V3` 是否允许通过 TPS259474 送到 `3V3_PROT` / external load？ | TPS259474 UV/OV、startup、over-current 与 fault policy |

因此，**Battery discharge disabled ≠ USB system supply disabled**：电池因功能性欠压退出时，合格 USB 仍应维持 `SYS`。同样，**`3V3` valid ≠ `3V3_PROT` necessarily enabled**：稳压器输出正常时，M5 仍可能因启动、UV/OV 或负载故障而关闭外部输出。

这四个资格边界是理解后续模块协同和调试顺序的核心。

## 3. 为什么不是简单的 TP4056 + LDO

TP4056 一类器件适合单纯的 1S 电池充电，但 Charger 不等于 Power Path。若把充电器、电池和系统负载简单并联，系统负载电流可能干扰充电终止判断，也难以可靠实现“负载优先、充电退让”和无电池运行。本项目因此选择带真实 `SYS` Power Path 的 BQ25185。

LDO 也无法覆盖 1S 电池跨越 3.3V 的工作区间：电池高于 3.3V 时需要降压，低于 3.3V 时需要升压；Buck-only 和 Boost-only 各自只能覆盖其中一侧。因此 `SYS → 3.3V` 使用 Buck-Boost。

最后，稳压器的 self-protection 主要保护自身功率级，不等于对外部负载建立了确定的故障策略。本项目还要求在输出欠压、过压、持续过流或硬短路时隔离外部负载，并按 latch-oriented policy 恢复，所以在 TPS63802 后增加 TPS259474。

由此，系统并非简单的 Charger + LDO，而是 Power-path-aware charging、宽输入 Buck-Boost 与独立 downstream fault isolation 的组合。

## 4. M1 — USB-C 输入、CC 与输入能力策略

### 4.1 为什么 M1 存在

USB-C Sink 不能只连接 VBUS 和 GND。Source 与 Sink 需要通过 CC1 / CC2 上的 Rp / Rd 关系完成 attach、方向和供电能力识别。Rev.A 使用 `TUSB320LAIRWBR` 固定为 UFP / Sink，负责 CC 检测；USB-PD、USB 数据、Source 和 Dual-Role 均不在范围内。

### 4.2 输入能力策略

TUSB320 在 GPIO mode 下通过 `CC_OUT1 / CC_OUT2` 报告 unattached、Default、1.5A 和 3A advertisement，`SN74LVC2G132DCUR` 将其译码为 `ILIM_HIGH_CMD` 与 `CHG_ENABLE_CMD`。

| Source 状态 | BQ25185 输入策略 | Charging authorization |
| --- | --- | --- |
| Unattached | Conservative | Disabled |
| Default | Conservative，约 500mA input-limit state | Allowed |
| 1.5A advertisement | High-current，约 1.1A input-limit state | Allowed |
| 3A advertisement | High-current，约 1.1A input-limit state | Allowed |

策略可以概括为：Default 对应 conservative input-current state；只有 1.5A / 3A advertisement 才允许进入 high-current input-limit state。**Source advertises 3A ≠ Project sinks 3A**；本板实际吸取电流仍受 BQ25185 input-current limit、系统负载和充电状态约束。

### 4.3 输入保护与边界

Rev.A 的 CC ESD Primary 为 `PESD5V0U2BT,215`；`VBUS_PROT` 使用 Littelfuse `SMF6.0A` TVS；USB shield 通过 1MΩ 与 1nF high-voltage capacitor 的并联支路接 GND。上述属于当前工程实现，不代表已经取得 ESD 或 EMC 实测 PASS。

## 5. M2 — BQ25185 Charger / Power Path

### 5.1 为什么选择 BQ25185

`BQ25185DLHR` 同时提供 1S CC-CV charging、`SYS` Power Path、Load Sharing、输入/充电电流管理、BATFET、BUVLO、native TS 和 batteryless system supply。它解决的不是“把电池充满”这一件事，而是 USB、电池和系统负载如何在不同模式下共享能量。

`IN` 接收受保护的 USB 输入，`BAT` 连接电池，`SYS` 为后级提供独立系统电源。USB 输入能力有限时，Power Path 优先维持 `SYS`，充电电流退让；这样系统负载不会被简单计入电池充电电流，充电终止判断也不会依赖负载恰好很小。

### 5.2 当前关键实现与取舍

- `R1 = 600Ω / 1%` 将默认 fast-charge 目标设为约 500mA。所选 2000mAh 电池商品规格虽给出约 1A / 0.5C 级别充电能力，Rev.A 仍选择较保守的热与输入预算；线性充电损耗可近似理解为 `PLOSS ≈ (VIN - VBAT) × ICHG`，实际温升待 Stage 8 测量。
- ILIM/VSET 使用 `R4 = 13.0kΩ`、`R8 = 4.99kΩ` 和 `Q2 = 2N7002BK,215` 形成两档：Q2 OFF 时约 18kΩ，对应 conservative state；Q2 ON 时 bypass R8，约 13kΩ，对应 high-current state。
- Q1 是最终 charge-enable switch。资格满足时 Q1 可将 BQ25185 `CE#` 拉低；资格失效时 Q1 关断，R3 将 `CE#` 拉高以禁止充电。

### 5.3 Battery-discharge qualification

BQ25185 的 BATFET / BUVLO 负责功能性电池欠压管理：电池电压过低时撤销 Battery-to-SYS 的放电资格。但这一动作不应关闭有效 USB 的 IN-to-SYS 路径。该职责边界正是 **Battery discharge disabled ≠ USB system supply disabled** 的具体实现。

## 6. M3 — Battery / External NTC / Charging Qualification

### 6.1 External NTC 与温度电压

External NTC 使用 `MF52D 103F3435-100`：`10kΩ @25℃`，`B25/85 = 3435K`。它应贴近 pouch 的代表性表面并保持电气绝缘；电池表面温度不等于电芯内部温度，安装位置和 thermal lag 仍需实物验证。

在 BQ25185 TS/MR 偏置下，Rev.A 外部路径可用近似关系理解：`V_NTC_TS ≈ I_TS × (R15 + R_NTC)`。因此 Temperature ↑ → `R_NTC` ↓ → `V_NTC_TS` ↓。

`R15 = 2.7kΩ` 的主要作用之一，是缓解 NTC short 将复用 TS/MR 节点直接硬拉近 GND 所引起的 push-button / long-low / factory-mode interaction。它是 short mitigation，不是 precision temperature calibration resistor。

### 6.2 Native TS 与辅助高温 veto

BQ25185 native TS 仍独立负责其内部温度资格和 NTC 异常处理。`TLV7041DBVR` 是额外的 auxiliary high-temperature charging veto：它比较 `NTC_TS` 与由 `R16 = 90.9kΩ`、`R17 = 10kΩ` 产生的 nominal `VHOT_REF ≈ 0.327V`。温度合格时其 open-drain 输出为 High-Z；温度过高时拉低 `CHG_GATE`，使 Q1 关断并由 R3 拉高 CE#。

温度异常只应禁止 charging，不应切断合格 USB 对 `SYS` 的供电。这再次体现 Charging qualification 与 Power-path qualification 的分离。

### 6.3 CHG_GATE 不是普通 AND gate

```text
CHG_ENABLE_CMD ── HIGH authorization ──→ CHG_GATE → Q1 → BQ25185 CE#
TLV7041 OUT    ── open-drain LOW veto ─┘
TPS63802 PG    ── open-drain LOW veto ─┘

BQ25185 native TS：独立的内部 charging qualification
```

`CHG_ENABLE_CMD` 是唯一 HIGH authorization source；TLV7041 和 TPS63802 PG 只能以 open-drain LOW veto 否决。因此该节点是 shared qualification node，而不是一个可按普通推挽数字逻辑理解的 AND gate。

`CHG_GATE = LOW` 只能说明至少一个 charging qualification 未满足，不能单独判断是 `CHG_ENABLE_CMD` 缺失、TLV7041 veto，还是 TPS63802 PG veto。这个 shared-node diagnostic limit 必须在 Bring-up 时通过其他测点和状态联合定位。

### 6.4 温度数字的证据边界

必须区分三种数字：battery product temperature specification、nominal circuit threshold 和 measured hardware threshold。电池商品规格的充电温度 Design Basis 约为 0℃～45℃；Rev.A 电路的 nominal conservative hardware window 约为 4℃～39℃；后者不是 measured hardware trip range。

电阻、Comparator threshold、NTC/Beta 容差、安装方式和 pouch thermal lag 都会影响实际结果。Actual trip temperatures 必须等待 Stage 8 实测，不能把约 4℃～39℃写成已验证范围。

## 7. M4 — TPS63802 3.3V Buck-Boost

### 7.1 为什么是 Buck-Boost

```text
SYS > 3.3V  → Buck
SYS ≈ 3.3V  → Buck-Boost transition region
SYS < 3.3V  → Boost
```

1S 电池供电时 `SYS` 会跨越 3.3V，因此 LDO 或单向 Buck / Boost 都无法覆盖完整工作区间。`TPS63802DLAR` 能在上述区域连续调节，满足 3.3V / 500mA continuous requirement，并为尚待验证的 1A / 100ms Design Target 提供较合理的能力基础。

### 7.2 当前外围与布局重点

当前电感 baseline 为 `L1 = Murata DFE201612E-R47M=P2`、470nH、LCSC C668312；输入 `C11 = 10µF`，输出 `C14 = C15 = 22µF`；`R20 = 511kΩ` 与 `R22 = 91kΩ` 的 feedback divider 设定 3.3V。

Buck-Boost 的 VIN capacitor、IC、inductor 和 VOUT capacitor 构成高 di/dt 回路，需要紧凑布局；`FB_3V3` 属于 quiet analog node，应远离 L1/L2 switching region，避免把开关噪声耦合为调节误差。

### 7.3 PG 与启动边界

TPS63802 PG 作为 `CHG_GATE` 的 open-drain veto，意图是在 3.3V 系统尚未有效建立时延后 charging。不过，TPS63802 在输入低于自身 UVLO 的最早启动区间具有 datasheet-defined PG `Undefined` 边界，因此不能声称该启动链已经 absolute fail-safe。Stage 8 需要用示波器联合观察 `VBUS / SYS / 3V3 / PG / CHG_ENABLE_CMD / CHG_GATE / CE#`。

Datasheet capability 也不等于本板性能：效率、纹波、瞬态、电感电流、温升和 1A / 100ms 能力都与 PCB、外围和工况有关，尚未实测。

## 8. M5 — TPS259474 Downstream eFuse

### 8.1 为什么稳压器后还要 eFuse

**Regulator self-protection ≠ External load / system-level protection。** TPS63802 首先保证自身功率级能够调节和存活；`TPS259474LRPWR` 则决定 raw `3V3` 是否应继续送给外部负载，并对 external load boundary 建立明确的 fault policy。

Rev.A 的 nominal design basis 包括 UVLO rising ≈ 3.10V、OVLO rising ≈ 3.57V、ILIM ≈ 1A、over-current blanking ≈ 277ms typical，以及 PGTH ≈ 3.0V。它们来自 datasheet-based calculation，不是 hardware measured PASS。

TPS259474 还通过 dV/dt 控制降低外部输入电容造成的 inrush；瞬时过流可经过受控 blanking，持续故障则按 circuit-break / latch-oriented policy 断开，并通过受控方式复位。

### 8.2 Fault-state behavior 是选型条件

保护器件不能只比较 Voltage、Current 和 Package，还必须比较 **Fault-state behavior**。TPS259474 对应本项目需要的 circuit-break / latch-oriented policy；TPS259470 的 persistent-over-current 侧重 active-current-limit behavior，不能作为 TPS259474 的等价替换。只有重新设计并验证 external latch / control 后，TPS259470 才可能成为 conditional fallback。

短路响应、实际 UV/OV、过流计时、热行为及 latch/reset 流程都必须在 Stage 8 验证，不能由 datasheet feature list 直接升级为整板测试结论。

## 9. 三层保护架构与模块协同

| Layer | Protection target | Implementation | Responsibility |
| --- | --- | --- | --- |
| Layer 1 | Battery cell / pack | Battery pack internal protection | 未知阈值的最后安全后备，不作为正常控制点 |
| Layer 2 | Charging / battery path | BQ25185 + native TS + External NTC + TLV7041 | 正常充电、Power Path、电池放电资格与温度管理 |
| Layer 3 | External 3.3V load | TPS259474 | 输出启动资格、外部负载故障隔离和受控恢复 |

三层保护对象不同，因此不是简单重复堆叠保护芯片。Layer 1 面向电池包的最后后备，且其实际阈值、延时和恢复方式尚未独立验证；Layer 2 管理日常充电和电池路径；Layer 3 隔离外部 3.3V 负载故障。任何一层都不能替代另外两层的责任。

## 10. Design for Bring-up / Debug

Rev.A 为主要能量节点和控制节点预留了测试入口：`VBUS_PROT`、`SYS`、`BAT`、`3V3`、`3V3_PROT`、`CHG_GATE`、`CE#`、`STAT1/STAT2`、`NTC_TS`、`M5_RESET` 和 GND。这些测点用于沿能量链与控制链逐级缩小故障范围。

### 10.1 Power-chain diagnosis

```text
VBUS_PROT → SYS → 3V3 → 3V3_PROT
    M1       M2     M4        M5

VBUS_PROT、SYS 正常而 3V3 = 0V      → 优先检查 M4
3V3 正常而 3V3_PROT = 0V            → 优先检查 M5 qualification / fault state
```

对于不能充电的问题，应同时检查 USB attach/capability、`NTC_TS`、TLV7041、TPS63802 PG、`CHG_GATE`、`CE#` 和 `STAT1/STAT2`。尤其要记住 `TP_CHG_GATE` 的 shared-node diagnostic limit：它能确认共享节点为 LOW，却不能独自指出是哪一个来源造成 LOW。

首次 Bring-up 应先做 PCB 外观与装配检查、电池 mating / polarity 实物核对和限流 USB-only 上电，再沿电源链确认各轨。最早 startup transient、PG / CHG_GATE / CE# 时序以及动态故障行为需要示波器，而不是仅靠万用表静态读数。

## 11. 关键器件选型总结

| 器件 | 主要职责 | 选择理由 | 关键取舍 / 待验证 |
| --- | --- | --- | --- |
| `BQ25185DLHR` | Charger + Power Path | 集成 CC-CV、SYS、输入管理、BATFET / BUVLO、TS 与 batteryless operation | 线性充电热、启动和实际 BUVLO 行为待实测 |
| `TUSB320LAIRWBR` | USB-C CC / capability detection | 无 MCU 即可作为 Sink 区分 Default / 1.5A / 3A advertisement | 无 PD / BC1.2；startup 与 capability transition 待验证 |
| `SN74LVC2G132DCUR` | 硬件 capability decode | 无 firmware 生成 ILIM 与 charging authorization 命令 | 依赖 TUSB320 GPIO 状态与启动时序 |
| `PESD5V0U2BT,215` / `SMF6.0A` | CC ESD / VBUS TVS | 分别保护 CC 与 VBUS 输入边界 | 实际 ESD、回流与 clamp 表现未测试 |
| `TPS63802DLAR` | 3.3V Buck-Boost | 覆盖 1S 输入跨越 3.3V 的区域 | 效率、热、纹波、瞬态和 1A / 100ms 待实测 |
| `TPS259474LRPWR` | Downstream eFuse | UV/OV、inrush、over-current、circuit-break 与 latch-oriented policy | 实际阈值、短路、热和 latch/reset 待实测 |
| `TLV7041DBVR` | Auxiliary high-temperature veto | Open-drain 输出可加入 shared `CHG_GATE` | actual trip temperature 与 startup interaction 待实测 |
| `MF52D 103F3435-100` | External battery NTC | 10kΩ @25℃、B3435，可贴近 pouch 表面 | 安装、thermal lag 与 actual trip temperature 待验证 |

器件选择围绕 Power-path behavior、Fault-state behavior、Startup behavior、Thermal behavior 和 Interface behavior 展开，而不是比较单一最大额定值。完整证据和备选方案仍由 `docs/component_selection_plan.md` 维护。

## 12. 当前设计状态与待验证事项

| Item | Current status |
| --- | --- |
| Requirements / architecture | Complete |
| Component selection | Complete |
| Schematic design / review | Complete |
| PCB layout / routing | Complete |
| DRC / Gerber / manufacturing review | Complete |
| Rev.A PCB | Submitted for fabrication |
| Assembly | Not started |
| Bring-up | Not started |
| Hardware validation | Not completed |

Final DRC、Gerber / Drill 和 JLCPCB manufacturing viewer 已完成工程审查，Rev.A 已批准并提交制造。详细 DRC、waiver 和 release record 由 [docs/pcb_review.md](pcb_review.md) 维护，不在本文重复。

当前仍不能声称已经验证以下内容：measured efficiency、1A / 100ms PASS、actual NTC trip temperatures、startup transient PASS、actual UV/OV thresholds、short-circuit behavior、over-current timing、thermal performance、latch/reset behavior，以及 whole-board Bring-up PASS。

Rev.A 的设计与制造释放已经完成，但实际 assembly、首次上电和 hardware validation 尚未开始。Stage 8 的任务是把 datasheet capability、design calculation、engineering assumption 和 Design Target 转换为可重复的 measured hardware evidence，而不是在测量前把设计预期写成 PASS。
