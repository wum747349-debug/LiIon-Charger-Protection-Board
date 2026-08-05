# 资料索引

状态：Stage 1 Baseline — Gate 1.5 Pending

关键硬件参数必须依据当前型号的官方 datasheet、reference manual、application note 或可追溯的供应商正式资料核对。商品页只能作为采购证据，不能单独证明关键参数。

## 商品宣传与采购证据

| Evidence item | 商品标称内容 | Allowed use | Evidence limit | Status |
| --- | --- | --- | --- | --- |
| 当前采购电池包商品信息 | 18650-2P、并联、3.7V、4000mAh | 记录采购身份与待核对声明 | Procurement Evidence / 商品标称；容量与结构未证明 | Not Verified |
| 当前采购电池包商品信息 | 两节 18650、自带保护板、最大放电电流 3A | 建立正式规格和受控实测的核对清单 | 不能证明持续 3A、保护功能、阈值、延时或恢复方式 | Not Verified |
| 当前采购电池包商品信息 | PH2.0-2P、A 序、红黑线、约 10cm | 形成接口与首次接入检查项 | 不能证明实际极性、连接器型号或载流能力；首次接入前必须实测 | Not Verified |
| 当前采购电池包宣传图 | 声称多种保护 | 记录供应商宣传声明 | 不能作为保护类型、阈值、恢复方式或一致性的依据 | Not Verified |

允许的 Working Assumption 仅为：该电池包推测为 1S2P，属于常规 1S 锂离子体系。该假设仍需正式资料或受控核验支持。

## 非当前型号参考规格书

| Document | Relationship to current pack | Permitted use | Prohibited use | Status |
| --- | --- | --- | --- | --- |
| FLY18650-3350mAh 规格书 | 非当前采购电池包型号 | 仅作为“合格规格书应包含哪些参数”的格式和字段参考 | 不得把其中容量、电压、电流、温度、寿命或其他参数用于当前电池包 | Reference Only |

## Stage 2 所需官方 datasheet

当前没有选定器件；下表定义后续候选必须取得的资料类别，不构成器件选择。

| Function | Required official source | Parameters / behavior to verify | Status |
| --- | --- | --- | --- |
| USB-C Sink 与输入保护 | 候选方案官方 datasheet / Type-C 官方规范中适用章节 | CC1/CC2、Sink-only、输入额定、故障与浪涌边界 | Open |
| 单节锂离子 Charger / Power Path | 候选 IC 官方 datasheet、reference design、application note | CC-CV、输入/充电限流、负载优先、终止隔离、无电池/Instant-On、深放电启动、NTC 故障行为 | Open |
| 3.3V Buck-Boost | 候选 IC 官方 datasheet、设计工具或 application note | 输入范围、500mA 连续、1A/100ms 目标、启动、限流、短路、热与布局要求 | Open |
| Functional battery UVLO | 候选监控/开关方案官方 datasheet | 阈值、迟滞、延时、反向路径、USB 与电池路径隔离 | Open |
| 3.3V 输出监控与保护 | 候选方案官方 datasheet | UV/OV、启动禁止、过流/短路、重试/锁断/复位与热行为 | Open |
| External NTC network | 电池包正式温度要求、NTC 与充电方案官方 datasheet | 允许温度窗、阻值、Beta、容差、封装、偏置及开短路行为 | Open |
| 连接器与高电流互连 | 制造商正式图纸和额定规格 | 极性/键位、接触电阻、额定电流、线规与温升 | Open |

## 当前缺失的电池包正式参数

| Missing parameter / evidence | Why required | Resolution plan | Status |
| --- | --- | --- | --- |
| 可追溯的电池包与内部电芯型号 | 确认化学体系与规格适用性 | 向供应商取得正式型号、规格书或批次证据 | Open |
| 标称/最小容量及测试条件 | 功率预算与续航估算 | 取得正式规格；后续按批准条件受控实测 | Open |
| 最大持续/峰值放电电流及热条件 | 500mA 连续与 1A 瞬态边界 | 取得正式规格，不能用商品“3A”直接定版 | Open |
| 充电终止电压、标准/最大充电电流 | 充电安全和寿命 | 取得当前型号正式充电规格 | Open |
| 预充电、终止、再充电要求 | 深放电启动与终止控制 | 取得当前型号正式规格并与候选方案核对 | Open |
| 充电/放电允许温度范围 | 确定外部 NTC 温度窗 | 取得当前型号正式温度规格；0℃～45℃仅为 Working Assumption | Open |
| 包内保护类型、阈值、延时和恢复方式 | 三层保护协调与异常恢复 | 取得保护板规格或可追溯供应商说明；必要时制定受控验证 | Open |
| 循环寿命与条件 | 明确使用和维护边界 | 取得正式寿命规格 | Open |
| PH2.0 实际系列、极性、线规和载流能力 | 防反接与互连热安全 | 查连接器/线材官方规格，并在首次接入前实测极性 | Open |
| 包内是否存在温度检测及其位置 | 避免误解外部 NTC 的保护覆盖 | 取得结构或保护板正式资料 | Open |

## 证据结论限制

- 当前资料不能证明容量、持续 3A、充电参数、保护阈值与恢复方式、温度范围、循环寿命、电芯身份、连接器载流能力或内部温度检测。
- 商品资料、非当前型号规格书和 Framework 文档均不能证明本项目的 EDA 实现或测试状态。
- Stage 2 只有在获得对应候选的官方资料后才能形成器件决策；当前没有最终器件或 BOM 结论。
