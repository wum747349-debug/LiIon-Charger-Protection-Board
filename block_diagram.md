# 系统框图

状态：Stage 1 Baseline — Gate 1.5 PASS；Rev.A Stage 3 implementation sync

## 系统能量流

```text
[USB-C] → [VBUS_PROT] → [BQ25185 Charger / Power Path] → [SYS]
                                                         |
                                                         v
                    [External Output] ← [3V3_PROT] ← [TPS259474] ← [3V3] ← [TPS63802]

[BQ25185 BAT] ↔ [J_BAT1 / 1S Battery Pack]
[J_NTC1] → [R15] → [NTC_TS]
```

## 运行与控制关系

```text
USB 有效 -------------------------------> 维持 System Rail（允许无电池运行）
系统负载需求 ---------------------------> Power Path 优先分配输入能力
输入能力不足 ---------------------------> 降低或暂停充电
Functional Battery UVLO ---------------> 只控制电池放电资格
电池功能性欠压 + USB 有效 -------------> USB 路径保持开启
External NTC 异常 ----------------------> 禁止充电，不关闭 USB 系统路径
TUSB320LAIRWBR -------------------------> SN74LVC2G132 → ILIM_HIGH_CMD / CHG_ENABLE_CMD
NTC_TS + VHOT_REF ---------------------> TLV7041 → CHG_GATE
CHG_ENABLE_CMD + TLV7041 + TPS63802 PG -> CHG_GATE → Q1 → CE_N / BQ25185 CE#
稳压器尚未完成启动 --------------------> 外部负载保持关闭
输出欠压 / 过压 -----------------------> 禁止或断开外部负载
持续过流 ------------------------------> 关闭外部负载
硬短路 --------------------------------> 快速限制或关闭
```

## 模块边界

| 模块 | 输入 | 输出 / 控制 | Stage 1 边界 |
| --- | --- | --- | --- |
| USB-C 与 CC | 外部普通 5V Type-C Source | 经识别与保护的输入 | Sink-only，无 PD、无数据，不假设 2A/3A 恒可用 |
| 输入保护与能力限制 | USB-C 5V、CC 识别结果 | 受控输入能力 | 保护细节和阈值待后续官方资料决定 |
| Charger / Power Path / System Rail | 受控 USB 输入、电池；独立 Charger Temperature Input 接收外部 NTC | System Rail、受控充电、电池路径 | 负载优先、充电退让、终止隔离、无电池运行 |
| Battery Pack | 1S 103450 protected Li-Polymer pouch pack | 双向充放电能量 | 3.7V / 2000mAh 商品规格作为 Design Basis；内部电芯身份及保护阈值/延时/恢复行为未验证 |
| Functional Battery UVLO | 电池状态 | 电池放电资格 | 不控制有效 USB 的系统供电路径 |
| 3.3V regulator | System Rail | 3.3V | 500mA 连续为需求；1A/100ms 为 Design Target |
| 输出监控与保护 | 3.3V 状态、负载故障 | 启动许可、负载断开、限流或关闭 | UV/OV 断开负载、启动前关闭负载、持续过流关闭、硬短路快速限制或关闭；恢复策略仍为 Open Question |

## 关键不变量

- USB 接入时负载优先，剩余输入能力才用于充电。
- 输入不足时充电必须退让，不得拖垮系统输出。
- USB 可在无电池、电池保护板断开或电池功能性欠压时维持 System Rail。
- 无电池模式不依赖电池补充峰值。
- 电池欠压只影响电池放电资格，不关闭 USB 路径。
- 图中器件与网络名同步当前 Rev.A design record，但不代表 ERC、PCB、制造或硬件测试通过。
