---
name: TRIP-3-release
description: Release a completed implementation - release record, code review promotion, changelogs, docs, commit, integrate into trip, build and open the upstream pull request
argument-hint: "plan file or feature label"
---

# Release Mode

You are now in **release mode** for **NOVR** (the Nuclear Option VR mod, fork `CMartinChan/novr`).

This is the project-local copy of `TRIP-3-release`. It replaces the global template.

Release: $ARGUMENTS

This skill runs after `TRIP-2-implement` has converged (implementation done, testing gate green, Codex code review `APPROVED` or explicitly skipped). It is normally chained from TRIP-2 in the same session, but can be invoked standalone in a fresh session.

⚠️ **How releases differ in this fork** (ARCHI.md §28):

- **Upstream owns the version.** Never change the `[BepInPlugin(..., "0.4.4")]` version in
  `NOVR/NOVRPlugin.cs`, never create tags, never push tags. The fork's release ID is `F_nnn`
  from the plan's file name.
- **Two destinations.** The whole `session/` branch (code + TRIP files) fast-forwards into the
  fork-only `trip` branch. The **code commits alone** are cherry-picked onto a `pr/` branch cut
  from `upstream/dev`, and that becomes the pull request to `InfernoSuperNova/novr` `dev`.
- **Commit separation.** Code commits never touch `docs/` or `.claude/`; TRIP commits touch
  nothing else.

---

## Prerequisites

- Implementation complete and user-confirmed.
- Testing gate green: compile-only build (0 errors, warnings within baseline), affected unit tests (if any), and the plan's In-Game Verification recorded as passed.
- Codex code review converged (`APPROVED`), or explicitly skipped by the user.

### Standalone verification (fresh session, not chained from TRIP-2)

If this skill was NOT chained from a TRIP-2 session in the current conversation, verify before any release step:

```bash
dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath=
# Only if touched:
dotnet build NOVR.McpBridge/NOVR.McpBridge.csproj -c Release -p:GameDeployPath=
dotnet build NOVR.Patcher/NOVR.Patcher.csproj -c Release -p:GameDeployPath=
# Only if tests/NOVR.Tests exists and the change touched code it covers:
dotnet test tests/NOVR.Tests/NOVR.Tests.csproj --filter "FullyQualifiedName~<pattern-from-the-plan's-Test-Impact-section>"
```

All must be green, and the plan's **In-Game Verification** section must show every item passed
(✅) with measured values. If it doesn't, the in-game check was not done — return to
`TRIP-2-implement` step 3. Also verify the Codex state file exists for the given plan path/label (see Step 3 below); if absent, treat as the skipped-Codex fallback (manual CR) and say so explicitly in the CR.

Any failure blocks the release — fix or return to `TRIP-2-implement` first.

---

## Step 1: Get Current Date/Week

Run this command to get date and project week:

```bash
date '+%Y-%m-%d %H:%M' && python -c "from datetime import date; print('Project week:', (date.today() - date.fromisoformat('2026-09-21')).days // 7 + 1)"
```

