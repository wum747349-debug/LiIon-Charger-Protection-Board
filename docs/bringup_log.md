# Stage 8 — Assembly / Bring-up Log

Status: `REV.A ASSEMBLED — PRIMARY-FUNCTION BASIC BRING-UP COMPLETE; EXTENDED PERFORMANCE / PROTECTION VALIDATION DEFERRED`

Date initialized: 2026-09-05

## Purpose and evidence boundary

This file is the execution plan and evidence log for Rev.A Stage 8 assembly / Bring-up.

It records:

- assembly preparation and inspection;
- first-power-up execution;
- main power-chain / battery / charging / Power Path checks;
- actual measurements, observations, failures, fixes and retest evidence;
- deferred validation items that cannot yet be verified with the available equipment.

A planned test is **not** a PASS. No item may be promoted to measured / verified until an actual instrument reading, waveform, observation or other hardware evidence is recorded.

`docs/test_report.md` will be created when formal hardware testing starts and real measured results are available.

---

## Current hardware state

- PCB fabrication: `COMPLETE`.
- Assembly: `COMPLETE — Rev.A board hand-assembled`.
- First power-on: `COMPLETE`.
- Bring-up: `PRIMARY-FUNCTION BASIC BRING-UP COMPLETE — USB-only, bench Battery-only, real Battery-only, USB+Battery charging, repeated source insertion/removal and a basic 3.3V ripple/noise observation have real evidence`.
- Extended performance / protection validation: `DEFERRED — controlled load, transient, exact threshold, thermal and fault-injection items remain`.
- Hardware functional evidence: main power chain, approximately 500mA charging behavior and basic output stability have real measurements; this is not a full performance / protection qualification PASS.
- Battery connector was successfully used on the real pack during Bring-up; the exact pre-connection polarity reading was not separately captured in the execution log.

---

## Available instruments

| Instrument | Primary use |
| --- | --- |
| Digital multimeter | continuity / resistance checks, rail DC voltage, battery polarity, NTC resistance, optional series charge-current measurement |
| Oscilloscope | startup / transition waveform, ripple / obvious instability, `CHG_GATE` / `CE#` timing when required |
| Adjustable bench DC power supply with current limit | controlled first-power stimulus; Battery-only emulation and BUVLO sweep with USB absent |

### Global test constraints

- Do not connect the real battery until `J_BAT1` mating, pin order and polarity are physically verified with the multimeter. Board convention: `J_BAT1 pin 1 → BAT`, `pin 2 → GND`.
- Do not intentionally short the real Li-ion battery.
- Bench-supply battery emulation is Battery-only unless the supply is explicitly capable of safely sinking reverse current.
- Direct 5V injection into `VBUS_PROT` validates downstream power circuitry only; it does **not** validate TUSB320 attach / CC-current detection.
- USB-C CC / Source-capability validation requires a compliant USB-C Source connected through `USBC1`.
- For ripple measurements, use the shortest practical oscilloscope ground connection / ground spring near the measured rail.
- No direct-wire short-circuit test is required during initial Bring-up.

---

# Approved execution sequence

```text
B0  Assembly / Visual Inspection
        ↓
B1  Unpowered Static Screening
        ↓
G1  Pre-Power Gate
        ↓
B2A Current-Limited First Power via VBUS_PROT
        ↓
B2B Native USB-C Power-Up
        ↓
B3  Startup / Basic Ripple Observation
        ↓
B4  Bench-Supply Battery Emulation + BUVLO
        ↓
G2  Battery Mating / Polarity + NTC Sanity Gate
        ↓
B5  Real Battery-Only First Connection
        ↓
B6  USB + Battery Charging
        ↓
B7  USB / Battery Power-Path Transition
        ↓
B8  NTC / Charging Qualification
        ↓
G3  Basic Rev.A Bring-up Gate
```

Each step uses the same execution pattern:

`Preconditions → Stimulus → Measure / Observe → Expected behavior → STOP conditions → Evidence to record`

Do not skip a Gate.

---

## B0 — Assembly / Visual Inspection

Status: `RUN — no unresolved visual assembly blocker reported`

**Preconditions**

- Board unpowered.
- No USB connected.
- No battery connected.
- No external load connected.

**Inspect**

