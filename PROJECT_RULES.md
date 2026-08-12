# PROJECT_RULES

1. Current Project facts come only from this repository.
2. The Framework version is the Release + Commit recorded in `FRAMEWORK.md`.
3. Do not automatically adopt or read Framework `main`.
4. Framework binding change requires explicit user authorization: prerelease dogfooding uses Pinned Framework Evaluation, backward-compatible adoption uses Compatible Framework Sync, and breaking Runtime / Structural Contract adoption uses Framework Contract Migration; all paths require applicable validation.
5. Verify critical hardware parameters against official datasheets, reference manuals, or application notes. For Rev.A component selection, prefer parts realistically procurable through JLCPCB/LCSC when functional, electrical, safety, thermal, and other hard requirements are satisfied. Marketplace information is procurement / availability evidence only; manufacturer official datasheets / documentation remain the technical qualification authority.
6. `.SchDoc` and `.PcbDoc` are the authoritative EDA implementation sources.
7. AI/Codex must not fabricate EDA, ERC, DRC, manufacturing, bring-up, or test results.
8. Maintain the current Project stage only in the root `README.md`.
9. Another Project cannot be used as this Project's fact source.
10. Read only the Stage Skill and evidence required for the current task.
