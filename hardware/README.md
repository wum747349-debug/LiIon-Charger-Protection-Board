# Hardware Sources and Evidence

Create hardware subdirectories only when the corresponding work or evidence exists:

- `altium_project/`: authoritative `.PrjPcb`, `.SchDoc`, `.PcbDoc`, and necessary project libraries;
- `outputs/`: traceable schematic PDF, BOM, reports, and manufacturing outputs;
- `images/`: review, assembly, bring-up, test, and presentation images.

`.SchDoc` and `.PcbDoc` are the authoritative EDA implementation sources. A directory, PDF, image, or written design intent does not prove ERC, DRC, rule matching, copper state, or manufacturing release. The user performs actual Altium operations and exports; AI/Codex only analyzes supplied evidence within stated limits.

Do not create empty output categories or low-information placeholder files merely to make the tree look complete.