- component value / orientation / polarity;
- WSON / DFN / VQFN solder bridges, shift, insufficient wetting or suspicious exposed-pad soldering;
- USB-C fine-pitch pins and mechanical shell joints;
- Q1 / Q2 MOSFET orientation;
- BQ25185, TPS63802, TPS259474, TUSB320 and TLV7041 orientation;
- L1 placement and soldering;
- battery / NTC / output connector orientation and silkscreen readability;
- solder balls, debris, damaged pads, lifted components or obvious rework defects.

**STOP conditions**

Any unresolved orientation doubt, visible bridge, badly shifted leadless IC, damaged pad / trace or connector ambiguity.

**Evidence to record**

- assembly photos;
- inspection findings;
- any rework performed.

---

## B1 — Unpowered Static Screening

Status: `RUN — no hard short found on screened main rails`

**Preconditions**

- B0 complete with no unresolved assembly blocker.
- Board remains unpowered.

**Measure**

Using the multimeter, screen the major rails against GND:

- `VBUS_PROT ↔ GND`
- `SYS ↔ GND`
- `BAT ↔ GND`
- `3V3 ↔ GND`
- `3V3_PROT ↔ GND`

Also inspect continuity around any assembly area that looked suspicious during B0.

**Expected behavior**

No obvious hard short. Capacitor charging may cause changing / transient resistance readings; do not classify every low initial reading as a fault.

**STOP conditions**

Stable near-zero resistance / continuity behavior that is inconsistent with the circuit, especially on a major rail, until the cause is understood.

**Evidence to record**

Stable / comparative readings and any suspicious node investigated.

---

## G1 — Pre-Power Gate

Status: `PASSED FOR POWERED BRING-UP`

Proceed to powered Bring-up only when:

- B0 has no unresolved assembly blocker;
- B1 has no unexplained hard-short indication;
- battery and external load remain disconnected.

Passing G1 is permission to attempt first power only. It is **not** a functional PASS.

---

## B2A — Current-Limited First Power via `VBUS_PROT`

Status: `DEFERRED — TP_VBUS_PROT was not practical for bench-lead attachment; downstream chain was instead first exercised from the BAT connector with current-limited bench supply`

**Purpose**

Perform the first energized smoke-test of the downstream power chain with a controllable current limit before relying on a normal USB-C source.

**Preconditions**

- G1 passed.
- No battery connected.
- No external load connected.
- USB-C disconnected.

**Stimulus**

```text
Bench PSU + → TP_VBUS_PROT / VBUS_PROT
Bench PSU - → TP_GND / GND
```

Set approximately 5.0V and start with a conservative current limit appropriate for first power. Increase only if the board is behaving normally and the limit is clearly preventing intended startup.

**Measure / observe**

```text
TP_VBUS_PROT → TP_SYS → TP_3V3 → TP_3V3_PROT
```

Also observe bench-supply current-limit behavior and unexpected heating.

**Expected engineering behavior**

- `VBUS_PROT` follows the applied bench supply;
- BQ25185 establishes `SYS` without a battery if downstream circuitry is healthy;
- TPS63802 establishes `3V3` near its design target;
- `3V3_PROT` appears only if TPS259474 qualification is satisfied.

This step does **not** validate USB-C CC / TUSB320 operation.

**STOP conditions**

- immediate or persistent current limiting without an understood reason;
- rail collapse;
- strong oscillation;
- rapid / abnormal component heating;
- smoke, smell or visible damage.

**Evidence to record**

- current-limit setting;
- observed supply current;
- measured rail voltages;
- any abnormal behavior.

---

## B2B — Native USB-C Power-Up

Status: `RUN — basic USB-only DC path verified`

**Preconditions**

- B2A shows no downstream power-chain blocker.
- Bench supply disconnected.
- No battery connected.
- No external load connected.

**Stimulus**

Connect a compliant 5V USB-C Source through `USBC1`.

**Measure / observe**

```text
TP_VBUS_PROT → TP_SYS → TP_3V3 → TP_3V3_PROT
```

Observe whether native USB-C attachment establishes the same main power chain.

**Expected engineering behavior**

A valid USB-C attach supplies the board through the intended connector / CC path and establishes the downstream rails.

**STOP conditions**

Unexpected heating, rail collapse, obvious instability or a major mismatch versus B2A.

