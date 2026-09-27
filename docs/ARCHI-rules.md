# Architecture Documentation Rules

[ARCHI.md](ARCHI.md) documents the NOVR architecture as it exists on the fork's `trip` branch
(upstream `dev` plus fork work). After each task (new feature, refactor, bug fix, or an upstream
sync that brings in someone else's change), determine if ARCHI.md needs updating.

## When to Update

Update after ANY change that alters:

- Project structure (new projects, folders or moved files) — §4
- Technology stack (packages, SDK or target framework, game or Unity version) — §3
- Build targets, deploy behaviour or build commands — §6
- Configuration entries or config files — §7
- Startup order or per-frame behaviour of `Core`, `NOVRBehaviour` or `APIBus` — §8
- Patch style, patched game types, or `UIBehaviorPatcher` mappings — §9
- XR startup, the patcher payload or patched assemblies — §10
- Cameras, the URP camera stack, layers or depth behaviour — §11
- Head tracking, calibration or recentring — §12
- HUD placement, sizing, projection or frames — §13
- Menus, panels, map, cursor or hit testing — §14
- Input sources — §15
- Diagnostics, logging or MCP bridge tools and endpoints — §19
- Installer or release packaging — §20
- Data flow between the components in the §21 diagrams
- Testing layers or the compile baseline — §23 (and `docs/4-unit-tests/TESTING.md`)
- The fork's branch model or upstream conventions — §28
- Known issues resolved or discovered — §27

## How to Update by Change Type

### Major Feature / Refactor

Review: §2 Overview, §4 Project structure, §5 Core architecture principles, the subsystem
section(s) touched (§8–§20), §21 Data flow diagrams, §24 Performance considerations, §27 Known
issues, §29 Conclusion

### Minor Feature / Enhancement

Update: the subsystem section touched (§8–§20), §7 Configuration if settings changed, §27 Known
issues if one was resolved

### Bug Fix

Usually no update needed, unless it reveals or fixes an architectural flaw — then update the
subsystem section and §27.

### Dependency Changes

Update: §3 Technology stack, §6 Build system & toolchain, and any affected architectural sections

### Game Update Compatibility

Update: §3 (game and Unity versions), §9 (patched types that moved), the header's "written
against" line, and §27.

### Upstream Sync

When `git merge upstream/dev` into `trip` brings in upstream changes, re-read the upstream diff
and update the sections it touched — ARCHI.md must describe the merged code, not only the fork's
own work. Update the "written against" line in §1.

## Guidelines

- Be precise and factual — reflect the actual codebase, not the design docs (proposals live in `docs/design/` until built)
- Be concise — enough detail to understand, not implementation specifics
- Update diagrams when data flow changes
- Reference actual file paths and type names
- Mark anything inferred rather than verified in-game as such
