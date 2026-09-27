---
name: TRIP-2-implement
description: Implement a feature following TRIP plan
argument-hint: "plan file or feature to implement"
---

# Implementation Mode

You are now in **implementation mode** for **NOVR** (the Nuclear Option VR mod, fork `CMartinChan/novr`).

This is the project-local copy of `TRIP-2-implement`. It replaces the global template.

## Prerequisites - Read First

Before implementing, you MUST read ALL THE LINES of:

1. @docs/ARCHI.md - Understand current system architecture

## Your Task

Implement: $ARGUMENTS

---

## Step 0: Sync and Create a Branch (Pre-Implementation)

**Branch model (ARCHI.md §28).** `trip` is the fork-only integration branch: upstream `dev` plus
the TRIP files (`docs/`, `.claude/`). Feature work happens on `session/<short-description>`
branches cut from `trip`. Upstream pull requests are built later, by `TRIP-3-release`, from
`upstream/dev` with the code commits only.

**Always** do this before implementing — no need to ask:

```bash
git fetch upstream
git checkout trip
git merge upstream/dev        # sync: the ONLY merge commits allowed on trip
git checkout -b session/[short-description]
```

- If `git merge upstream/dev` is a no-op, fine. If it conflicts, stop and surface the conflict to
  the user — do not resolve upstream conflicts silently.
- Derive the short description from the plan/feature name.
- If already on a `session/` branch for this work (e.g., resuming a session), continue on it.

**Commit discipline for the whole feature**: a commit touches either code (anything outside
`docs/` and `.claude/`) or TRIP files (`docs/`, `.claude/`) — **never both**. TRIP-3 relies on
this to build a clean upstream PR. Implementation itself makes no commits (see step 4 below).

---

## Implementation Phase — Delegate to Codex

You do NOT write the implementation yourself — delegate it to Codex via the `codex-implement` skill. (Exception: trivial unplanned changes of a few lines may be done directly.)

Delegation is **batched**: Codex implements a few of the plan's checkboxes per turn, you review and fix each batch, then request the next one with your corrections attached. Same persistent thread throughout — context and conventions compound across turns.

The Codex scripts live in the global skills folder (`$HOME/.claude/skills/codex-*`); this project has no local copies. Run them from the repository root, in the background, with `CODEX_TIMEOUT=1800`.

### 1. Read the plan and decide the batches

Read the plan fully and split its to-dos into batches. You are the judge of batch size:

- A batch is the **smallest set of checkboxes that leaves the tree green** (compiles). Never split an interface from its implementation and wiring.
- Target a reviewable diff — roughly ≤300 changed lines per batch. A checkbox that alone exceeds this becomes its own batch.
- Size by risk: novel, architectural, or game-integration work (new patches, reflection, camera stack) → small batches (down to one checkbox). Mechanical, repetitive work → larger batches.
- Never span phase boundaries.
- **One-shot escape hatch**: a low-risk plan (or phase) of ≤3-4 checkboxes is delegated whole — no batching ceremony.
- **Filter out non-Codex items**: checkboxes needing human input, the running game, the headset or the MCP bridge are yours — resolve them with the user before or between batches, never delegate them. Codex cannot run Nuclear Option.

### 2. Delegate batch by batch

**Start** the session with the first batch (state dir is handled by the script):

```bash
bash "$HOME/.claude/skills/codex-implement/scripts/start.sh" \
    --prompt-file "$HOME/.claude/skills/codex-implement/prompts/implement.tpl" \
    <plan-path> "Implement only: <batch-1 checkboxes>"   # or omit instructions to one-shot a small plan
```

**Each next batch resumes the same thread**, carrying your review corrections as `--notes`:

```bash
bash "$HOME/.claude/skills/codex-implement/scripts/resume.sh" \
    --prompt-file "$HOME/.claude/skills/codex-implement/prompts/continue.tpl" \
    --notes "<what you fixed after the last batch and why; conventions to apply from now on>" \
    <plan-path> "Now implement: <next batch checkboxes>"
```

Tell Codex in the first batch's instructions: **compile-only builds use
`-p:GameDeployPath=`** — a plain `dotnet build` writes into the real game folder.

**Parse the trailing tag** of each report:
- `IMPLEMENTATION_COMPLETE` → review the batch (below).
- `IMPLEMENTATION_PARTIAL` → read the report; resume with instructions for the remainder, or finish small leftovers yourself during the batch review.

### 3. Review each batch (delta review)

After each Codex report, before requesting the next batch:

