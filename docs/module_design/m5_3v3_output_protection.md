# M5 — 3.3V Output Protection

Status: `COMPLETE FOR STAGE-3 HANDOFF`

## Responsibility and interfaces

M5 保持 Stage 2 Primary `TPS259474LRPWR`，负责 3.3V downstream load 的 startup qualification、UV/OV protection、inrush / dV/dt control、persistent-overcurrent circuit breaker、short-circuit protection，以及 latch-off / controlled recovery。当前 power boundary 为：

`M4 3V3 → U2 TPS259474LRPWR → 3V3_PROT → future M6 / external load`

后续外部负载必须使用 `3V3_PROT`，不得绕过 M5 直接使用 raw `3V3`。当前 connection record 来自 user-reported / screenshot-reviewed schematic capture，不是对 authoritative `.SchDoc` 的解析验证。

## Ambiguity-sensitive EN / OVLO / reset topology

```text
3V3 ── R5 100kΩ ──┬── U2.1 EN/UVLO
                   └── TP_M5_RESET

U2.1 EN/UVLO ── R9 8.25kΩ ── U2.2 OVLO/OVCSEL ── R13 54.9kΩ ── GND
```

`TP_M5_RESET` 与 U2 pin 1 是同一 node，但它只是普通 test point，不永久接 GND。板上另有独立 `TP_GND`；Bring-up 时可临时短接 `TP_M5_RESET ↔ TP_GND`，把 EN/UVLO 拉低并方便 latch reset，正常工作时两个 test point 完全断开。

## Structured connection record

- `U2.3 PG/AUXOFF → R10 47kΩ → 3V3`。PG 是当前 TPS259474 function，且与 ILM 明确分离。
- `3V3_PROT → R2 75kΩ / 1% → U2.4 PGTH/FLT# → R11 49.9kΩ / 1% → GND`。当前设计使用该 pin 的 PGTH intent；不把它误记为独立 FLT# connection。
- `U2.5 IN → 3V3`；`C4 100nF: 3V3 ↔ GND`。
- `U2.6 OUT → 3V3_PROT`；`C5 10uF / 0805: 3V3_PROT ↔ GND`；`C6 100nF: 3V3_PROT ↔ GND`。
- `U2.7 DVDT → C8 2.2nF → GND`。
- `U2.8 GND → GND`。
- `U2.9 ILM → R14 3.32kΩ / 1% / 1206 → GND`。ILM 与 PG 是独立网络/功能。
- `U2.10 ITIMER → C7 330nF → GND`。

This is the current designator mapping. `U3 = TLV7041DBVR`; the former `U3/R4/R17/R18/R22/R20/R21/R19/C10/C11/C12` M5 mapping is superseded and is not a current baseline. The documented net connectivity is synchronized from current Project schematic evidence; actual symbol-pin/footprint mapping remains `REQUIRES EDA VERIFICATION`.

Pin 1 / pin 2 的完整连接由上方 compact topology diagram 记录，避免把同一 network 再复制成第二份 pin table。

## Engineering basis

- Nominal UVLO rising ≈ 3.10V。
- Nominal OVLO rising ≈ 3.57V。
- `R14=3.32kΩ / 1%` 对应约 1A nominal current-limit setting。
- `C7=330nF` 保持 reviewed nominal ≈277ms typical over-current blanking。
- `R2=75kΩ / 1%` 与 `R11=49.9kΩ / 1%` 保持 PGTH nominal threshold ≈ 3.0V，同时提高 divider current，以降低 PGTH leakage-related error。

这些是基于 TI TPS25947 official datasheet 的 design basis / calculation，不是 measured evidence。PGTH threshold 未实测；这些参数不构成严格 ±5% output window 保证，nominal current-limit setting 也不证明 1A/100ms output transient 已满足。

## Open evidence boundary

- Schematic capture basis: `USER-REPORTED / SCREENSHOT-REVIEWED`。
- Actual `.SchDoc` pin mapping: `NOT VERIFIED`。
- ERC: `NOT YET VERIFIED`。
- Latest schematic/BOM review baseline: `SYNCHRONIZED`。
- UV/OV threshold、1A/100ms transient、hard short、thermal 与 latch/reset behavior: `NOT MEASURED`。

M5 已按 current schematic 完成 documentation-only designator sync；PG/ILM 分离、`C5 = 10uF / 0805`、`R14 = 1206`、`C7 = 330nF` 与 `C8 = 2.2nF` 已同步。该结论不是 `ERC PASS`、新的 Stage 4 PASS、PCB/layout verified 或 hardware verified。
