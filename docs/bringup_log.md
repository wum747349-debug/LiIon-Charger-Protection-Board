# Stage 8 — Assembly / Bring-up Log

Status: `ASSEMBLY PREPARATION / TEST PLAN READY — NO HARDWARE TEST RESULT YET`

Date initialized: 2026-09-05

## Purpose and evidence boundary

This file is activated because the manufactured PCB is now available and assembly / Bring-up is about to begin.

It records:

- assembly preparation and inspection;
- first-power-up sequence;
- available instruments and their intended use;
- planned Bring-up / functional tests;
- actual observations, failures, fixes and measurements once they occur.

A planned test is **not** a PASS. No item below may be promoted to measured / verified until an actual instrument reading, waveform, observation or other hardware evidence is recorded.

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

## Available test instruments

Confirmed available instruments:

| Instrument | Primary use in this project |
| --- | --- |
| Digital multimeter | continuity / resistance checks, rail DC voltage, battery polarity, NTC resistance, optional series charge-current measurement |
| Oscilloscope | `SYS` / `3V3` / `3V3_PROT` startup waveform, USB/Battery transition, ripple/noise observation, `CHG_GATE` / `CE#` timing when required |
| Adjustable bench DC power supply with current limit | controlled first-power stimulus where appropriate; battery-only emulation and BUVLO sweep with USB absent |

### Important limits of the current instrument set

The following cannot yet be claimed as formally verified without additional load / thermal equipment or a dedicated test fixture:

- 3.3V / 500mA continuous-load capability;
- 1A / 100ms load-transient target;
- exact TPS259474 over-current trip current and timer behavior;
- controlled hard-short / latch-off qualification;
- full efficiency curve;
- accurate component temperature-rise characterization without a thermal measurement tool;
- exact NTC trip-temperature / pouch thermal-lag characterization without controlled temperature measurement.

These are deferred rather than assumed PASS.

---

## Safety / test constraints

1. Do not connect the real battery until `J_BAT1` mating, pin order and polarity are physically checked with the multimeter. Current board convention is `J_BAT1 pin 1 → BAT`, `pin 2 → GND`.
2. Do not intentionally short the real Li-ion battery.
3. Bench-supply battery emulation is for **Battery-only** testing with USB absent unless the supply is explicitly capable of safely sinking reverse current. An ordinary bench supply must not be treated as a rechargeable battery during USB charging tests.
4. USB-C CC / Source-capability validation requires an actual compliant USB-C Source through `USBC1`. Direct 5V injection into `VBUS_PROT` can validate downstream power circuitry only; it does not prove TUSB320 attach / CC-current detection behavior.
5. For oscilloscope ripple measurements, use the shortest practical ground connection / ground spring near the measured rail to avoid probe-loop pickup being mistaken for real ripple.
6. No direct-wire short-circuit test is required during initial Bring-up. M5 short / latch testing is deferred until a controlled load method is available.

---

## Bring-up execution order

### B0 — Assembly and visual inspection

Status: `NOT YET RUN`

Check before applying power:

- component value / orientation / polarity;
- WSON / DFN / VQFN solder bridges or insufficient wetting;
- USB-C connector soldering and mechanical joints;
- Q1 / Q2 MOSFET orientation;
- BQ25185, TPS63802, TPS259474, TUSB320 and TLV7041 orientation;
- L1 placement and soldering;
- battery / NTC / output connector orientation and silkscreen readability;
- exposed-pad soldering quality where visually accessible.

Record actual findings below when assembly is complete.

### B1 — Unpowered resistance / continuity screening

Status: `NOT YET RUN`

Using the multimeter, check for obvious shorts between GND and the major rails before first power:

- `VBUS_PROT ↔ GND`
- `SYS ↔ GND`
- `BAT ↔ GND`
- `3V3 ↔ GND`
- `3V3_PROT ↔ GND`

Do not judge capacitor-charging resistance transients as an automatic short; record stable / comparative behavior and investigate only obvious abnormal low resistance.

### B2 — USB-only controlled first power-up

Status: `NOT YET RUN`

Preferred stimulus: compliant 5V USB-C Source connected through `USBC1`.

Initial objective: establish the main power chain with **no battery connected** and no external load.

Measure:

```text
TP_VBUS_PROT → TP_SYS → TP_3V3 → TP_3V3_PROT
```

Expected engineering behavior:

- `VBUS_PROT`: valid USB input rail;
- `SYS`: established by BQ25185 Power Path without a battery;
- `3V3`: regulated by TPS63802 near the 3.3V target;
- `3V3_PROT`: available only if TPS259474 output qualification is satisfied.

Also observe abnormal supply-current limiting, unexpected heating, rail collapse or oscillation. Actual readings must be recorded before any PASS claim.

### B3 — Startup waveform / basic ripple observation

Status: `NOT YET RUN`

Use the oscilloscope to observe, as practical:

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

A no-load / light-load ripple result does **not** validate the 500mA loaded-ripple requirement.

