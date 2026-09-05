# Project Rules

Project: LiIon Charger Protection Board
Repository: `wum747349-debug/LiIon-Charger-Protection-Board`
Default branch: `main`

## Authority

This standalone repository is the only active project authority. The legacy source repository is frozen migration history and must not be used to overwrite current project facts.

## Evidence hierarchy

When sources conflict, use the latest current-project implementation and evidence in this order:

1. current authoritative EDA implementation / manufacturing output / hardware evidence;
2. current project documentation synchronized to that implementation;
3. older handoff or historical records;
4. legacy migration source.

Do not promote a design target, datasheet capability, calculated value, planned test or expected behavior to measured / verified PASS without actual evidence.

## Stage 8 execution

For Rev.A Assembly / Bring-up, `docs/bringup_log.md` is the execution authority for the approved Gate-driven sequence and the append-only-style evidence log. Do not skip required Gates. Record deviations, failures, fixes and retests rather than rewriting the approved plan after the fact.

The real battery must not be connected until physical mating, board pin order and actual battery polarity / wire order are verified with a multimeter. Deferred performance / fault items remain `DEFERRED / NOT YET VERIFIED` until suitable test equipment and hardware evidence exist.

## Change discipline

Keep documentation changes narrowly scoped, preserve unrelated user work, and avoid unnecessary Framework / Stage / structure changes. Repository writes must remain consistent with the current Project authority and evidence boundary.