1. **Review the delta only**: `git status -s && git diff` — worktree vs index shows just this batch, since previous batches are staged (step 4). Check it against the plan, ARCHI.md patterns, and project conventions: patches fail soft (return early on null, never throw from a postfix), reflected members cached in `static readonly` fields, NOVR rendering stays on layers 29–31, sizes in degrees not pixels, no per-frame allocations or scans, file-scoped namespaces and nullable annotations like the surrounding code.
2. **Fix problems directly yourself** — no back-and-forth with Codex over fixes. What you fixed and why becomes the `--notes` of the next resume.
3. **Micro-gate**: run the compile-only build from the Testing Gate (step 1 only). Fix errors and new warnings now.
4. **Checkpoint**: `git add -A` — stage the reviewed batch so the next delta review starts clean. No commits — history stays clean for release.
5. Verify the plan checkboxes Codex ticked match what the diff actually contains; cross any it completed but missed.

**Adapt as you go**: clean batch → grow the next one; heavy corrections → shrink the next one and spell out the fix pattern in the notes. If Codex ignores notes or repeats corrected mistakes late in a long session, reset the thread at the next batch boundary — the plan file plus a summary note rebuilds context.

### 4. Final pass

After the last batch, read the **full feature diff** once (`git diff HEAD`). Batch reviews catch local issues; this pass catches cross-batch drift — duplicated helpers, divergent naming, dead code left by course corrections. Fix directly.

The testing gate and Codex code review run **once**, after the final pass — never per batch. Proceed to the testing gate once you consider the implementation good for review.

---

## Testing Gate

After implementation, before the Codex review loop. Any failure here blocks the loop from starting.

### 1. Compile-only build

**There is no lint step and no separate type-check step. The compiler is the type check.**
Build every project the change touched, **without deploying** (`-p:GameDeployPath=`):

```bash
dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath= 2>&1 | tee /tmp/_trip2-build.txt
grep -E "error CS|[0-9]+ Warning\(s\)|[0-9]+ Error\(s\)" /tmp/_trip2-build.txt | tail -5

# Only if touched:
dotnet build NOVR.McpBridge/NOVR.McpBridge.csproj -c Release -p:GameDeployPath=
dotnet build NOVR.Patcher/NOVR.Patcher.csproj -c Release -p:GameDeployPath=
```

- **0 errors.** The `NOVR` warning count must not rise above the baseline recorded in
  `docs/4-unit-tests/TESTING.md` (396 on `dev@2a0f789`), and there must be **no new warning in a
  touched file** (`grep` the touched paths in the build output). Update the baseline in
  `TESTING.md` when upstream moves it.
- ⚠️ Never build the full solution here: `NOVR.Installer` needs the .NET 10 SDK and this machine
  has .NET 9 only.

### 2. Run affected unit tests

Only pure-logic code (no `UnityEngine` dependency) has unit tests — see `TRIP-test` and
`docs/4-unit-tests/TESTING.md`. If `tests/NOVR.Tests/` exists and the change touched code it
covers:

```bash
dotnet test tests/NOVR.Tests/NOVR.Tests.csproj --filter "FullyQualifiedName~<pattern>"
```

Name the pattern the plan's **Test Impact** section gave you. If no test project exists yet and
the change adds no pure logic, record `tests: none (no pure-logic change)`.

⚠️ **Never pass `--no-build`** to a test run on a tree you have not just built: it runs whatever
assembly is on disk and reports green.

### 3. In-game verification (the integration check)

**Required whenever runtime behaviour changes** — anything under `NOVR/`, `NOVR.McpBridge/` or
`NOVR.Patcher/` beyond comments. Docs-only changes skip this. This step needs the user and the
headset; it is never delegated.

1. Ask the user to **close Nuclear Option** (Windows locks the plugin files).
2. **Deploying build** — installs this branch's build over the installed mod:
   ```bash
   dotnet build NOVR/NOVR.csproj -c Release
   ```
   (Also `NOVR.McpBridge` / `NOVR.Patcher` if touched.) Say so plainly: this replaces the
   player's installed NOVR until they reinstall a release (installer, ZIP or NOMM).
3. Ask the user to launch the game and load the plan's **In-Game Verification → Setup**.
4. **Measurements** — run each one through the MCP bridge and compare with the expected value:
   ```bash
   curl -s http://localhost:3334/tools
   curl -s -H "Content-Type: application/json" \
        -d '{"tool":"inspect_gameobject","args":{"name":"<object>","includeComponents":true}}' \
        http://localhost:3334/invoke
   ```
   (`args`, not `arguments`. Tools: `get_player_state`, `get_scene_hierarchy`,
   `inspect_gameobject`, `find_objects_by_type`, `get_canvas_group_chain`.)
5. **Headset checks** — put each check to the user with `AskUserQuestion` (pass/fail options,
   batched up to 4 per call). Never mark a headset check passed yourself.