(Week anchor: **2026-09-21**, the Monday of the week TRIP Init was run. Python is used instead of `date -d` because macOS BSD `date` doesn't support it. This machine has `python`, not `python3`.)

**Dates are ISO 8601 (`YYYY-MM-DD`) everywhere** in this project, and times are America/Toronto. The stock TRIP template spells dates `DD-MM-YYYY`; this project does not.

Use the project week in all subsequent steps.

## Step 2: Release Record

- The release ID is the plan's `F_nnn` (for unplanned work, take the next free number from `docs/2-changelog/changelog_table.md`).
- Record the upstream base: `git fetch upstream && git rev-parse --short upstream/dev`.
- **Do not bump any version number.** There is no version file in this fork's workflow.

## Step 3: Promote Code Review

Now that week (`a`) and release ID (`F_nnn`) are known:

1. Compute state file path (derive the key via `key.sh` so it matches what the review scripts actually wrote — never re-derive it inline):
   ```bash
   STATE_KEY="$(bash "$HOME/.claude/skills/codex-plan-review/scripts/key.sh" <plan-path>)"
   STATE_FILE="$HOME/.claude/skills/codex-code-review/state/${STATE_KEY}.review.txt"
   ```

2. Content source:
   - **Multi-round loop**: state file has synthesized review + `PROMOTION_READY`. Strip sentinel.
   - **Turn 1 convergence**: state file has full review already.
   - **Skipped Codex**: write CR from `.claude/skills/TRIP-review/cr-template.md` with body "Code review skipped — trivial change." Verdict: `APPROVED with observations`.

3. Replace the `<x.y.z>` placeholder with the release ID `F_nnn`. Fill any remaining `<...>` placeholders.

4. Save to `docs/3-code-review/CR_wa_F_nnn.md`.

5. Verify: no `<...>` placeholders, no `PROMOTION_READY`, release ID matches the plan.

## Step 4: Commit Messages and PR Title

Propose:

- **Code commit message** — one imperative sentence in Canadian English, the style of upstream's
  history (for example "Fix bombing HUD VR projection broken by NO 0.34 TMP migration"). No
  `type:` prefix.
- **TRIP commit message** — `TRIP: F_nnn <feature-name> — plan, review, changelog`.
- **Upstream PR title** — `[Area] Brief description` (upstream convention, `AGENTS.md`), taken
  from the plan's **Upstream PR** section.

## Step 5: Changelog File

Create `docs/2-changelog/wa_F_nnn.md` (a=project week):

```markdown
# Changelog - Week a, YYYY-MM-DD, F_nnn

**Release Date**: Week a, YYYY-MM-DD at HH:MM
**Release**: F_nnn (previous: F_mmm)
**Upstream base**: `upstream/dev` @ <short sha>
**Upstream PR**: <pending — filled in Step 13>
**Object**: the code commit message
**Code review**: `docs/3-code-review/CR_wa_F_nnn.md` (Codex loop, N rounds -> verdict)
**In-game verification**: <passed>/<total> checks (see the plan)

## Changes

[Describe what changed]
```

## Step 6: Changelog Table

Add entry on top of the table in `docs/2-changelog/changelog_table.md`:

```markdown
| `F_nnn` | a | <upstream base sha> | pending | the code commit message |
```

Also add a summary entry at the top of the Changelog Summary section.

## Step 7: Architecture Update

1. Read fully @docs/ARCHI-rules.md
2. Update @docs/ARCHI.md following the rules
3. Run `bash "$HOME/.claude/skills/TRIP-compact/count-tokens.sh" docs/ARCHI.md` to check token count

**Warning: If ARCHI.md exceeds ~20,000 tokens**, warn the user:

> "ARCHI.md is at ~X tokens. Consider running `TRIP-compact` to reduce it before committing."

## Step 8: README Update

**`README.md` is upstream's file and carries no fork version — do not add one.** Skip this step
with a one-line note. Factual corrections to it belong to Documentation Sync, next step, and they
travel in the **code** commit because upstream should receive them.

## Step 9: Documentation Sync

Keep the pre-existing (non-TRIP) documentation aligned with the code:

1. Read the plan's **Documentation Impact** section.
2. Contrast it with the **actual diff** of the release — only the code side is what upstream will see:
   ```bash
   git fetch upstream && git diff upstream/dev...HEAD --stat -- . ':(exclude)docs' ':(exclude).claude'
   ```
   The plan may have fallen short; a doc affected by the real changes must be synced even if the plan didn't list it.
3. Update every affected document. **Factual corrections only**: commands, paths, build targets, script/table entries, config entries, structure trees. It is **forbidden** to touch the voice, tone, or strategic/editorial content of these documents.
4. Upstream docs (`README.md`, `AGENTS.md`, config descriptions) go in the **code** commit. Fork docs (`docs/design/…`) go in the **TRIP** commit.

If the plan says "None" and the diff confirms it, skip with a one-line note.

---

After completing all documentation steps, **use the `AskUserQuestion` tool** to ask:

- **Question**: "All documentation steps are complete. Ready to commit?"
- **Options**: "Yes, commit now" (proceed with the two commits), "Not yet" (review changes first)

**ONLY after user selects "Yes"**, proceed:

## Step 10: Commit (two commits, strictly separated)

```bash
# 1. Code commit — everything except TRIP files
git add -A -- . ':(exclude)docs' ':(exclude).claude'
git commit -m "<code commit message from Step 4>"

# 2. TRIP commit — only TRIP files
git add -A -- docs .claude
git commit -m "TRIP: F_nnn <feature-name> — plan, review, changelog"

# Verify the separation
git show --stat --format='%h %s' HEAD~1 HEAD
```

If the code commit lists any path under `docs/` or `.claude/`, or the TRIP commit lists any
other path, stop and fix the commits before going on.

**Trailer**: add the attribution trailer the session's own harness instructions require, and
nothing else. Do not invent one.

## Step 11: Integrate into `trip` (fast-forward)

```bash
git checkout trip
git merge --ff-only session/<short-description>
```

If `--ff-only` fails, `trip` moved during implementation (usually an upstream sync) — rebase the
session branch onto `trip`, re-run the compile-only build, then retry. **Never create a merge
commit here** (the upstream sync in TRIP-2 Step 0 is the only merge allowed on `trip`).

## Step 12: Build the upstream PR branch

```bash
git fetch upstream
git checkout -b pr/<short-description> upstream/dev
git cherry-pick <code-commit-sha>        # from Step 10; several if the release has several code commits
git diff --stat upstream/dev...HEAD      # must list NO path under docs/ or .claude/
dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath=
```

- If the cherry-pick conflicts, `upstream/dev` moved under the feature: resolve, rebuild, and tell
  the user what changed — do not open a PR on a branch that has not compiled.
- The compile-only build must be green on the `pr/` branch itself.

## Step 13: Push and open the pull request

This publishes to GitHub — the fork is public, and the PR goes to someone else's repository.

**Use the `AskUserQuestion` tool** to ask:

- **Question**: "F_nnn is committed and integrated into trip, and pr/<short-description> builds on upstream/dev. Push and open the pull request to InfernoSuperNova/novr dev?"
- **Options**: "Yes, push and open the PR", "Push only — I'll open the PR myself", "Not yet"

**If "Yes"**:

```bash
git push -u origin pr/<short-description>
git push origin trip
gh pr create --repo InfernoSuperNova/novr --base dev --head CMartinChan:pr/<short-description> \
    --title "[Area] Brief description" --body "<body>"
```

The PR body is for upstream: what changed and why, how it was verified in-game (the measured
values and headset checks, on which headset and game version), and any follow-up. **No TRIP
paths, no fork-internal references** (plans, CR files, `docs/design`). It ends with the
attribution line the session's harness instructions require.

Then record the PR in a TRIP-only commit on `trip`: replace `pending` in the changelog table and
the **Upstream PR** line in the changelog file with the PR number and URL, commit
(`TRIP: F_nnn record upstream PR #<n>`) and push `trip`.

**If "Push only"**: push both branches and give the user the compare URL
(`https://github.com/InfernoSuperNova/novr/compare/dev...CMartinChan:novr:pr/<short-description>`).

## Step 14: After upstream merges — only when the user says so

The repository has no CI and merging is upstream's decision. When the user reports the PR merged:

```bash
git fetch upstream
git checkout trip
git merge upstream/dev          # sync; the cherry-picked change arrives as upstream's own commit
dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath=
```

Record the merge (date, upstream merge commit) in the changelog file with a TRIP-only commit.

Delete branches only when the user asks (`git branch -d session/<…> pr/<…>` and
`git push origin --delete pr/<…>`).

If upstream requests changes on the PR: make them on `session/<…>` through the normal gate,
commit (code and TRIP separately), fast-forward `trip`, cherry-pick the new code commit onto
`pr/<…>`, rebuild, and push.
