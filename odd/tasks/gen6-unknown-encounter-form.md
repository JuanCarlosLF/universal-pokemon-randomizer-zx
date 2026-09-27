# Feature: gen6-unknown-encounter-form fallback

## Objective
Apply the minimal Gen6 parser fallback so unknown alternate forms resolve to the base species instead of null, fixing both Eternal X NPEs.

## Problem
`Gen6RomHandler.readEncounter()` stores `pokes[0]` (null) when `(species, forme)` is unknown. Four Eternal X slots (Route 7 Munchlax f2/f4, Route 16 Misdreavus f1, Victory Road Whiscash f1) produce null Pokemon, crashing `pickWildPowerLvlReplacement` and the check-value pass in `Randomizer.randomize`.

## Why
Handoff `upr-zx-compatibility-hotfix-handoff.md` proves in-memory that base-form fallback lets `randomEncounters` complete (`null encounters=0; sets=336`) and that field items themselves are compatible (530 items / 25 TMs preserved).

## Scope
- In: `src/com/dabomstew/pkrandom/romhandlers/Gen6RomHandler.java` `readEncounter()` only.
- In: branch `hotfix/gen6-unknown-encounter-form`, work-unit commit, diff review.
- Out: no Eternal-X-specific branches, no warning removal, no checksum skipping, no build-system change, no ROM/assets in repo, no release build yet.

## Constraints
- Preserve behavior for recognized cosmetic/alt forms (`forme <= cosmeticForms`, 30, 31).
- Preserve original `Encounter.formeNumber` on read.
- Let randomization replace/reset form normally.
- Upstream base v4.6.1, `Version.VERSION = 322` unchanged.
- Java 8 / IntelliJ flow; no Maven/Gradle introduced.

## Authorized scope
User authorized first implementation + check on 2026-09-27: implement parser fallback on dedicated branch and verify whether it resolves the issue. Push/PR/release remain user decisions under ordinary policy.

## Acceptance criteria
- [x] No parsed Gen6 encounter contains null Pokemon solely because encoded form is unknown.
- [x] Both supplied Eternal X configs no longer throw recorded NPEs — user-validated 2026-09-27 ("Funciona perfecto": app runs under JDK 8 and randomizes the Eternal X ROM without the recorded exceptions).
- [x] Clean-ROM behavior unchanged for recognized forms (code-path inspection + compile).
- [x] Diff is minimal (fallback only) and reviewed.

## Applicable checks
- `javac` syntax-level check of edited file if JDK available (else structural readback).
- `git diff` review of the single-file change.
- Targeted null-scan reasoning over `readEncounter` callers (get/setEncounters XY/ORAS).
- Full manual matrix + LayeredFS + emulator remain pending (needs user ROM run) — recorded as next step, not claimed here.

## TDD mode
- Resolved: OFF. Source: no `*Test*.java` harness in repo, no `sdd-init` strict_tdd record, user did not request TDD. Ordinary functional checks apply, not RED/GREEN.

## Delivery forecast
- Forecast: ~4 added lines (1 file). Strategy: `ask-on-risk` default, single work-unit commit on feature branch. No chained PR (far below ~400-line budget). Generated files excluded.

## Tasks
- [x] T1 (route: inline, trigger: 1-file read to decide) — Verify current `readEncounter` matches handoff and error logs. Evidence: Gen6RomHandler.java:1485-1488, error_2026-09-27 logs.
- [x] T2 (route: inline, trigger: doc write is mechanical, no code research) — Create this feature document + Engram mirror before first write.
- [x] T3 (route: inline, trigger: single mechanical already-understood 1-file edit per Write rule) — Create branch `hotfix/gen6-unknown-encounter-form` and apply base-form fallback. Commit `5a267ce`.
- [x] T4 (route: inline, trigger: state check + 1-file verify) — Verify: diff review, compile attempt, null-invariant reasoning.
- [x] T5 (route: inline) — Report verified outcome, failed/pending checks, next step (manual matrix + build).

## Progress
- 2026-09-27: T1-T5 done. Branch `hotfix/gen6-unknown-encounter-form`, commit `5a267ce` (1 file, +3/-1). Compiled, assessed, ready for user ROM matrix.

## Verification evidence
- `git diff`: only `Gen6RomHandler.java` hunks 1485-1490, `e.pokemon = pokes[speciesWithForme]` -> `pokemonWithForme != null ? pokemonWithForme : baseForme`, `formeNumber` untouched.
- Focused check `javac -d <tmp>/gen6-check -sourcepath src src/.../Gen6RomHandler.java`: exit 0 (only pre-existing deprecation/unchecked notes).
- Null-invariant: unknown `(species, forme)` no longer yields `pokes[0]`/null; cosmetic/30/31 path unchanged; `writeEncounter` preserves bytes via `getBaseNumber() + formeNumber`, so Unchanged/levels-only keeps species/form data.
- Native assess (`--base-ref master --committed-only`, untracked excluded): `risk=medium` (`executable_change`), `changed_lines=4`, `review_due=false`, `reason=under_budget` — slice stays pending, no review lifecycle opened.
- Runtime harness (Eternal X GUI + LayeredFS + emulator): N/A here — pending user run. Rollback: revert hunks 1485-1490 in `Gen6RomHandler.java`.

## Next step
- After T3-T4: user runs manual compatibility matrix with actual Eternal X ROM; only then build `PokeRandoZX.jar`.

## Rationale for meaningful changes
- (none yet; fallback chosen over Randomizer null-checks per handoff: parser-level fix prevents invalid domain object from reaching ROM-write paths.)
