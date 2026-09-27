---
name: TRIP-1-plan
description: Plan a new feature following project standards
argument-hint: "describe the feature you want to build (add --speedrun to chain straight into implementation)"
---

# Planning Mode

You are now in **planning mode** for **NOVR** (the Nuclear Option VR mod, fork `CMartinChan/novr`).

This is the project-local copy of `TRIP-1-plan`. It replaces the global template.

## Prerequisites - Read First

Before creating any plan, you MUST read ALL THE LINES of:

1. @docs/ARCHI.md - Understand current system architecture
2. `docs/design/hud-redesign.html` — **only if the feature touches the HUD, helmet display, map,
   cockpit panels or VR UI sizing.** It is the approved design reference for that work. Read the
   parts the feature touches (the spec table, the HUD window and the map and panels sections) and
   keep to its angles unless the plan says why not.
3. `AGENTS.md` — **only if the feature changes the build, the patch conventions, the installer or
   anything upstream's PR guidelines cover.** It is upstream's contributor guide; ARCHI.md §27
   lists where it has drifted from the code.

## Your Task

Plan the following feature: $ARGUMENTS

**Speedrun**: if the arguments contain `--speedrun`, strip the flag from the feature description and run in speedrun mode — Step 4's approval question is skipped and the plan chains directly into `TRIP-2-implement` once Codex returns `APPROVED`. `NEEDS_REWORK` always cancels speedrun and falls back to the normal Step 4 question. Discovery questions (Step 1) still run — speedrun removes the end gate, not the understanding phase.

---

## Step 1: Discovery & Clarification (Interactive)

**Do NOT start writing a plan immediately.** First, engage in a discovery conversation to fully understand the user's intent.

### 1.1 Initial Understanding

After reading the feature request, summarize your understanding in 2-3 sentences, then **use the `AskUserQuestion` tool** to present clarifying questions with structured options.

Frame questions around:

- **Scope**: What's included vs excluded?
- **Behavior**: How should it work from the user's perspective — in the headset?
- **Constraints**: Game version, headset (Chris uses a Rift S, ~14 px/deg), upstream acceptance
- **Priority**: What's most important if trade-offs are needed?

For each question, provide 2-4 concrete options based on your analysis of the codebase and the feature request. Always let the user provide custom input via the built-in "Other" option.

After the user answers, proceed **directly to writing the plan** (Step 2) — no approach-confirmation question. Ask a follow-up round with `AskUserQuestion` only if a blocking ambiguity remains (**maximum 3 rounds total**; if still unclear, summarize what you know and proceed with noted assumptions).

### 1.2 Verify against the game before planning

NOVR patches a closed-source game. Before naming any game type, field or method in the plan,
confirm it exists in the **installed** game:

- Reflection over `D:\SteamLibrary\steamapps\common\Nuclear Option\NuclearOption_Data\Managed\Assembly-CSharp.dll`
  from PowerShell (`[Reflection.Assembly]::LoadFrom` plus an `AssemblyResolve` handler for the
  `Managed` folder) lists types and fields.
- `ilspycmd` (see `AGENTS.md`) decompiles method bodies. It is not installed on this machine;
  ask before installing it, since it is a NuGet download.
- If the plan depends on runtime values (sizes, positions, hierarchy), measure them first through
  the MCP bridge with the game running (ARCHI.md §19). A plan built on an unmeasured assumption
  must say so in its Problem Statement.

---

## Step 2: Plan Document Creation

Once understanding is confirmed, create the plan document.

### File Naming

**This fork does not version releases — upstream owns NOVR's version and tags** (ARCHI.md §28).
Plans are numbered in a fork-local sequence instead. Take the next free number after the highest
`F_nnn` in `docs/1-plans/` and in `docs/2-changelog/changelog_table.md` (`F_000` is the TRIP
initialization), and create:

`docs/1-plans/F_[nnn]_[feature-name].plan.md` — for example `F_001_hud-angles.plan.md`

The release ID `F_nnn` replaces `x.y.z` everywhere downstream (code review, changelog, test
summary).

### Required Sections