**Evidence to record**

Rail voltages and qualitative native USB-C startup behavior.

---

## B3 — Startup / Basic Ripple Observation

Status: `PARTIAL / BASIC OBSERVATION COMPLETE — representative 3.3V output ripple/noise capture recorded; formal startup and loaded/high-bandwidth ripple characterization deferred`

**Preconditions**

- B2A / B2B basic DC behavior is understood.
- No external power load required.

**Measure / observe**

Use the oscilloscope to inspect, as practical:

- `SYS` startup;
- `3V3` startup;
- `3V3_PROT` startup;
- overshoot / ringing / obvious instability;
- no-load or light-load 3.3V ripple.

Recommended first capture:

```text
CH1 → TP_SYS
CH2 → TP_3V3_PROT
```

Additional captures may use `TP_3V3`, `TP_CHG_GATE` and `TP_CE_N` if startup qualification needs diagnosis.

**Expected engineering behavior**

Rails establish without gross instability or destructive overshoot. Exact acceptance limits require actual waveform review.

**STOP conditions**

Large unexplained oscillation, repeated startup collapse or abnormal overshoot that could endanger the board.

**Evidence to record**

Saved waveform captures and measurement settings.

A no-load / light-load ripple result does **not** validate loaded ripple or the 500mA requirement.

**Recorded basic ripple/noise observation**

One representative oscilloscope capture of the 3.3V output showed approximately:

- vertical scale: `10mV/div`;
- time base: `10ms/div`;
- `Vpp ≈ 12.4mV`;
- `Vrms ≈ 3.53mV`;
- dominant displayed period `≈20ms`;
- displayed frequency `≈50Hz`.

Other observed output-ripple readings were reported to be approximately the 10mV range.

This is recorded as a **low-frequency output ripple/noise observation under the actual Bring-up probing setup**, not as the TPS63802 switching-ripple frequency. The visible approximately 50Hz component may include environmental / mains-coupled pickup or probe-loop noise; TPS63802 switching behavior is in the MHz range and requires a much shorter time base and controlled probing method for dedicated switching-ripple characterization.

The capture supports the limited Bring-up conclusion that no large low-frequency oscillation was observed at the measured output. It does not replace loaded, bandwidth-controlled ripple qualification.

---

## B4 — Bench-Supply Battery Emulation / BUVLO

Status: `PARTIAL — nominal 3.8V Battery-only emulation verified; BUVLO sweep deferred`

**Preconditions**

- USB absent.
- Real battery absent.
- Bench supply current limit enabled.

**Stimulus**

```text
Bench PSU + → J_BAT1 pin 1 / BAT
Bench PSU - → J_BAT1 pin 2 / GND
```

Start at a normal 1S battery-range voltage.

**Measure / observe**

```text
BAT → SYS → 3V3 → 3V3_PROT
```

Then lower the simulated battery voltage gradually and observe:

- approximate BATFET / BUVLO transition;
- `SYS`, `3V3`, `3V3_PROT` behavior;
- recovery while raising the simulated battery voltage again.

**Expected engineering behavior**

Battery-only operation should establish the main chain above the applicable battery qualification threshold and eventually lose Battery-to-SYS qualification as the simulated battery is reduced sufficiently.

**STOP conditions**

Unexpected current limiting, abnormal heating or behavior inconsistent with the expected Battery-only path.

**Evidence to record**

Approximate transition / recovery voltages and rail measurements. Treat these as measured board-level behavior, not a guaranteed datasheet threshold unless tolerance analysis supports that statement.

---

## G2 — Battery Mating / Polarity + NTC Sanity Gate

Status: `PARTIAL EVIDENCE — real battery and NTC were successfully connected; room-temperature NTC measured about 9kΩ; separate pre-connection polarity reading was not recorded`

This Gate is mandatory before the first real battery connection.

### Battery physical verification

- verify connector mechanical mating;
- measure the actual battery connector polarity with the multimeter;
- confirm battery positive lands on board `BAT` / `J_BAT1 pin 1`;
- confirm battery negative lands on GND / `J_BAT1 pin 2`;
- verify no housing / cable orientation ambiguity.

Do **not** infer polarity from wire color or product images alone.

### NTC pre-charge sanity check

Before real charging:

- confirm the NTC is connected to `J_NTC1`, not `J_BAT1`;
- measure room-temperature NTC resistance and confirm it is broadly consistent with the selected nominal 10kΩ NTC;
- check for obvious NTC open / hard short / harness fault.

Passing G2 permits the real battery to be connected. It does not validate charging or NTC thresholds.

---

## B5 — Real Battery-Only First Connection

Status: `RUN — basic real Battery-only DC path verified`

**Preconditions**

- G2 passed.
- USB disconnected.
- External load disconnected.

**Stimulus**

Connect the verified real battery only.

**Measure / observe**

```text
TP_BAT → TP_SYS → TP_3V3 → TP_3V3_PROT
```

Observe for abnormal current, rail collapse or heating.

**Expected engineering behavior**

The real battery should reproduce the basic Battery-only power path already exercised with the bench-supply emulator, subject to its actual state of charge and BQ25185 qualification.

**STOP conditions**

Unexpected heating, rail collapse, connector heating, smoke / odor, or behavior materially inconsistent with B4.

**Evidence to record**

Battery open-circuit voltage before connection and measured board rails after connection.

---

## B6 — USB + Battery Charging Behavior

Status: `RUN — charging established after NTC connector fault was repaired; battery charge current measured about 0.49A`

**Preconditions**

- B5 real Battery-only operation is understood.
- NTC remains correctly connected.
- Use a compliant USB-C Source through `USBC1`.

**Stimulus**

Connect USB while the verified real battery is installed.

**Measure / observe**

- battery voltage before charging;
- actual battery charging current when safely measured in series with the multimeter;
- charging authorization behavior;
- `CHG_GATE` and `CE#` if diagnosis is needed;
- qualitative current taper as battery voltage rises, if the session duration allows;
- abnormal heating by safe observation.

**Expected engineering behavior**

Charging occurs only when USB, temperature and 3V3 qualification permit it. Actual charge current must be measured; do not infer a verified 500mA result from the programming resistor alone.

**STOP conditions**

Unexpected battery / board heating, polarity concern, unstable rails, excessive current, or any sign of battery distress.

**Evidence to record**

Battery voltage, measured charge current, relevant qualification-node readings and operating condition.

---

## B7 — USB / Battery Power-Path Transition

Status: `PARTIAL — repeated USB insertion/removal showed stable DMM reading at 3V3_PROT; oscilloscope transition capture deferred`

**Preconditions**

- USB-only, Battery-only and USB+Battery DC operation are individually understood.

**Measure / observe**

With the oscilloscope on `3V3_PROT` and optionally `SYS`, capture:

- USB insertion while battery-powered;
- USB removal while battery-powered;
- minimum observed output voltage / dropout;
- recovery behavior / abnormal oscillation.

**Expected engineering behavior**

The board transitions between valid power sources without destructive instability. Do not claim zero-drop / seamless transfer unless the waveform supports it.

**STOP conditions**

Repeated restart, prolonged output collapse, oscillation or abnormal heating.

**Evidence to record**

Waveform captures and measured minima / recovery observations.

---

## B8 — NTC / Charging Qualification

Status: `PARTIAL — room-temperature NTC path and open-NTC behavior observed; exact thermal thresholds deferred`

**Available checks with current instruments**

- measure room-temperature NTC resistance;
- observe `NTC_TS`, `CHG_GATE` and `CE#` under normal conditions;
- if performed under a safe setup, disconnect the NTC and verify the fault is not silently treated as a fully valid charging condition;
- later, use resistor substitution to exercise threshold logic before attempting controlled hot / cold battery testing.

**Evidence boundary**

Exact hot / cold trip temperature, tolerance stack and pouch thermal lag remain deferred until controlled temperature stimulus and measurement are available.

**STOP conditions**

Any test method that would require unsafe battery heating / cooling, damaging the pouch, or bypassing charge-safety mechanisms without a controlled setup.

**Evidence to record**

Measured resistance / voltages / logic state and the exact fault-injection setup used.

---

## G3 — Basic Rev.A Bring-up Gate

Status: `FORMAL GATE NOT CLOSED — primary-function practical Bring-up complete, but the originally approved sequence still contains deferred / deviated items`

Basic board-level Bring-up may be considered complete only after real evidence supports the main intended operating paths:

