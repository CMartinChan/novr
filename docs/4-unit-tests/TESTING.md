# Testing Guidelines

NOVR is a runtime mod for a closed-source Unity game, so most of its behaviour can only be
proven in the running game. Verification has three layers, cheapest first. The TRIP-2 testing
gate runs them in this order; `TRIP-test` is the detailed authoring guide.

## Test Framework

- **Compile gate:** the C# compiler (the .NET 9 SDK here; the plugin targets `net48`). There is
  no linter, formatter or analyzer beyond compiler warnings.
- **Unit tests:** none yet. The first feature with pure logic creates `tests/NOVR.Tests`
  (`net9.0`, xUnit), which **compile-links** pure source files rather than referencing
  `NOVR.dll` (see `TRIP-test`).
- **In-game verification:** the running game, the headset (Chris: Rift S, Oculus runtime), and
  the **MCP bridge** (`NOVR.McpBridge`, `http://localhost:3334/`) for measured checks.

## Running Tests

```bash
# 1. Compile gate — never deploys
dotnet build NOVR/NOVR.csproj -c Release -p:GameDeployPath=
dotnet build NOVR.McpBridge/NOVR.McpBridge.csproj -c Release -p:GameDeployPath=   # if touched
dotnet build NOVR.Patcher/NOVR.Patcher.csproj -c Release -p:GameDeployPath=       # if touched

# 2. Unit tests — once tests/NOVR.Tests exists
dotnet test tests/NOVR.Tests/NOVR.Tests.csproj --filter "FullyQualifiedName~<pattern>"

# 3. In-game — close the game, deploy, relaunch, then measure
dotnet build NOVR/NOVR.csproj -c Release          # replaces the installed NOVR
curl -s http://localhost:3334/tools
curl -s -H "Content-Type: application/json" \
     -d '{"tool":"get_scene_hierarchy","args":{"maxDepth":2}}' http://localhost:3334/invoke
```

Log to check after an in-game session:
`%USERPROFILE%\AppData\LocalLow\Shockfront\NuclearOption\Player.log` (BepInEx output is mirrored
there; `BepInEx/LogOutput.log` may be empty).

## Compile Baseline

| Project | Errors | Warnings | Measured on |
|---|---|---|---|
| `NOVR` | 0 | 396 | `dev@2a0f789`, 2026-09-27 |
| `NOVR.McpBridge` | 0 | 0 | `dev@2a0f789`, 2026-09-27 |
| `NOVR.Patcher` | 0 | 0 | `dev@2a0f789`, 2026-09-27 |

A change must not raise a project's warning count, and must add no warning in a file it
touched. Update this table when an upstream sync moves the baseline.

`NOVR.Installer` (net10.0) cannot build on this machine's .NET 9 SDK; don't build the full
solution.

## Test Organization

- `tests/NOVR.Tests/` (to be created): one `<Subject>Tests.cs` per pure helper; behaviour-named
  test methods.
- In-game results live in each plan's **In-Game Verification** section (✅/❌ per item, with
  measured values), and in the release's test summary `docs/4-unit-tests/wa_F_nnn_test.md`.
- Untestable risky paths are listed in `COVERAGE-DEBT.md`.

## Writing Tests

- Unit-test only code with no `UnityEngine`, Harmony or game-type dependency. Keep such helpers
  in their own files so the test project can link them.
- Measure before and after: when refactoring Unity-bound code, record the MCP bridge values
  first, change the code, then compare.
- Headset checks are phrased as yes/no questions with a pass criterion, and answered by the
  user — never assumed.
- Fail-soft behaviour (a renamed game member) is checked by reading the code path, since it
  can't be triggered in the current game version.

## Coverage Requirements

Not defined. There is no coverage gate. Every behaviour a plan promises must have a passed
In-Game Verification item, and new pure logic must have unit tests or a `COVERAGE-DEBT.md` entry.