```markdown
# [Feature Name] Implementation Plan

**Release**: F_nnn
**Upstream base**: `upstream/dev` @ <short sha> (game version <x.y.z>)

## Overview

[2-4 sentences describing the feature and its purpose]

## Problem Statement (if applicable)

[Current limitations/issues this feature addresses. Say which facts were measured in-game and which are inferred from code.]

## Solution Architecture

[High-level design approach]

## Implementation Details

### 1. [Component/Module/File Name]

**File**: `path/to/file`

[Detailed description of changes needed]

**Current state** (if modifying existing):
[Describe what currently exists]

**Modifications**:

- Specific change 1 (around line X)
- Specific change 2 (around line Y)

### 2. [Next Component/Module/File]

[Continue with same pattern]

## Technical Considerations

- **Pattern Usage**: Which existing patterns to follow (ARCHI.md §5 principles; new patches use `[PatchPrefix]`/`[PatchPostfix]` with `PatchLoader` unless a nested `[HarmonyPatch]` class is needed, ARCHI.md §9)
- **Game Internals**: Every game type, method and private field the change touches, and confirmation it exists in the installed game version. What happens when a game update renames it (must fail soft — no throwing postfixes)
- **Rendering & Layers**: Which layer (29 clipped HUD / 30 VR UI / 31 capture) and which camera draws it; depth behaviour (hidden by cockpit or drawn on top); position in the URP camera stack (ARCHI.md §11)
- **Angular Sizing & Comfort** (UI/HUD work): sizes and offsets in degrees, never desktop pixels or `Screen.height`; which frame the element lives in (world, aircraft, head, cockpit panel); its depth; how much is head-locked
- **Frame Budget**: 11.1 ms at 90 Hz, 12.5 ms at 80 Hz (Rift S). No new per-frame allocations, `Find*` calls or scene scans in hot paths
- **Configuration**: new `ModConfiguration` entries (section, name, default, description), and whether they apply live through `OnSettingChanged`
- **Multiplayer**: client-side only; no change that affects other players or could read as a cheat
- **Upstream Mergeability**: smallest diff that works; matches upstream style (file-scoped namespaces, nullable, existing naming); no TRIP files in code commits
- **Edge Cases**: menus vs flight vs external/spectator cameras; aircraft with and without cockpit screens; desktop resolution and window mode; headset FOV and PPD; floating-origin shifts; scene reloads (UIBehaviorPatcher re-attaches components)

## Files to Modify/Create

[Comprehensive numbered list with purposes]

1. `path/to/file1` (modify) - Purpose description
2. `path/to/file2` (new) - Purpose description

## Type Definitions (if applicable)

[New types, interfaces, structs, or modifications to existing ones]

## Performance & Cost Impact (if applicable)

[Expected per-frame cost; any new camera, render pass or canvas]

## Backward Compatibility (if applicable)

[Config migration for existing players' `deltawing.novr.cfg`; behaviour for players who never touch new settings]

## Test Impact

[2-5 bullets: which pure-logic parts get unit tests (see `TRIP-test` — only code with no UnityEngine dependency), what the compile gate covers, and that the In-Game Verification section below is the integration check. No test code — the TRIP-2 testing gate consumes this section.]

## In-Game Verification

[How to prove the change in the running game. The TRIP-2 testing gate executes this section.]

- **Setup**: what to load (main menu / free flight in <aircraft> / a mission) and any config to set
- **Measurements** (MCP bridge, `POST http://localhost:3334/invoke`): tool + args, and the expected value or range for each (for example, a canvas `localScale`, an element's angular position)
- **Headset checks**: what Chris looks at and the pass criterion for each, phrased so the answer is yes/no
- **Log check**: no new exceptions or `NOVR:` errors in `Player.log`

## Upstream PR

- **Target**: `InfernoSuperNova/novr` `dev`
- **Title**: `[Area] Brief description` (upstream convention)
- **Contents**: the code paths that go upstream (everything outside `docs/` and `.claude/`)
- **Stays in the fork**: this plan, the review, changelog and any design docs
- **Coordination**: related upstream issues/branches, and whether to open an issue before the PR

## Documentation Impact

[Mandatory. List every document OUTSIDE the TRIP docs that this feature will leave outdated, with one line each on what becomes stale. If none are affected, write "None". The TRIP-3 Documentation Sync step consumes this section before the release commit. Always evaluate the candidates below.]

