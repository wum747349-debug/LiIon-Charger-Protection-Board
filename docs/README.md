# Stage Documents

This directory contains Stage-enabled and Conditional project documents. Bootstrap does not pre-create them.

| Path | Responsibility | Enable When |
| --- | --- | --- |
| `component_selection_plan.md` | Critical component candidates and decisions | Stage 2 begins |
| `module_design/*.md` | Module connections, calculations, evidence, risks, layout requirements | A module enters Stage 3 detail design |
| `schematic_review.md` | Actual schematic review evidence, issues, status, conclusion | Stage 4 begins |
| `pcb_design_rules.md` | Project rule values, Scope, Priority, configuration status, waiver definitions | Before Stage 5 Layout Preflight |
| `pcb_review.md` | Layout Preflight, routing issues, user DRC summary, waivers, manufacturing release | Layout Preflight is recorded; maintained through Stages 6–7 |
| `bringup_log.md` | Assembly, first power-on, measurements, debug history | Stage 8 assembly or bring-up activity begins |
| `test_report.md` | Test conditions, results, limits, acceptance conclusion | Stage 8 formal testing begins |
| `revision_history.md` | Hardware revisions, reasons, impact, required revalidation | First hardware revision or material change |
| `user/` | User-facing setup, wiring, interface, and safety guidance | The project requires user documentation |

The current project stage is maintained only in the repository root `README.md`. A file's existence never proves that its stage passed.