6. **Log check** — the current session log must show no new exceptions or `NOVR:` errors:
   ```bash
   grep -nE "Exception|NOVR: .*[Ee]rror|Harmony" "$USERPROFILE/AppData/LocalLow/Shockfront/NuclearOption/Player.log" | tail -20
   ```
   (Read it with a shared-read open if the game still has it locked.)
7. Record results in the plan's In-Game Verification section (✅/❌ per item, with measured
   values).

### 4. Author missing tests

If the change adds new **pure** logic (angle maths, layout tables, parsing), write its tests
**now**, guided by the plan's **Test Impact** section and `TRIP-test`. Unity-bound code is proven
by step 3, not by unit tests.

**Hard-to-cover code policy:**

- Test **observable behavior** (inputs → outputs), never internal wiring.
- **Mock-pain tripwire**: if you would need to fake Unity or the game to test it, don't — move the
  logic into a pure helper if that is smaller than the feature, otherwise rely on step 3 and add
  one line to `docs/4-unit-tests/COVERAGE-DEBT.md` (`path | why hard | escape plan`).
- **Critical-path floor**: anything that writes game files (patcher), changes the camera stack or
  could break XR startup must have an explicit In-Game Verification item.
- Never hide untested code (no coverage-ignore comments, no config exclusions).

### 5. Build the summary

Format: `build: 0 errors, <W> warnings (baseline 396) | tests: N passed (M new) or none | in-game: <passed>/<total> checks`

Fix failures before starting the loop.

---

## Codex Code Review

Always run the Codex code review after the testing gate passes — no confirmation needed.

### Loop

The code-review wrappers pin their own state dir — no `export` needed:

1. **Start**:
   ```bash
   bash "$HOME/.claude/skills/codex-code-review/scripts/start.sh" \
       --prompt-file "$HOME/.claude/skills/codex-code-review/prompts/start.tpl" \
       <plan-path> "$GATE_SUMMARY"
   ```
   `$GATE_SUMMARY` is the testing-gate summary (`build | tests | in-game`). For unplanned work (no `F_*.plan.md`), pass a free-form label instead of a plan path.

2. **Parse trailing tag**: `APPROVED` -> synthesize. `NEEDS_REWORK` -> surface to user. `REQUEST_CHANGES` -> continue.

3. **Address findings** — quote each with `file:line`, read the actual code, fix legitimate ones, push back on incorrect ones. Critical/Major block approval; Minor/Suggestion are case-by-case. Codex applies `.claude/skills/TRIP-review/checklist.md`.

4. **Write implementer notes** (1-3 sentences): which findings you fixed, which you pushed back on and why, any user decisions or environment limitations Codex should stop re-flagging (for example: "cannot run the game; in-game results are in the plan").

5. **Resume** (re-run the testing gate first — compile build, affected tests, and the in-game items the fix touched — and build a fresh summary):
   ```bash
   bash "$HOME/.claude/skills/codex-code-review/scripts/resume.sh" \
       --prompt-file "$HOME/.claude/skills/codex-code-review/prompts/resume.tpl" \
       --notes "Fixed X. Pushed back on Y because Z." \
       <plan-path> "$GATE_SUMMARY"
   ```
   Loop to step 2.

6. **No cap** — keep iterating until Codex returns `APPROVED`.

### Synthesize

Skip if loop converged on Turn 1 (state file already holds full review).

Turn-N state files hold only that turn's delta. After multi-round convergence, produce a consolidated review:

```bash
bash "$HOME/.claude/skills/codex-code-review/scripts/resume.sh" \
    --prompt-file "$HOME/.claude/skills/codex-code-review/prompts/synthesize.tpl" \
    <plan-path> "Today's date is YYYY-MM-DD"
```

Outputs `PROMOTION_READY` sentinel. The `<x.y.z>` Version placeholder is left unfilled — `TRIP-3-release` fills it with the release ID `F_nnn`.

Edge case:
- **User skipped Codex**: no synthesis. The CR is written manually during `TRIP-3-release`: "Code review skipped — trivial change."

### Operating Notes

Surface reviews verbatim. Keep edits scoped. If Codex repeats a finding, re-read carefully — you likely addressed an adjacent concern. Reset thread only if context is confused. The testing gate (compile build, affected tests, in-game checks) must pass before APPROVED.

---

## Handoff to Release

After Codex converges (or is skipped):

- Cross the corresponding checkboxes in the plan todo list (if any)
- Then **use the `AskUserQuestion` tool** to ask:
  - **Question**: "Is the implementation complete?"
  - **Options**: "Yes, everything is complete" (proceed to release), "No, there are remaining items" (continue working)

**If "Yes"**: proceed directly into the release — read `.claude/skills/TRIP-3-release/SKILL.md` and follow it in this session, passing the same plan path (or feature label). The release skill owns everything from the release record to the upstream pull request.

**If "No"**: continue working, then repeat the sequence: testing gate → Codex review → this question.
