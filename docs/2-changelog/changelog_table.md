# Changelog Table

Fork-local releases. **Upstream owns NOVR's version numbers and tags** — these IDs never appear
in code. `F_nnn` matches the plan file `docs/1-plans/F_nnn_<feature>.plan.md`. Week 1 is the
week of 2026-09-21, when TRIP was initialized.

| Release | Week | Upstream base | Upstream PR | Commit message |
| ------- | ---- | ------------- | ----------- | -------------- |
| `F_000` | 1    | `2a0f789`     | —           | TRIP: initialize the workflow |

# Changelog Summary

- **F_000 (TRIP Initialization - Week 1, 2026-09-27)**:
  - **Setup**: Initialized the TRIP workflow on the fork-only `trip` branch (upstream `dev` @ `2a0f789`, game 0.34.2).
  - **Documentation**: Generated `docs/ARCHI.md` for a Game-type project (runtime VR mod for a closed-source Unity game), with custom sections for game integration, XR bootstrap, head tracking, the HUD, menus and the cursor, diagnostics, the installer and the upstream relationship.
  - **Workflow decisions**: TRIP files live only in the fork. Code reaches upstream as pull requests to `InfernoSuperNova/novr` `dev`, built by cherry-picking code-only commits onto `upstream/dev`. No version bumps or tags (upstream owns them); plans are numbered `F_nnn`. Every plan carries In-Game Verification and Upstream PR sections. No tutorials.
  - **Skills**: Project-local copies of `TRIP-1-plan`, `TRIP-2-implement`, `TRIP-3-release`, `TRIP-review` and `TRIP-test` in `.claude/skills/`; Codex scripts are used from the global skills folder.
  - **Files Added**: `docs/ARCHI.md`, `docs/ARCHI-rules.md`, `docs/2-changelog/changelog_table.md`, `docs/4-unit-tests/TESTING.md`, `docs/4-unit-tests/COVERAGE-DEBT.md`, `docs/design/hud-redesign.html` (the 2026-09-27 HUD design pass), `.claude/skills/TRIP-*`.