### B4 — Battery-only emulation / BUVLO behavior

Status: `NOT YET RUN`

Use the current-limited bench supply as a battery emulator with USB absent:

```text
Bench PSU + → J_BAT1 / BAT
Bench PSU - → GND
```

Start from a normal 1S battery-range voltage and verify:

```text
BAT → SYS → 3V3 → 3V3_PROT
```

Then reduce the simulated battery voltage gradually and observe:

- `SYS`;
- `3V3`;
- `3V3_PROT`;
- approximate battery-discharge cutoff / BUVLO transition;
- recovery behavior while raising the simulated battery voltage again.

This test validates board-level Battery-only / BQ25185 BATFET-BUVLO behavior without requiring the real battery to be deeply discharged.

### B5 — Real battery pre-connection physical gate

Status: `NOT YET RUN`

Mandatory before first real battery connection:

- verify connector mechanical mating;
- measure actual battery connector polarity with the multimeter;
- confirm actual battery positive wire lands on board `BAT` / J_BAT1 pin 1;
- confirm actual battery negative wire lands on GND / J_BAT1 pin 2;
- verify no ambiguous reversal due to connector housing / cable orientation.

Only after this gate passes may the real battery be connected.

### B6 — Real battery / charging behavior

Status: `NOT YET RUN`

With real battery and compliant USB-C Source:

Observe / measure:

- battery voltage before charging;
- charging authorization behavior;
- `CHG_GATE` and `CE#` if diagnosis is needed;
- actual battery charging current when safely measured in series with the multimeter;
- qualitative CC → CV behavior as battery voltage rises and charge current tapers;
- BQ25185 / board abnormal heating by safe observation; quantitative temperature-rise measurement remains deferred without a proper thermal measurement tool.

Do not infer a verified 500mA charge current from the design resistor alone; record the actual measured value.

### B7 — Power Path operating modes / USB transition

Status: `NOT YET RUN`

Verify separately:

- USB-only operation;
- Battery-only operation;
- USB + Battery operation.

With the oscilloscope on `3V3_PROT` and optionally `SYS`, capture:

- USB insertion while battery-powered;
- USB removal while battery-powered;
- minimum observed output voltage / obvious dropout;
- recovery behavior / abnormal oscillation.

Do not claim zero-drop / seamless transfer unless the waveform supports that statement.

### B8 — Charging qualification / NTC logic

Status: `NOT YET RUN`

Available basic checks with current instruments:

- measure room-temperature NTC resistance;
- observe `NTC_TS` / `CHG_GATE` / `CE#` under normal conditions;
- verify that an intentionally disconnected NTC does not silently remain interpreted as a fully valid charge condition, if this fault-injection test is performed under a safe setup.

Exact hot / cold trip temperature, tolerance stack and pouch thermal lag remain deferred until controlled temperature stimulus and measurement are available. Resistor substitution may later be used to validate threshold logic before heating an actual battery.

### B9 — M5 downstream protection

Status: `DEFERRED — CONTROLLED LOAD NOT AVAILABLE`

Current design functions to be tested later:

- output UV / OV behavior;
- inrush / dV/dt behavior;
- over-current threshold;
- over-current timer;
- short-circuit response;
- latch-off and reset through `TP_M5_RESET`.

Do not use an uncontrolled direct-wire short as a substitute for a proper downstream protection test.

### B10 — Output-load capability

Status: `DEFERRED — ELECTRONIC LOAD / SUITABLE POWER LOAD NOT AVAILABLE`

Still required for final output-performance validation:

- 3.3V / 500mA continuous output;
- loaded output voltage regulation;
- loaded ripple;
- 1A / 100ms design-target transient;
- load-step droop / recovery.

These are not current PASS items.

---

## Priority for the first practical Bring-up session

With the currently available instruments, complete these first:

1. assembly / visual inspection;
2. unpowered rail-resistance screening;
3. USB-only controlled first power-up;
4. measure `VBUS_PROT / SYS / 3V3 / 3V3_PROT`;
5. capture `SYS / 3V3_PROT` startup waveform;
6. battery-only bench-supply emulation and BUVLO sweep;
7. physical battery mating / polarity verification;
8. real-battery charging-current / charging-behavior check;
9. USB-only / Battery-only / USB+Battery operation and USB insertion/removal waveform.

If these are completed with real evidence, the project may legitimately claim board-level Bring-up and validation of the main power / charging / Power Path behavior, while keeping load-capability and controlled-fault items explicitly open.

---

## Bring-up event log

No hardware event has been executed yet.

Future entries should record at minimum:

```text
Date / board ID:
Assembly state:
Test setup / instrument:
Stimulus / operating mode:
Measured node(s):
Measured result / waveform:
Expected behavior:
PASS / FAIL / OBSERVATION:
Issue / diagnosis:
Change made, if any:
Retest result:
Evidence file / photo / oscilloscope capture:
```
