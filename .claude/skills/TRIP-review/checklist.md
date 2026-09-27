# Code Review Checklist

This file is the **single source of truth** for code-review criteria. Both human-driven reviews via `.claude/skills/TRIP-review` and Codex-driven reviews via the global `codex-code-review` skill apply the criteria below — referenced, not copied — so the two review surfaces cannot drift.

## Systematic Review Checklist

### 1. Functional Requirements

- [ ] Implementation matches the plan, and the relevant design doc in `docs/design/` where the plan cites one
- [ ] What the player sees in the headset matches what the plan promised
- [ ] Every behaviour the plan promised has an In-Game Verification item, and each is recorded as passed with measured values
- [ ] Edge cases handled: main menu vs flight vs external/spectator/turret cameras; aircraft with and without cockpit screens; scene reloads; floating-origin shifts; desktop resolution and window mode; different headset FOV

### 2. Code Quality

- [ ] Nullable reference types respected; no `!` suppression without a reason next to it
- [ ] DRY — no duplicate of an existing helper (`VrHudProjectionHelper`, `LayerHelper`, `NOVRBehaviour.Create`, `NOVRLog`)
- [ ] KISS — not unnecessarily complex
- [ ] Naming, comment density and idiom match the surrounding code (file-scoped namespaces, upstream's naming); comments explain *why*
- [ ] Canadian English in NOVR's own comments, config descriptions and UI text (`colour`, `centre`, `-ize` verbs); game type names are left as they are
- [ ] No leftover debug logging outside the Verbose Diagnostics gate
- [ ] `using` directives organized, unused ones removed

### 3. Architectural Compliance

- [ ] Follows the principles in ARCHI.md §5: observe and patch, re-parent don't replace, one self-healing `Core`, NOVR rendering on its own layers
- [ ] New patches use `[PatchPrefix]`/`[PatchPostfix]` via `PatchLoader` unless a nested `[HarmonyPatch]` class is justified (ARCHI.md §9)
- [ ] New components derive from `NOVRBehaviour` when they need `OnBeforeRender`, URP frame hooks or live config (`OnSettingChanged`)
- [ ] New settings live in `ModConfiguration` with a section, a default and a description
- [ ] Consistent with the existing codebase style

### 4. Game Integration & Patching

- [ ] Every game type, method and private field used was confirmed to exist in the installed game version (named in the plan)
- [ ] Reflected members are resolved once and cached in `static readonly` fields, never looked up per frame
- [ ] Patches fail soft: a null reflected member or missing object makes the patch return early; nothing throws out of a postfix into the game's method
- [ ] A prefix that skips the original (`return false`) re-implements everything the original did that the game still needs, and says so in a comment
- [ ] No interference with another NOVR patch on the same method (order, double-application)
- [ ] Scene reloads handled — objects the game rebuilds get NOVR's components again (`UIBehaviorPatcher`)

### 5. VR Rendering, UI & Comfort

- [ ] NOVR content renders only on layers 29 (`VrUiClippedHud`), 30 (`VrUi`) or 31 (`VrUiCapture`) through NOVR's cameras; game layers untouched
- [ ] Camera-stack order preserved (`VrClippedHudCamera` directly after `cockpitRenderer`; `VrCockpitHudCamera` last)
- [ ] **UI sizes and offsets are in degrees**, converted at the element's distance — nothing new depends on `Screen.height`, desktop pixels or the CanvasScaler
- [ ] Depth is deliberate: collimated symbology at 1,000 m; panels at or in front of the cockpit surface they overlay; nothing drawn over nearer geometry it sits behind
- [ ] Head-locked content is minimal and smoothed; large panels are cockpit-fixed
- [ ] Text and lines legible on a ~14 px/deg headset (≥10 px cap height, ≥2 px strokes at the configured scale)
- [ ] Stereo correct in both eyes; works with VR zoom

### 6. Error Handling

- [ ] Failures are logged through `NOVRLog` (or the plugin logger) with enough context to act on, once — not every frame
- [ ] Failure modes are graceful: VR keeps running if a HUD patch fails
- [ ] User-facing text (config descriptions, native menu labels) is clear and actionable

### 7. Security

- [ ] Nothing new listens on a network port, or it binds to `localhost` only and is documented (the MCP bridge precedent)
- [ ] No new downloads or file writes outside `BepInEx/` and the patcher's documented targets
- [ ] No behaviour that affects other players in multiplayer or could read as a cheat

### 8. Performance

- [ ] Fits the VR frame budget (11.1 ms at 90 Hz, 12.5 ms at 80 Hz): no per-frame allocations, `Find*`/`GetComponentsInChildren` calls or scene scans in hot paths
- [ ] No new camera, render pass or canvas without the plan justifying its cost
- [ ] Per-marker or per-unit work scales acceptably with many visible units
- [ ] Resources created at runtime (materials, textures, GameObjects) are destroyed with their owner

### 9. Upstream Mergeability

- [ ] Code commits touch nothing under `docs/` or `.claude/`; TRIP commits touch nothing else
- [ ] `NOVRPlugin`'s version string unchanged; no tags
- [ ] Diff is the smallest that achieves the plan — no drive-by reformatting of upstream code
- [ ] Upstream-facing docs (`README.md`, `AGENTS.md`, config descriptions) updated where the change affects them
- [ ] The `pr/` branch builds on current `upstream/dev`

---

## Issue Severity Classification

**Critical (Block Deployment)**:

- Anything that can break XR startup, crash the game or corrupt game files
- A patch that can throw into the game's own method
- Security exposure (open network port, unsafe file writes)
- TRIP files in a code commit bound for upstream

**Major (Require Immediate Fix)**:

- Incorrect behaviour against the plan or design doc
- Missing fail-soft handling on reflected game internals
- A significant frame-time regression, or new per-frame allocations or scans
- UI sized in pixels or tied to `Screen.height`
- Compilation/build errors, or new warnings in touched files
- A promised behaviour without a passed In-Game Verification item

**Minor (Should Fix)**:

- Code style inconsistencies with the surrounding code
- Missing or stale config descriptions or docs
- Code duplication
- Missing edge-case handling that does not break VR

**Suggestions (Nice to Have)**:

- Performance optimizations
- Readability improvements
- Moving logic into a pure, unit-testable helper

---

## Review Completion Criteria (Approval Gate)

Minimum for approval:

- [ ] All functional requirements implemented
- [ ] No critical or major issues remaining
- [ ] Compile-only builds succeed with 0 errors and no new warnings in touched files:
      `dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath=` (plus `NOVR.McpBridge` / `NOVR.Patcher` if touched)
- [ ] Affected unit tests pass, if `tests/NOVR.Tests` covers the change:
      `dotnet test tests/NOVR.Tests/NOVR.Tests.csproj --filter "FullyQualifiedName~<pattern>"`
- [ ] In-Game Verification recorded as passed in the plan (measurements, headset checks, log check)
- [ ] New pure logic has unit tests, or a coverage-debt ledger entry per the hard-to-cover policy
- [ ] Documentation updated per project standards (ARCHI.md via ARCHI-rules.md; upstream docs in the code commit)
