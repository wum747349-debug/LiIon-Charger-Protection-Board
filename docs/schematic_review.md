# Stage 4 — Schematic Review Record

Status: `CLOSED — PCB LAYOUT-ENTRY APPROVED`

## Review scope and evidence

- Formal Schematic Review was completed against the latest complete Rev.A Schematic PDF and current BOM supplied for review.
- Cross-Module Integration Review and BOM Consistency Review were completed; no new substantive board-level electrical blocker was found.
- The reviewed baseline includes `C1,C5,C11 = 10uF / 0805`, `L1 = XFL4015-471MEC / 470nH`, D2 = Littelfuse `SMF6.0A` / LCSC `C207260`, and the selected external NTC/interface recorded below.
- These statements record the completed human + AI review supplied as current Project evidence; this documentation transaction did not rerun EDA tools or alter the schematic, PCB or BOM XLSX.

## 当前结论

**可以进入 PCB Layout。** No unresolved finding affects board-level safety, schematic correctness, package/footprint decision, interface definition or critical PCB layout input. Stage 4 is closed and Layout Preflight is explicitly authorized.

Battery exact mechanical mating and battery polarity/wire order are not claimed as verified. They remain a mandatory physical gate before the first actual battery connection; the prohibition on connecting the battery before that check controls the downstream risk without changing the reviewed board-side interface or blocking Layout Preflight.

## 问题清单

| 编号 | 模块 | 问题描述 | 风险等级 | 处置 | 状态 | 证据或关联文件 |
| --- | --- | --- | --- | --- | --- | --- |
| SR-001 | M3 | External NTC exact selection and interface were previously open | 中 | Freeze `MF52D 103F3435-100` / C394023 and C5221417 harness; retain existing R15 network | 已关闭 | `docs/module_design/m3_battery_external_ntc.md` |
| SR-002 | M1 | D2 exact sourcing wording was stale against the current BOM | 低 | Synchronize Littelfuse `SMF6.0A` / C207260 without changing topology | 已关闭 | `docs/module_design/m1_usb_c_input_cc_current_control.md` |
| SR-003 | Battery interface | Actual battery mating and polarity/wire order cannot be checked without the battery | 中（由首次连接禁令控制） | Verify exact mating and polarity with the physical battery and a multimeter before first connection | `PHYSICAL VERIFICATION PENDING`；非 Layout blocker | `requirements.md`; `docs/module_design/m3_battery_external_ntc.md` |
| SR-004 | BOM metadata | Ordinary commodity R/C exact sourcing and non-electrical metadata may remain incomplete | 低 | Complete during Manufacturing Preparation | Deferred；非 schematic blocker | Current BOM review disposition |

## Frozen M3 interface

- External NTC: Nanjing Shiheng `MF52D 103F3435-100`, LCSC `C394023`; `10kΩ @25°C`, R25 tolerance `±1%`, B25/85 `3435K`, Beta tolerance `±1%`, epoxy/wired, 100mm insulated leads.
- Circuit remains `J_NTC1 → R15 2.7kΩ → NTC_TS`; nominal conservative calculated charging window remains approximately 4°C–39°C.
- `J_NTC1 = HX-XH2.54-2PZZ`; harness `C5221417`, HX 2.54-2P DT, 200mm, 24AWG, pre-crimped 2-pin female.
- NTC is non-polarized; maintenance convention is red → J_NTC1 pin 1 → R15 / `NTC_TS`, black → J_NTC1 pin 2 → GND.
- `J_BAT1 = HX-XH2.54-2PZZ`; pin 1 → BAT, pin 2 → GND. Connector capability is approximately 3A / 250V and the PCB pitch drawing is 2.5mm; this does not verify actual battery mating or polarity.

## 需要核对 datasheet / downstream evidence 的项目

| 器件 / 接口 | 需要核对或验证的项目 | 处置阶段 |
| --- | --- | --- |
| MF52D 103F3435-100 + pouch attachment | Actual hot/cold trip temperatures, tolerance stack, pouch thermal lag and attachment behavior | Bring-up / Test and mechanical verification |
| J_BAT1 + actual battery lead | Exact mechanical mating, pin order and measured polarity | Mandatory before first actual battery connection |
| Littelfuse SMF6.0A | Placement, return path and Project-level surge/ESD behavior | PCB Layout and hardware verification |
| TPS63802 / TPS259474 power path | Earliest startup transient, UV/OV/PGTH, over-current, short, thermal and latch/reset behavior | Bring-up / Test |

## 建议/必须保留的测试点

| 网络 | 测试目的 | 是否必须 |
| --- | --- | --- |
| `VBUS_PROT / SYS / BAT / 3V3 / 3V3_PROT / GND` | Power-path, startup and protection measurements | 是 |
| `NTC_TS` | Actual trip-temperature and fault-injection verification | 是 |
| `PG / CHG_ENABLE_CMD / CHG_GATE / CE#` | Charging qualification and earliest-startup observation | 是 |

## Evidence boundary

- ERC: `NOT YET VERIFIED` (no ERC report is currently recorded). Per the bound Framework, absence of ERC output is not itself a Stage 4 blocker and is not promoted to PASS.
- PCB DRC, simulation, layout verification, manufacturing, hardware measurement, actual temperature trips and bring-up: `NOT YET VERIFIED`.
- Stage 4 closure authorizes Layout Preflight only; it does not claim manufacturing release or any downstream verification PASS.

## 下一步操作

1. Enter Stage 5 Layout Preflight and freeze manufacturing/mechanical baseline, footprints, connector orientation/silkscreen, PCB rules and critical power/protection placement inputs.
2. Keep NTC trip/thermal-lag verification and other hardware behavior in downstream Bring-up/Test.
3. Do not connect the actual battery until exact mating and polarity/wire order have been physically verified.