- `README.md` — affected when install steps, features, supported headsets/runtimes or user-facing behaviour change
- `AGENTS.md` — affected when build commands, project structure, patch conventions, troubleshooting or PR guidelines change (upstream's file: factual corrections only)
- `ModConfiguration.cs` entry descriptions — user-facing text shown in the config file; affected when a setting's meaning, range or default changes
- `docs/design/hud-redesign.html` — fork-only design reference; affected when the built HUD deliberately departs from its spec (record the departure, don't silently diverge)
- `NOVR.XR.OpenXR/MockRuntime/README.md` — vendored Unity doc; affected only if the mock runtime itself changes

## To-dos

### Phase 1: [Phase Name] (if multiple phases are needed) or simply skip title if only one phase is needed

- [ ] Task description
- [ ] Another task

### Phase 2: [Phase Name] (if applicable)

- [ ] Task description
- [ ] Another task

**Note**: For simple plans, a single phase is sufficient. Split into multiple phases only for complex features requiring sequential implementation.

**Note**: Do NOT write test code during planning — the Test Impact section above only names what the TRIP-2 testing gate will run and author.
```

## Quality Standards

- **Zero Ambiguity**: Every step must be clear and actionable
- **File-Level Specificity**: List exact files and functions to modify
- **Architecture Alignment**: Must conform to existing patterns in ARCHI.md
- **Risk Assessment**: Highlight potential failure points — especially game-update fragility
- **Verifiable**: every behaviour the plan promises appears in In-Game Verification with a pass criterion

---

## Step 3: Codex Second-Opinion Review

Before the user sees the plan, run the Codex plan review loop. **Always run it — no confirmation question.** The user gets exactly one decision point in this skill, and it comes after the plan is reviewed (Step 4).

The Codex scripts live in the global skills folder (`$HOME/.claude/skills/codex-*`); this project has no local copies. Run them from the repository root, in the background, with `CODEX_TIMEOUT=1800`.

### Loop

1. **Start**: `bash "$HOME/.claude/skills/codex-plan-review/scripts/start.sh" --prompt-file "$HOME/.claude/skills/codex-plan-review/prompts/start.tpl" <plan-path>`
2. **Parse trailing tag**: `APPROVED` -> Step 4. `NEEDS_REWORK` -> surface to user. `REQUEST_CHANGES` -> continue.
3. **Address findings critically** — quote each P1/P2, push back on incorrect ones, fix legitimate ones by editing the plan in place.
4. **Write implementer notes** (1-3 sentences): which findings you fixed, which you pushed back on and why, any user decisions that override existing docs or environment limitations that can't be resolved in the plan.
5. **Resume** with notes:
   ```bash
   bash "$HOME/.claude/skills/codex-plan-review/scripts/resume.sh" \
       --prompt-file "$HOME/.claude/skills/codex-plan-review/prompts/resume.tpl" \
       --notes "Fixed X. Pushed back on Y because Z. User decided W." \
       <plan-path>
   ```
   -> back to step 2.
6. **No cap** — keep iterating until Codex returns `APPROVED`.

Surface Codex reviews verbatim. Keep edits scoped to findings. Reset thread (`bash "$HOME/.claude/skills/codex-plan-review/scripts/reset.sh" <plan-path>`) only if context is genuinely confused.

Codex cannot run the game or the MCP bridge. When it asks for runtime facts, answer from measurements already taken or mark them as In-Game Verification items — do not invent values.

---

## Step 4: User Review

After the Codex review converges, present a summary:

- **Feature**: [name] (F_nnn)
- **Approach**: [1-2 sentences]
- **Files affected**: [count] files ([list key ones])
- **Estimated complexity**: [simple/moderate/complex]
- **Codex status**: [APPROVED after N rounds / NEEDS_REWORK surfaced to you]

**Speedrun mode**: present the summary above (so the record exists), then skip the question and proceed directly into `TRIP-2-implement` as if the user had answered "Approved — implement now". (A `NEEDS_REWORK` Codex status always cancels speedrun — ask the question normally.)

Otherwise, **one `AskUserQuestion`** — the single decision point of this skill:

- **Question**: "Review the plan at `docs/1-plans/F_nnn_feature-name.plan.md`. How to proceed?"
- **Options**:
  - "Approved — implement now" → continue straight into `TRIP-2-implement` with this plan
  - "Approved — stop here" → plan saved, no implementation
  - "Rework" → the user provides feedback as text

Handle the answer:

- **Rework**: update the plan from the user's feedback, then re-present. Run another Codex pass if the changes are substantive.
- **Other (custom input)**: handle accordingly.

Approval and the implement-now decision are one question on purpose — approving a plan and choosing when to build it is a single thought, and splitting it into two prompts buys nothing.

---

## IMPORTANT: No Code Implementation

**DO NOT write code snippets or implement anything during planning.**

This is a high-level planning phase only. Your plan should describe:

- WHAT needs to be done (features, changes, structures)
- WHERE changes will happen (files, modules, functions)
- WHY certain approaches are chosen (trade-offs, rationale)

But NOT:

- Actual code implementations
- Detailed algorithm code

Keep it architectural and descriptive. Code comes in the `TRIP-2-implement` phase.

## For a HUD or helmet-display change

Required analysis:

- Which game objects are affected (`FlightHud`, `HUDCenter`, `HMDCenter`, `CombatHUD`, `HUD*State`) and which NOVR behaviour or patch places them (ARCHI.md §13)
- The frame each element lives in (world-conformal, aircraft/HUD window, head, cockpit panel) and its angular position and size, taken from `docs/design/hud-redesign.html` where it applies
- How sizes convert from degrees to world units at the element's distance (`2·d·tan(θ/2)`), and that nothing depends on `Screen.height`
- Conformal elements: the projection path (world → main camera → UI camera → normalize × 1,000) and what happens when the target is behind or off-view
- Depth: clipped layer 29 (hidden by the cockpit) or overlay layer 30 (always visible)
- Legibility on a 14 px/deg headset (Rift S): text cap height and stroke width in pixels

## For a new or changed Harmony patch

Required analysis:

- Target type and method, and how that was confirmed in the installed game (Step 1.2)
- Prefix vs postfix, and whether a prefix skips the original (returning `false`) — and why that is safe
- Private fields read through `AccessTools`, cached in `static readonly` fields
- Behaviour when a reflected member is null (return early, never throw from a postfix)
- Registration style: `[PatchPrefix]`/`[PatchPostfix]` via `PatchLoader`, or a nested `[HarmonyPatch]` class, and why
- Interaction with existing patches on the same method

## For a menu or panel change (native VR UI, map, gameplay canvases)

Required analysis:

- Which surface: `NativeVrUiRoot` panels, patched game canvases (`NOVRGameplayUIBehaviour`, `NOVRMainMenuBehavior`) or the map (`DynamicMap` patches)
- Placement frame (cockpit-fixed, gaze-summoned, head-locked) and distance, and that nothing interactive sits behind nearer geometry
- Cursor path: canvas registered with `VrCanvasHitTester`; works in Mouse and Controller modes (`VrUiCursor`)
- Recentre behaviour (measured against the cockpit forward, never the playspace)

## For camera, rendering or XR changes

Required analysis:

- Which camera (tracked `NOVR Main Camera`, `VrClippedHudCamera`, `VrCockpitHudCamera`, game overlay cameras) and its position in the URP stack
- Stereo correctness in both eyes; interaction with VR zoom (`XRPassZoomPatch`)
- Effect on each camera state (cockpit, orbit, selection, turret, menu)
- OpenXR runtime differences (Oculus, SteamVR, WiVRn) where the change touches XR startup

## For input and cursor changes

Required analysis:

- Mouse mode (virtual mouse device) and Controller mode (OpenXR controllers) both covered
- Per-frame edge detection stays idempotent (`VrControllerInput`)
- No conflict with the game's Rewired bindings for flight controls

## For configuration changes

Required analysis:

- Section, key name, default, range and description text
- Live application through `NOVRBehaviour.OnSettingChanged`, or restart required (say which)
- Effect on existing players' config files

## For patcher, installer or build changes

Required analysis:

- Which project and target framework; whether it builds with the .NET 9 SDK (`NOVR.Installer` needs .NET 10)
- Effect on `DeployToGame` (it writes into the real game folder)
- Files the patcher copies into `NuclearOption_Data` on every launch, and locked-file behaviour
