# Requirements Baseline

Status: Stage 1 Baseline — Gate 1.5 PASS

## Requirement Classification

| Classification | Meaning in this baseline |
| --- | --- |
| Confirmed Requirement | 用户已确认的第一版项目边界；后续设计必须满足，变更需回到需求审查。 |
| Design Target | 需要在后续设计与实测中争取达到的目标，不构成当前保证。 |
| Working Assumption | 为推进架构分析暂时采用的假设，必须由官方资料、设计决策或实测确认，不能升级为已确认需求。 |
| Open Question | 会影响架构、器件、规则或验收的未决事项，保留责任来源和解决计划。 |
| Not Proven / Must Not Claim | 当前证据明确不足，禁止描述为已证明、已保证或已测试通过。 |

## Project Goal

- **Confirmed Requirement:** 建立一块面向 STM32、普通传感器及类似低压实验负载的单节锂离子电池实验电源板。
- **Confirmed Requirement:** 以充电为核心功能，整合普通 5V USB-C 输入、专用单节锂离子 CC-CV 充电管理、真实 Power Path / Load Sharing、无电池运行、三层保护边界及 3.3V 输出。
- **Confirmed Requirement:** 当前项目没有 firmware 需求。

## Out of Scope

- USB-PD、USB 数据功能、Source 或 Dual-Role 功能不在第一版范围内。
- 电机、继电器、加热器、大功率无线模块和大型显示屏等高功率或高脉冲负载不在第一版负载范围内；若新增此类负载，必须返回 Stage 1 重新评估输入、储能、瞬态、热与保护需求。
- 量产认证和无人值守充电不在第一版范围内；本项目定位为有人看护条件下的实验电源板。
- 第一版当前不增加第二套完整电芯保护 IC；电池包内部保护板仅作为未知阈值的最后安全后备。
- Stage 1 不选择最终充电、Power Path、Buck-Boost、保护或 NTC 器件，不固定 NTC 阻值、Beta、封装或偏置电阻。
- Stage 1 不产生原理图、PCB、BOM 定版、制造输出、固件、ERC、DRC、Bring-up 或测试结论。
- 电池包容量、持续放电能力、充电参数、保护阈值与恢复方式、温度范围、循环寿命、电芯身份、连接器载流能力及内部温度检测均不在当前已证明范围。

## Functional Boundary

### USB-C input and source capability

- **Confirmed Requirement:** 输入为普通 5V USB-C、Sink-only，CC1/CC2 必须正确实现 Sink 识别；不需要 USB-PD 或数据连接。
- **Confirmed Requirement:** 不假设任意 Type-C Source 均能提供 2A 或 3A；建议使用 5V/2A 或具有更高 5V 电流能力的适配器。
- **Confirmed Requirement:** 输入能力不足时，系统负载优先，充电必须动态降低或暂停，不得因维持充电而拖垮系统供电。

### Power Path and batteryless operation

- **Confirmed Requirement:** 支持边充边用和真实 Power Path / Load Sharing；USB 接入时优先供给系统负载，剩余输入能力用于充电。
- **Confirmed Requirement:** 系统负载不得明显干扰充电终止判断；禁止普通充电器、电池和负载简单并联。
- **Confirmed Requirement:** 未接 USB 时由电池供电。
- **Confirmed Requirement:** USB 有效时，即使无电池、电池保护板断开或电池达到功能性欠压，系统仍应运行；电池欠压不得关闭 USB 系统供电路径。
- **Confirmed Requirement:** 无电池模式不能依赖电池补充启动或负载峰值。

### Charging and temperature control

- **Confirmed Requirement:** 使用专用单节锂离子 CC-CV 充电管理器，提供充电电压、电流限制、终止控制和可配置充电电流。
- **Confirmed Requirement:** 充电管理必须覆盖 Power Path、系统负载优先、输入电流限制、输入不足时动态降低或暂停充电，以及无电池运行或 Instant-On 等效行为。
- **Confirmed Requirement:** 设计必须考虑深度放电电池接入时的系统启动行为。
- **Working Assumption:** 第一版采用保守、可配置的低速充电策略；默认值与最大值尚未确定。
- **Confirmed Requirement:** 第一版必须有贴近电池包表面的外部 NTC；NTC 开路、短路、未连接或异常时默认禁止充电，但不得切断有效 USB 对系统负载的供电。
- **Working Assumption:** 约低于 0℃或高于 45℃时禁止充电。该范围不是当前电池包的官方规格，必须在获得电池包与所选充电方案的官方依据后重新确认。

## Module Boundary

