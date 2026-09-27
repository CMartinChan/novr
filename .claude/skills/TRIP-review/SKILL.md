---
name: TRIP-review
description: Review code following project standards (manual fallback/audit path)
disable-model-invocation: true
argument-hint: "release ID (F_nnn) or feature to review"
---

# Review Mode

You are now in **code review mode** for **NOVR** (the Nuclear Option VR mod, fork `CMartinChan/novr`).

This is the **manual fallback/audit path**: normal reviews happen via the Codex loop inside `TRIP-2-implement`. Use this skill to audit a past release, review unplanned work, review an upstream contributor's change before merging it into `trip`, or replace the Codex loop when it is unavailable.

Review: $ARGUMENTS

## Prerequisites

Read before reviewing:
1. @docs/ARCHI.md — verify architectural compliance
2. Related plan in `docs/1-plans/` (its In-Game Verification and Upstream PR sections)
3. Related changelog in `docs/2-changelog/`
4. @.claude/skills/TRIP-review/checklist.md — **single source of truth** for review criteria, severity classification, and approval gate

---

## Apply the Checklist

Walk every section of `checklist.md` against the change. Tick passing items. Failing items become findings classified by the severity scale in that file. Approval requires the gate at the bottom of `checklist.md`.

Do not copy the checklist into output — link to it.

A reviewer cannot run the game. Items that need the headset are satisfied only by results
recorded in the plan's In-Game Verification section — never assume them.

---

## Create Review File

Save to `docs/3-code-review/CR_wa_F_nnn.md` (a=project week, F_nnn=release ID from the plan).

Render the skeleton from `@.claude/skills/TRIP-review/cr-template.md`:
1. Copy the markdown block from that file.
2. Replace every `<angle-bracket placeholder>` with concrete content.
3. Tick `[x]` for passing checklist items; leave unchecked with a one-line caveat otherwise.

Every checklist item must be ticked or annotated — a silent unchecked box is a red flag.
