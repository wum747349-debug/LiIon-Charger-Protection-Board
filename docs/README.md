# Project Documentation Guide

This directory contains the active engineering records for the standalone LiIon Charger Protection Board project.

## Current documentation authority

- `../requirements.md` — requirements baseline and acceptance boundaries.
- `../block_diagram.md` — system-level architecture.
- `../design_notes.md` — detailed design reasoning and constraints.
- `component_selection_plan.md` — critical component selection evidence and alternates.
- `project_design_overview.md` — system design / project explanation document.
- `module_design/` — Stage 3 module-level circuit design records.
- `schematic_review.md` — Stage 4 formal schematic review record.
- `pcb_design_rules.md` — PCB design-rule / implementation baseline.
- `pcb_review.md` — PCB placement, routing, DRC and Stage 7 manufacturing-release review record.
- `bringup_log.md` — Stage 8 approved execution plan plus actual Assembly / Bring-up evidence log. The current Rev.A sequence is Gate-driven (`B0/B1 → G1 → B2A/B2B → B3/B4 → G2 → B5/B6/B7/B8 → G3`). Planned tests are not PASS; execution evidence must be appended to the log.
- `test_report.md` — reserved for formal measured hardware-test reporting once real test results exist; do not create or populate it with design expectations alone.

## Stage 8 documentation rule

During Assembly / Bring-up:

1. use `bringup_log.md` as the execution authority for the approved test sequence;
2. record actual readings, waveforms, failures, fixes and retests in its Execution log;
3. keep deferred items explicitly `DEFERRED / NOT YET VERIFIED` until suitable equipment and real evidence exist;
4. only create / update formal test reporting when hardware measurements actually support the claim.

The repository must continue to distinguish design targets, datasheet capability and engineering expectations from measured hardware evidence.