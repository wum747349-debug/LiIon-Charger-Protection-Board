# Project Rules

Project: LiIon Charger Protection Board
Repository: wum747349-debug/LiIon-Charger-Protection-Board

## Scope

These rules apply only to this standalone project repository.

## Authority

This repository is the only active project authority for the current hardware project.

The legacy migration source is historical only. It must not be used as a live authority unless the user explicitly asks for historical comparison or recovery.

## Current implementation wins

When older documentation, handoff notes, archived BOM data, previous component records, or migration-era assumptions conflict with the current `.SchDoc`, `.PcbDoc`, synchronized project documents, or current manufacturing / hardware evidence, use the current authoritative implementation.

Do not silently reintroduce superseded component choices, pin mappings, layout assumptions, or old Stage conclusions.

## Evidence boundary

Do not claim any of the following without matching evidence:

- ERC PASS;
- DRC PASS;
- assembly complete;
- first power-on complete;
- rail measured / verified;
- charger function verified;
- battery protection verified;
- NTC temperature threshold verified;
- 500mA continuous output verified;
- 1A / 100ms target verified;
- manufacturing or hardware PASS beyond the evidence actually recorded.

Design intent, datasheet capability, calculations, simulation, planned tests and measured hardware results must remain clearly distinguished.

## Change discipline

- Keep changes scoped to the requested task.
- Preserve unrelated user work.
- Do not modify Framework binding, Stage, gate state, project structure version or repository architecture unless the task explicitly requires it.
- Prefer updating existing authoritative records over creating duplicate documents.
- Do not create synthetic test results to make documentation look complete.