| Logical module | Responsibility | Explicit boundary |
| --- | --- | --- |
| USB-C 5V input | Sink 识别、输入保护与输入能力边界 | 不含 USB-PD 和数据；不得宣称固定可用 2A/3A |
| Charger / Power Path | CC-CV、输入/充电限流、负载优先、充电退让、终止隔离、无电池运行 | 不允许充电器、电池与负载简单并联 |
| Battery interface | 连接单节锂电池包、外部 NTC 与极性核对 | 包内保护阈值未知；PH2.0 极性必须首次接入前实测 |
| Functional battery management | 功能性 UVLO、迟滞和延时，决定电池是否有放电资格 | 只控制电池路径，不关闭有效 USB 路径 |
| 3.3V regulator | 在三种正常供电模式产生稳定 3.3V | **Working Assumption:** 第一版架构方向为 Buck-Boost，不是器件选择 |
| Output monitoring and protection | 输出启动禁止、欠压、过压、过流和短路保护 | 重试/锁断/复位方式和最终阈值待定 |
| External load interface | 为实验负载提供 3.3V 与回流路径 | 500mA 连续为需求，1A/100ms 为尚未保证的目标 |

## Power Requirements

- **Confirmed Requirement:** USB-only、Battery-only、USB + Battery 三种正常模式均支持 3.3V / 500mA 连续输出。
- **Design Target:** 3.3V / 1A 峰值，单次不超过 100ms；重复周期、允许跌落和过冲尚未确定。
- **Working Assumption:** 第一版稳压架构方向为 Buck-Boost，以覆盖电池端电压跨越 3.3V 的工作区间；这不是最终器件或实现承诺。
- **Working Assumption:** 功能性电池 UVLO 约在 3.1V 关闭电池放电，并约在 3.3V～3.4V 恢复；最终阈值、迟滞、延时、负载条件和精度均待确认。
- **Working Assumption:** 3.3V 输出保护窗口约为 3.3V ±5%；最终阈值与动态响应待确认。
- **Confirmed Requirement:** 持续过流与硬短路必须限制或关闭输出；不接受无限制持续热重启。
- **Open Question:** 1A 峰值的重复周期、允许电压跌落、过冲、负载边沿和恢复时间。

## Interface Requirements

- **Confirmed Requirement:** USB-C 口只作为 5V Sink，CC1/CC2 分别正确处理；D+/D− 无功能需求。
- **Confirmed Requirement:** 电池接口面向商品标称 PH2.0-2P、A 序、红黑线、约 10cm 的电池包，但“A 序”不能替代极性实测；第一次连接前必须确认连接器机械键位、线序和正负极。
- **Confirmed Requirement:** 外部 NTC 必须靠近电池包热中心；对于当前推测的两节并联电芯结构，建议安装在两节电芯之间附近的热缩膜表面，并连接到 Charger Temperature Input。
- **Confirmed Requirement:** NTC 安装必须保持与电芯及导体电气绝缘，不得刺穿、割伤、过度挤压或以其他方式损伤电池包热缩膜；可使用独立两线连接器，以便装配、故障检测和更换。
- **Not Proven / Must Not Claim:** NTC 测得的电池包表面温度不等于电芯内部温度，建议安装位置也不能替代当前电池包正式结构与温度规格。
- **Confirmed Requirement:** 3.3V 输出接口必须支持规定的连续负载，并提供适合调试的地与关键测试接入。
- **Open Question:** 接口具体位置、连接器系列/额定能力、测试点形式及机械固定方式。

## Safety Boundary

### Layer 1 — battery-pack internal protection

- **Confirmed Requirement:** 商品声称的包内保护板仅作为未知阈值的最后安全后备，不作为正常充电、功能性欠压、输出或温度控制点。
- **Confirmed Requirement:** 第一版当前不增加第二套完整电芯保护 IC。
- **Not Proven / Must Not Claim:** 包内保护的实际功能、阈值、延时、恢复方式和一致性尚未由正式资料或实测证明。

### Layer 2 — board-level charging and battery functional management

- **Confirmed Requirement:** 覆盖 CC-CV、Power Path、输入和充电电流限制、充电终止、外部 NTC，以及带迟滞和延时的功能性电池 UVLO。
- **Confirmed Requirement:** 电池功能性欠压只撤销电池放电资格，不得关闭有效 USB 的系统供电路径。
- **Confirmed Requirement:** NTC 异常默认禁止充电，但不得切断有效 USB 的系统供电。
- **Not Proven / Must Not Claim:** 外部 NTC 不替代 CC-CV、过流、短路或过充保护；其表面读数不能声称等同电芯内部温度。

### Layer 3 — 3.3V output protection