- controlled downstream first power via `VBUS_PROT`;
- native USB-C power-up;
- `VBUS_PROT → SYS → 3V3 → 3V3_PROT` establishment;
- startup behavior checked for gross instability;
- Battery-only bench emulation and approximate BUVLO behavior;
- real battery mating / polarity verified;
- real Battery-only operation;
- basic real charging behavior;
- USB-only / Battery-only / USB+Battery Power Path operation and source-transition observation;
- no unresolved safety-critical Bring-up blocker.

Passing G3 may support a claim such as:

`Main Rev.A power / charging / Power Path Bring-up completed with recorded hardware evidence.`

It does **not** imply the deferred performance / fault tests below are PASS.

---

# Deferred validation

The current instrument set is insufficient for formal completion of the following items. Keep them `DEFERRED / NOT YET VERIFIED` until an appropriate controlled load, fixture or thermal setup is available.

| Deferred item | Reason / required capability |
| --- | --- |
| 3.3V / 500mA continuous-load capability | suitable controlled power load / electronic load required |
| Loaded output regulation and loaded ripple | controlled load required |
| 1A / 100ms design-target transient | controlled pulsed load and waveform capture required |
| Load-step droop / recovery | controlled dynamic load required |
| Exact TPS259474 over-current threshold | controlled current sweep / load required |
| TPS259474 over-current timer | controlled fault stimulus + oscilloscope required |
| Hard-short response / latch-off | controlled short / load fixture required; no direct-wire initial test |
| M5 UV / OV qualification | controlled rail stimulus / fixture required |
| Full efficiency curve | simultaneous calibrated input / output power measurement under controlled loads |
| Quantitative component temperature rise | thermal measurement tool required |
| Exact NTC trip temperatures / pouch thermal lag | controlled temperature stimulus + temperature measurement required |

---

# Execution log

## 2026-09-29 to 2026-10-02 — Rev.A first main Bring-up session

### Assembly / unpowered screening

- Rev.A was assembled and visually screened before powered tests.
- Main power rails were checked for an obvious hard short; none was found.
- No claim is made that every assembly defect mode was formally inspected or that resistance values were fully characterized.

### Bench Battery-only emulation

**Setup**

- USB absent.
- Real battery absent.
- Bench supply connected to the battery connector as a simulated 1S battery.
- Bench supply set to approximately 3.8V with current limiting.

**Measured**

| Node | Measured |
| --- | ---: |
| BAT | 3.79V |
| SYS | 3.79V |
| 3V3 | 3.31V |
| 3V3_PROT | 3.30V |

**Observation**

Nominal Battery-only power chain established:

`BAT → SYS → TPS63802 3V3 → TPS259474 3V3_PROT`.

This does not verify BUVLO threshold, loaded regulation or output current capability.

### Native USB-only

**Setup**

- Real battery absent.
- Bench supply removed.
- Compliant 5V USB-C source connected through USBC1.

**Measured**

| Node | Measured |
| --- | ---: |
| VBUS_PROT | 5.07V |
| SYS | 4.67V |
| 3V3 | 3.30V |
| 3V3_PROT | 3.29V |

**Observation**

Native USB-C attachment established the downstream power chain. This is a no-load / light-load DC observation, not a 3.3V/500mA output validation.

### Real Battery-only

**Measured**

| Node | Measured |
| --- | ---: |
| BAT | 3.95V |
| SYS | 3.95V |
| 3V3 | 3.31V |
| 3V3_PROT | 3.29V |

**Observation**

Real Battery-only operation reproduced the expected main DC power path.

### USB + Battery / Power Path observation

With the real battery connected, USB insertion produced approximately:

| Node | Measured / observed |
| --- | ---: |
| BAT | about 3.95V before charge observation |
| SYS | 4.56V |
| 3V3 | 3.31V |
| 3V3_PROT | 3.29V steady-state |

One early USB insertion produced a transient DMM reading of approximately 0.4V at `3V3_PROT`, followed by recovery to 3.29V. Repeated later USB insertions/removals did not reproduce the low reading and `3V3_PROT` remained about 3.29V. No oscilloscope waveform was captured, so zero-drop / seamless source transfer is **not** claimed.

### NTC / charging fault found during Bring-up

