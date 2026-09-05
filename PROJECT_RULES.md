# Project Rules

Project: `LiIon-Charger-Protection-Board`

## Scope

These rules apply only to this standalone project repository.

## Project Authority

This repository is the only active authority for this project.

The legacy migration source remains frozen and must not be used as an active design or documentation authority.

## Working Rules

- Work only inside this repository unless the user explicitly requests cross-repository work.
- Do not modify the Framework Repository from project-only tasks.
- Do not change `FRAMEWORK.md` binding unless the task is explicitly a Framework migration / repin.
- Preserve current Project Stage unless the task explicitly performs a valid Stage transition.
- Treat current authoritative EDA implementation and synchronized project documentation as the project baseline; when older handoff notes conflict with the current implementation, the current implementation wins.
- Do not claim hardware verification, measured PASS, ERC PASS, DRC PASS, manufacturing PASS or Stage completion without matching evidence.
- Keep design targets, datasheet capabilities, calculations, planned tests and measured hardware results clearly separated.
- For Stage 8, do not connect a real battery until actual mating, polarity and wire order are physically verified with a multimeter.
- Keep unrelated user work untouched.
- Do not commit or push unless the user explicitly requests repository changes.

## Documentation Discipline

- Use project documents for persistent engineering facts.
- Prefer updating an existing authoritative document over creating overlapping duplicate records.
- Keep `README.md` as a concise current-state / navigation summary, not a full engineering notebook.
- Record actual Bring-up measurements, failures, fixes and retest evidence in the active Stage 8 Bring-up / test record.
- A planned test is not a test result.