- **Confirmed Requirement:** 检测到输出欠压或过压时，必须禁止或断开外部负载。
- **Confirmed Requirement:** 3.3V 稳压器启动完成并进入有效输出状态前，外部负载必须保持关闭。
- **Confirmed Requirement:** 持续过流必须关闭外部负载；硬短路必须快速限制或关闭，禁止无限制持续热重启。
- **Open Question:** 欠压/过压、启动完成、过流和短路的最终阈值、响应时间，以及重试、锁断和复位恢复策略。

### Evidence and handling limits

- **Confirmed Requirement:** 高电流路径必须在后续阶段考虑铜宽、过孔、电压降和散热；电池接口必须降低反接风险。
- **Confirmed Requirement:** 首次接入电池前实测 PH2.0 极性；首次上电与边界测试计划需包含限流、测点和异常断电策略。
- **Not Proven / Must Not Claim:** 当前没有 EDA、ERC、DRC、制造、焊接、Bring-up 或 Test 证据，不得批准制造或描述为已验证。

## Manufacturing Baseline

- **Confirmed Requirement:** 第一版优先手工焊接、调试便利性和可测性。
- **Confirmed Requirement:** 高电流路径后续必须按实际电流、铜厚、温升和允许压降核对铜宽、过孔与散热。
- **Confirmed Requirement:** 电池接口设计需降低极性误接风险；PH2.0 极性在首次接入前必须实测，不能只依赖商品所称“A 序”。
- **Open Question:** 板框、接口位置、层数、铜厚、板厚、表面处理、PCB 厂家及装配方式。
- **Not Proven / Must Not Claim:** 尚未形成 PCB 制造规则或制造输出，当前不能声称可制造或已完成装配验证。

## Acceptance Criteria

以下均是未来验证条目，不是当前测试结果。

| ID | Requirement | Condition | Verification Method | Expected Result | Status |
| --- | --- | --- | --- | --- | --- |
| AC-001 | USB-only 模式连续提供 3.3V/500mA | 5V/2A 能力的输入；无电池；明确线缆、环境温度与持续时间 | 实测输入、System Rail、3.3V 输出和温升 | 能启动并持续供电，不依赖电池补峰；输出位于最终批准窗口内 | Planned |
| AC-002 | Battery-only 模式连续提供 3.3V/500mA | 电池接近最终批准的功能性最低电压；明确电池状态、环境温度与持续时间 | 测量 3.3V 输出、温升和功能性 UVLO 行为 | 允许工作区间内持续供电；达到功能性 UVLO 后受控关闭电池路径 | Planned |
| AC-003 | USB + Battery 模式以负载优先方式连续提供 3.3V/500mA | 逐步限制 USB 可用输入能力，同时电池已连接并允许充电 | 同步测量输入、系统负载、充电电流、3.3V 输出和终止行为 | 输入不足时充电降低或暂停；输出不被充电拖垮，负载不明显干扰终止判断 | Planned |
| AC-004 | 评估 3.3V/1A、单次不超过 100ms 的瞬态目标 | 任一适用正常供电模式；采用最终批准的边沿、重复周期和测量带宽 | 捕获输出跌落、过冲与恢复波形 | 达到最终批准的瞬态窗口；当前重复周期、跌落和过冲限制仍为 Open | Open |
| AC-005 | USB 有效时支持无电池启动和运行 | 无电池或电池保护板断开；采用最终批准的启动负载 | 重复冷启动与热插拔并监测 System Rail 和 3.3V | 系统能够启动并维持规定负载，不依赖电池补充峰值 | Planned |
| AC-006 | 电池功能性欠压不得关闭有效 USB 路径 | 电池处于功能性欠压，同时 USB 有效 | 降低电池电压并观察电池资格、System Rail 和 3.3V | 电池放电路径被禁止或隔离，USB 继续维持系统输出 | Planned |
| AC-007 | NTC 异常只禁止充电，不中断 USB 系统供电 | NTC 开路、短路、未连接或超出最终批准温度窗口；USB 有效 | 故障注入并测量充电电流、System Rail 和 3.3V | 充电被禁止，USB 对系统负载的供电保持有效 | Planned |
| AC-008 | 输出保护对持续过流与硬短路进入受控状态 | 采用最终批准的故障幅度、热条件和持续时间 | 受控故障注入并捕获输出电流、电压、保护响应和温升 | 持续过流关闭；硬短路快速限制或关闭；不得无限制持续热重启 | Planned |
| AC-009 | 输出保护在无效输出和启动期间隔离外部负载 | 输出欠压、过压，或稳压器尚未完成启动 | 注入 UV/OV 条件并记录启动、负载开关和输出波形 | UV/OV 时负载被禁止或断开；有效输出建立前负载保持关闭 | Planned |

