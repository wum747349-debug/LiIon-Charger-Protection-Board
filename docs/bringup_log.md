# Stage 8 — Assembly / Bring-up Log

Status: `ASSEMBLY PREPARATION / TEST PLAN READY — NO HARDWARE TEST RESULT YET`

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

- PCB fabrication: `COMPLETE — bare PCB received / available for assembly`.
- Assembly: `NOT YET STARTED / PREPARING`.
- First power-on: `NOT YET RUN`.
- Bring-up: `NOT YET RUN`.
- Formal hardware test: `NOT YET RUN`.
- Hardware functional PASS claims: `NONE YET`.
- Battery exact mating / polarity / wire order: `PHYSICAL VERIFICATION PENDING` before first real battery connection.

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

Status: `NOT YET RUN`

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

Status: `NOT YET RUN`

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

Status: `NOT YET PASSED`

Proceed to powered Bring-up only when:

- B0 has no unresolved assembly blocker;
- B1 has no unexplained hard-short indication;
- battery and external load remain disconnected.

Passing G1 is permission to attempt first power only. It is **not** a functional PASS.

---

## B2A — Current-Limited First Power via `VBUS_PROT`

Status: `NOT YET RUN`

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

Status: `NOT YET RUN`

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

Status: `NOT YET RUN`

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

---

## B4 — Bench-Supply Battery Emulation / BUVLO

Status: `NOT YET RUN`

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

Status: `NOT YET PASSED`

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

Status: `NOT YET RUN`

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

Status: `NOT YET RUN`

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

Status: `NOT YET RUN`

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

Status: `NOT YET RUN`

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

Status: `NOT YET PASSED`

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

No hardware event has been executed yet.

For every actual test event, append an entry using this minimum structure:

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