**Observed fault**

- Selected external NTC: `MF52D 103F3435-100` / LCSC `C394023`.
- Detached NTC resistance measured approximately 9kΩ at room conditions.
- R15 measured approximately 2.7kΩ.
- Unpowered `TP_NTC_TS → GND` with the NTC connected measured about 12kΩ.
- During the fault, `TP_NTC_TS` intermittently fell to approximately 0V / 0.01V and `TP_CHG_GATE` was observed around 0.2V; charging stopped.
- With NTC disconnected and USB present, `TP_NTC_TS` was observed at approximately 2.9V.
- During a normal interval, `TP_NTC_TS` measured approximately 0.46V and charging was active.
- In one fault event BAT was observed around 4.06V while charging and then about 3.98V after charging stopped.

**Diagnosis and correction**

The intermittent failure was localized to the external NTC connector / terminal contact rather than the NTC element or R15. The faulty NTC terminal was replaced.

**Retest**

After terminal replacement the NTC path became stable and normal charging resumed. The successful approximately 0.46V `NTC_TS` observation is consistent with the expected room-temperature 10k-class NTC network. Exact temperature thresholds were not tested.

### 3.3V output ripple / noise observation

A representative oscilloscope capture was recorded during the Bring-up session. The displayed measurements were approximately:

| Item | Observed |
| --- | ---: |
| Vertical scale | 10mV/div |
| Time base | 10ms/div |
| Vpp | 12.4mV |
| Vrms | 3.53mV |
| Vmax | +6.8mV |
| Vmin | -5.6mV |
| Displayed period | 20.00ms |
| Displayed frequency | 50.00Hz |

Other measurements of the 3.3V output were reported to be around the 10mV ripple/noise range.

The approximately 50Hz visible component is treated as low-frequency ripple/noise under the actual probe setup and may include environmental / mains pickup. It is **not** interpreted as the TPS63802 switching frequency. No large low-frequency instability was evident in this representative capture.

Formal loaded ripple, probe-loop-controlled high-frequency ripple and switching-frequency-domain characterization remain deferred.

### Battery current measurement

The multimeter was inserted in series with the battery positive lead using the high-current DC range.

- USB absent: battery current magnitude approximately **0.01A**, corresponding to Battery-only board consumption; current direction is battery → board.
- USB present and charging: battery current approximately **0.49A**, direction board / charger → battery.

The approximately 0.49A measured charging current is consistent with the Rev.A approximately 500mA fast-charge design target. This is a Bring-up measurement at the observed battery state, not a full charge-profile or thermal qualification.

### Session conclusion

Real evidence now supports:

- nominal bench-supply Battery-only operation;
- native USB-only operation;
- real Battery-only operation;
- USB + Battery operation;
- actual battery charging at approximately 0.49A after the NTC terminal fault was corrected;
- repeated USB insertion/removal with stable steady-state `3V3_PROT` by DMM.

Still deferred / not verified in this session:

- direct bench first-power injection through `VBUS_PROT`;
- formal startup and source-transition waveforms; dedicated loaded / high-bandwidth switching-ripple characterization;
- BUVLO threshold sweep and recovery;
- 3.3V / 500mA continuous-load test;
- 1A / 100ms transient;
- controlled over-current / short-circuit / UV / OV protection tests;
- efficiency and quantitative thermal testing;
- exact NTC hot / cold trip temperatures and thermal lag.

The first practical Rev.A Bring-up session is considered **complete for the primary intended functions**. Real evidence supports USB-only, Battery-only, USB+Battery, approximately 500mA charging, NTC qualification behavior after connector repair, repeated source insertion/removal at steady state, and basic 3.3V output ripple/noise observation.

The formal G3 gate is not closed only because the originally approved execution sequence included items that were intentionally deferred or deviated from, notably direct `VBUS_PROT` bench injection, BUVLO sweep and formal startup/source-transition waveform capture. This is a documentation / evidence-boundary distinction, not an unresolved functional fault.

The correct current claim is:

`Rev.A primary-function Basic Bring-up is complete with hardware evidence for the main USB / Battery power paths, approximately 500mA charging and basic 3.3V output stability. Extended performance, threshold and protection validation remains deferred.`

## 2026-10-30 — TPS63802 3.3 V light-load ripple / suspected PFM observation

