---
name: TRIP-test
description: Write/run tests following project standards (deep test authoring)
disable-model-invocation: true
argument-hint: "component or feature to test"
---

# Testing Mode

You are now in **testing mode** for **NOVR** (the Nuclear Option VR mod, fork `CMartinChan/novr`).

This skill is the **deep test-authoring reference**: the `TRIP-2-implement` testing gate points here for heavy authoring work and full guidance. Invoke it standalone for test backfill, verification work, or coverage work outside an implementation session.

## Prerequisites - Read First

Before testing, you MUST read:

1. @docs/ARCHI.md - Understand system architecture (§23 Testing strategy, §19 Diagnostics & tooling)
2. @docs/4-unit-tests/TESTING.md - Testing guidelines

## Your Task

Test: $ARGUMENTS

---

## Testing Guidelines

NOVR runs inside a closed-source game. Most of its code only means anything in the running game,
so verification has **three layers**, cheapest first:

1. **Compile gate** — the compiler is the type check. Compile-only builds (never deploying).
2. **Unit tests** — only for **pure logic**: code with no `UnityEngine`, Harmony or game-type
   dependency (angle maths, layout tables, parsers, config value validation).
3. **In-game verification** — the running game, the headset, and the MCP bridge for measured
   checks. This is where patches, cameras and UI placement are proven.

### Scope

- Only run tests for code that changed (not everything)
- Focus on the new feature/fix/refactor

### Commands

```bash
# Compile gate (no deploy). Baseline on dev@2a0f789: 0 errors, 396 warnings on NOVR.
dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath=
dotnet build NOVR.McpBridge/NOVR.McpBridge.csproj -c Release -p:GameDeployPath=   # if touched
dotnet build NOVR.Patcher/NOVR.Patcher.csproj -c Release -p:GameDeployPath=       # if touched

# Unit tests — once tests/NOVR.Tests exists (it does not yet)
dotnet test tests/NOVR.Tests/NOVR.Tests.csproj                                          # all
dotnet test tests/NOVR.Tests/NOVR.Tests.csproj --filter "FullyQualifiedName~<pattern>"  # specific
dotnet test tests/NOVR.Tests/NOVR.Tests.csproj --collect:"XPlat Code Coverage"          # coverage

# Deploying build for in-game checks (game CLOSED; replaces the installed NOVR)
dotnet build NOVR/NOVR.csproj -c Release

# In-game measurements (game running with NOVR loaded)
curl -s http://localhost:3334/tools
curl -s -H "Content-Type: application/json" \
     -d '{"tool":"inspect_gameobject","args":{"name":"<object>","includeComponents":true}}' \
     http://localhost:3334/invoke
```

⚠️ Never pass `--no-build` to `dotnet test` on a tree you have not just built.

### Test Structure

- **No test project exists yet.** The first feature that adds pure logic creates
  `tests/NOVR.Tests/NOVR.Tests.csproj`:
  - `net9.0`, xUnit (the SDK installed here is .NET 9).
  - **Compile-links** the pure source files instead of referencing `NOVR.dll`
    (`<Compile Include="..\..\NOVR\<path>\<File>.cs" Link="Linked\<File>.cs" />`). `NOVR.dll` is
    `net48` and references Unity and game assemblies that cannot load in a test process.
  - Therefore pure helpers must avoid `UnityEngine` types (use `float`/`double`, `System.Math`,
    small local structs), and live in their own files so they can be linked.
  - Add the project to the solution only if upstream wants it; otherwise keep it as a fork-side
    project that the upstream PR branch does not include (discuss in the plan's Upstream PR section).
- Test classes: `<Subject>Tests.cs`, one per helper; method names state the behaviour
  (`DegreesToWorldUnits_AtOneKilometre_MatchesTangent`).
- In-game results are recorded in the plan's **In-Game Verification** section, not in test files.

### Testing Priorities

**Unit Tests** (pure logic only):

- Angle and size conversions (degrees ↔ world units at a distance; cap height ↔ font size)
- Layout tables (positions in degrees for each HUD element and frame)
- Projection maths extracted into pure helpers (direction → azimuth/elevation, clamping to a window)
- Config value validation and clamping

**In-Game Verification** (the integration layer):

- Measured through the MCP bridge: canvas scale and position, element transforms, camera-stack
  order, layer assignment, component presence after a scene reload
- Headset checks by the user: legibility, placement, depth comfort, occlusion by the cockpit,
  stereo correctness, behaviour in each camera state
- Log: no new exceptions, Harmony errors or `NOVR:` errors in `Player.log`

**What to Test**:

- Happy path in the scenario the plan names
- Missing or renamed game objects (fail soft)
- Boundary values: pitch ±90°, targets behind the viewer, off-window targets, extreme desktop
  resolutions (1080p and 4K), headset FOV differences
- Scene transitions: menu → flight → menu, respawn, aircraft change

---

## Hard-to-Test Code

Seam ladder, cheapest first: **pure helper (compile-linked into tests) → in-game measurement through the MCP bridge → headset check by the user**. Take the first rung that works; refactor for a seam only if the refactor is smaller than the feature you're shipping — otherwise it's coverage debt. Before refactoring legacy code, pin its current behaviour with in-game measurements (record the values), then refactor and re-measure.

Uncovered risky paths: one line each in `docs/4-unit-tests/COVERAGE-DEBT.md` (`path | why hard | escape plan`). Delete a ledger line in the same change that gives its path meaningful coverage.

---

## Post-Testing Summary

After completing tests, create a summary file:

**File**: `docs/4-unit-tests/wa_F_nnn_test.md`
(a = project week, F_nnn = release ID)

**Content**:

```markdown
# Test Summary - Week a, F_nnn

## What Was Tested

[List of tested components/functions, and the in-game scenario]

## Test Results

- Compile gate: 0 errors, <W> warnings (baseline 396)
- Unit tests: X total, X passed, X failed (or "none — no pure logic")
- In-game checks: X/Y passed (measurements and headset checks)
- Coverage: <X% for tests/NOVR.Tests, or "not measured">

## Key Findings

[Any issues discovered, edge cases found, measured values worth keeping]

## Notes

[Headset and game version used, anything that could not be verified, recommendations]
```