## Open Questions

| ID | Question | Category | Impact | Must Resolve By | Owner / Source | Resolution Plan | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| OPEN-001 | 当前电池包最终允许的充电终止电压是多少？ | Battery / Charging | 充电安全、容量与寿命 | Stage 2 充电方案筛选前 | 电池包/电芯正式规格书或供应商可追溯声明 | 获取并核对当前型号正式资料；缺失时不得定版充电方案 | Open |
| OPEN-002 | 默认与最大充电电流是多少？ | Charging / Power Budget | 输入预算、充电时间、热与电池安全 | Stage 2 充电方案筛选前 | 电池包官方充电规格、适配器能力与热分析 | 建立各模式功率预算后确认；不预设 300mA 或 500mA | Open |
| OPEN-003 | 预充电、终止和再充电阈值及时序是什么？ | Charging State Machine | 深度放电启动、终止可靠性与寿命 | Stage 2 充电方案定版前 | 电池官方要求及候选方案官方 datasheet | 用官方资料形成可追溯的阈值与时序比较 | Open |
| OPEN-004 | NTC 的正式温度窗口、阻值、Beta、封装和偏置网络是什么？ | Temperature Safety | 温度保护准确性和故障安全 | Stage 2 充电方案定版前 | 电池官方温度要求、NTC 与候选方案官方 datasheet | 先确认电池允许范围，再完成网络与容差分析 | Open |
| OPEN-005 | 1A/100ms 峰值的重复周期、负载边沿、允许瞬态跌落/过冲和恢复时间是什么？ | Output Transient | 稳压架构、储能、环路和验收波形 | Stage 2 稳压方案筛选前 | 目标负载波形及用户确认 | 采集代表性负载需求并冻结可测波形与限值 | Open |
| OPEN-006 | 功能性电池 UVLO 的最终关闭/恢复阈值、迟滞、延时和负载条件是什么？ | Battery Functional Protection | 可用容量、误动作和包内保护协调 | Stage 2 电池路径方案定版前 | 电池放电规格及系统功率预算 | 获得正式电池证据并完成阈值与容差分配 | Open |
| OPEN-007 | 3.3V 输出保护的最终阈值、响应和重试/锁断/复位恢复策略是什么？ | Output Protection | 负载安全、故障响应和热行为 | Stage 2 输出保护方案筛选前 | 负载边界、候选方案官方资料与风险审查 | 冻结故障状态机、阈值和最大响应时间 | Open |
| OPEN-008 | 板框、接口位置、层数、铜厚、板厚、表面处理、PCB 厂家和装配方式是什么？ | Manufacturing / Mechanical | PCB 规则、热、成本和可制造性 | Stage 5 Layout Preflight 前 | 用户与目标板厂官方能力 | 冻结制造机械基线，并评估是否触发返回 Stage 1 | Open |
| OPEN-009 | 商品电池包的真实 1S2P 结构、容量、放电能力、连接器额定能力和包内保护行为是什么？ | Battery Evidence | 功率预算、保护协调、接口与验收 | Stage 2 相关方案定版前 | 当前型号正式规格书、供应商证据及后续受控实测 | 商品信息仅作采购证据；获得正式资料后逐项核验 | Open |
| OPEN-010 | 三种供电模式及关键负载点允许的 3.3V 输出纹波是多少？ | Output Quality | 输出滤波、环路、布局和验收带宽 | Stage 2 稳压方案筛选前 | 目标负载需求与测量方法 | 冻结纹波限值、带宽、探测方式和测试工况 | Open |
| OPEN-011 | USB-only、Battery-only、USB + Battery 及关闭状态允许的静态功耗和关断功耗是多少？ | Power Consumption | 电池续航、无负载温升和关断行为 | Stage 2 方案筛选前 | 用户使用场景与系统功率预算 | 定义各模式电流/功耗上限及测量边界 | Open |
| OPEN-012 | 连续负载、充电叠加和故障工况的环境温度、器件温升及降额热边界是什么？ | Thermal | 连续能力、保护阈值、可靠性和 PCB 散热 | Stage 2 关键器件定版前 | 使用环境、官方 datasheet 和热预算 | 冻结环境与温升限制，建立最坏工况热预算 | Open |
| OPEN-013 | 电感饱和/RMS 电流和稳压开关峰值/RMS 电流需要保留多少余量？ | Power Stage Margin | 瞬态能力、效率、热和器件可靠性 | Stage 2 稳压方案定版前 | 候选方案官方 datasheet、设计计算与降额规则 | 基于最坏输入、负载、容差和瞬态完成电流应力与降额核对 | Open |
