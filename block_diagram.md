# 系统框图

状态：Stage 1 Baseline — Gate 1.5 PASS

## 系统能量流

```text
[USB-C 5V, Sink-only]
          |
          v
[CC / 输入保护与能力限制]
          |
          v
[Charger / Power Path / System Rail] <----> [Battery Pack]
          |
          v
[Buck-Boost 3.3V（架构方向，非器件选择）]
          |
          v
[电压监控与输出保护 / 负载断开]
          |
          v
[STM32 / 普通传感器 / 类似低压实验负载]

[External NTC] -------------------------> [Charger Temperature Input]
                                                     |
                                                     v
                                  [Charger / Power Path control]
```

## 运行与控制关系

```text
USB 有效 -------------------------------> 维持 System Rail（允许无电池运行）
系统负载需求 ---------------------------> Power Path 优先分配输入能力
输入能力不足 ---------------------------> 降低或暂停充电
Functional Battery UVLO ---------------> 只控制电池放电资格
电池功能性欠压 + USB 有效 -------------> USB 路径保持开启
External NTC 异常 ----------------------> 禁止充电，不关闭 USB 系统路径
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
| Battery Pack | 包内电芯与未知保护板 | 双向充放电能量 | 推测 1S2P 仅为 Working Assumption；包内参数未证明 |
| Functional Battery UVLO | 电池状态 | 电池放电资格 | 不控制有效 USB 的系统供电路径 |
| 3.3V regulator | System Rail | 3.3V | 500mA 连续为需求；1A/100ms 为 Design Target |
| 输出监控与保护 | 3.3V 状态、负载故障 | 启动许可、负载断开、限流或关闭 | UV/OV 断开负载、启动前关闭负载、持续过流关闭、硬短路快速限制或关闭；恢复策略仍为 Open Question |

## 关键不变量

- USB 接入时负载优先，剩余输入能力才用于充电。
- 输入不足时充电必须退让，不得拖垮系统输出。
- USB 可在无电池、电池保护板断开或电池功能性欠压时维持 System Rail。
- 无电池模式不依赖电池补充峰值。
- 电池欠压只影响电池放电资格，不关闭 USB 路径。
- 图中不代表具体器件、已完成 EDA 或已通过任何电气测试。
