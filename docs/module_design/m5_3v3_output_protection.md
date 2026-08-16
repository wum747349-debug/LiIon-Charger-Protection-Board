# M5 — 3.3V Output Protection

Status: `COMPLETE FOR STAGE-3 HANDOFF`

## Responsibility and interfaces

M5 保持 Stage 2 Primary `TPS259474LRPWR`，负责 3.3V downstream load 的 startup qualification、UV/OV protection、inrush / dV/dt control、persistent-overcurrent circuit breaker、short-circuit protection，以及 latch-off / controlled recovery。当前 power boundary 为：

`M4 3V3 → U3 TPS259474LRPWR → 3V3_PROT → future M6 / external load`

后续外部负载必须使用 `3V3_PROT`，不得绕过 M5 直接使用 raw `3V3`。当前 connection record 来自 user-reported / screenshot-reviewed schematic capture，不是对 authoritative `.SchDoc` 的解析验证。

## Ambiguity-sensitive EN / OVLO / reset topology

```text
3V3 ── R4 100kΩ ──┬── U3.1 EN/UVLO
                   └── TP_M5_RESET

U3.1 EN/UVLO ── R17 8.25kΩ ── U3.2 OVLO ── R18 54.9kΩ ── GND
```

`TP_M5_RESET` 与 U3 pin 1 是同一 node，但它只是普通 test point，不永久接 GND。板上另有独立 `TP_GND`；Bring-up 时可临时短接 `TP_M5_RESET ↔ TP_GND`，把 EN/UVLO 拉低并方便 latch reset，正常工作时两个 test point 完全断开。

## Structured connection record

- `U3.3 PG → R22 47kΩ → 3V3`。PG 是当前 TPS259474 function，且与 ILM 明确分离。
- `3V3_PROT → R20 75kΩ / 1% → U3.4 PGTH → R21 49.9kΩ / 1% → GND`。PGTH 是当前 TPS259474 function，不是 `FLT#`。
- `U3.5 IN → 3V3`；`C10 100nF: 3V3 ↔ GND`。
- `U3.6 OUT → 3V3_PROT`；`C11 10uF / 0805: 3V3_PROT ↔ GND`；`C12 100nF: 3V3_PROT ↔ GND`。
- `U3.7 DVDT` uses the reviewed Rev.A 2.2nF dV/dt capacitor; the obsolete pre-closeout `C14` designator is not retained here because `C14` is now a 22uF M4 output capacitor.
- `U3.8 GND → GND`。
- `U3.9 ILM → R19 3.32kΩ / 1% / 1206 → GND`。ILM 与 PG 是独立网络/功能。
- `U3.10 ITIMER` 使用 reviewed Rev.A schematic 中的定时电容；旧文档将该电容误记为 `C13`，该 designator 不再属于 current baseline。

Pin 1 / pin 2 的完整连接由上方 compact topology diagram 记录，避免把同一 network 再复制成第二份 pin table。

## Engineering basis

- Nominal UVLO rising ≈ 3.10V。
- Nominal OVLO rising ≈ 3.57V。
- `R19=3.32kΩ / 1%` 对应约 1A nominal current-limit setting。
- Reviewed Rev.A ITIMER value 对应约 277ms typical over-current blanking；current designator 以 authoritative `.SchDoc` 为准。
- `R20=75kΩ / 1%` 与 `R21=49.9kΩ / 1%` 保持 PGTH nominal threshold ≈ 3.0V，同时提高 divider current，以降低 PGTH leakage-related error。

这些是基于 TI TPS25947 official datasheet 的 design basis / calculation，不是 measured evidence。PGTH threshold 未实测；这些参数不构成严格 ±5% output window 保证，nominal current-limit setting 也不证明 1A/100ms output transient 已满足。

## Open evidence boundary

- Schematic capture basis: `USER-REPORTED / SCREENSHOT-REVIEWED`。
- Actual `.SchDoc` pin mapping: `NOT VERIFIED`。
- ERC: `NOT YET VERIFIED`。
- Latest schematic/BOM review baseline: `SYNCHRONIZED`。
- UV/OV threshold、1A/100ms transient、hard short、thermal 与 latch/reset behavior: `NOT MEASURED`。

M5 已完成 Stage 3 handoff 所需的 design-record sync；PG/ILM 分离、`C11 = 10uF / 0805` 与 `R19 = 1206` 已同步。该结论不是 `ERC PASS`、Stage 4 PASS、PCB/layout verified 或 hardware verified。