**Scope / evidence:** Rev.A; RIGOL DS1102E oscilloscope photos provided by the user in the test conversation. This is a follow-up observation, separate from the 2026-09/10 approximately 12.4 mVpp / 50 Hz capture. Raw photographs are not committed in this repository. No external output load was connected; on-board quiescent / divider / protection loads still exist. The supply modes were exercised separately, **not** as a live USB-to-battery transition capture.

**Power and probe setup:** Native USB-C supply with battery disconnected (USB-only), or real battery with USB disconnected (Battery-only). Dedicated regulator-output readings were taken at the `TP_3V3` node with ground at C15 GND, upstream of TPS259474 (`3V3_PROT`). Scope setup reported/seen across captures: CH1, 1× probe, AC coupling, 20 MHz bandwidth limit, approximately 10 mV/div, 500 µs/div for repeated low-frequency waveform; 10 µs/div with CH1 rising-edge Normal trigger to examine the fast recovery edge. Exact probe-loop geometry, actual in-session SYS voltage, sampling mode/rate and a calibrated measurement noise floor were not independently documented.

| Operating condition | Measurement node | Displayed low-frequency Vpp | Displayed waveform repetition | Evidence interpretation |
| --- | --- | ---: | ---: | --- |
| USB-only, no external load | `TP_3V3` vs C15 GND | ≈39 mVpp | ≈0.8 kHz (≈1.25 ms period) | Periodic fast-rise / slow-decay output-voltage ripple |
| Battery-only, no external load | `TP_3V3` vs C15 GND | ≈42–43 mVpp | ≈0.77 kHz (≈1.30 ms period) | Similar periodic ripple; repeated captures were broadly consistent |
| Initial USB-only exploratory capture | `3V3_PROT` external output connector | ≈31.6 mVpp | ≈806.5 Hz | Different node and probe geometry; **not a like-for-like amplitude comparison** |

**Waveform / controls:** The 500 µs/div captures show roughly three to four repeated sawtooth-like cycles, with fast voltage recovery followed by a slower decline. A 10 µs/div rising-edge-triggered capture stably positioned one recovery event around the trigger marker; it **did not resolve or quantify the inductor/MOSFET MHz-scale switching pulses**. Triggering aligned repeated events and did not alter the regulator output.

**Engineering interpretation (not a verified operating-mode measurement):**
- This behavior is **consistent with** TPS63802 light-load power-save / PFM control at `MODE = GND`: pause switching while the output supplies small loads, then restore charge in a switching burst. TI TPS63802 Datasheet Rev.D §9.4.2 explains skip-comparator-controlled switching pauses; Figure 10-4 illustrates burst-frequency dependence on output current.
- The displayed ≈0.77–0.8 kHz is a **suspected ripple / burst repetition frequency**, **not** the IC's inductor switching frequency. Datasheet typical MHz switching values under `MODE = HIGH` / forced PWM are not measurements of this Rev.A board.
- Both source conditions produced similar low-frequency shapes and amplitude order. This supports a repeatable **light-load waveform observation**, but does not prove PFM mode exclusively, prove output ripple compliance, or eliminate measurement pickup / aliasing.

**Status:** `OBSERVATION COMPLETE — USB-only and Battery-only light-load 3V3 ripple`. No formal loaded-ripple PASS. Still `DEFERRED / NOT YET VERIFIED`: controlled-load ripple, no-probe-influence / noise-floor cross-check, sustained 100 ms-scale low-frequency scan under the same two setups, internal switching frequency / current waveform, load-step response, precise ripple-limit compliance, and source-transition waveform.

Datasheet: https://www.ti.com/lit/ds/symlink/tps63802.pdf

---

For every additional actual test event, append an entry using this minimum structure:

```text
Date / board ID:
Step / Gate:
Assembly state:
Test setup / instrument:
Stimulus / operating mode:
Measured node(s):
Measured result / waveform:
Expected behavior:
PASS / FAIL / OBSERVATION:
STOP condition encountered: YES / NO
Issue / diagnosis:
Change made, if any:
Retest result:
Evidence file / photo / oscilloscope capture:
```

Do not rewrite the Approved execution sequence to fit a result after the fact. Record deviations, failures, fixes and retests in this Execution log.