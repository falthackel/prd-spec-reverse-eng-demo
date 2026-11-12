# Progress — Demo B
Input: Demo A @ demo-v1 (6caaf96)

## Completed steps
- [2026-11-12] B0 folder setup: `git init`, `.gitignore` (`input/demo-a/`), cloned Demo A from GitHub (`falthackel/agentic-financial-monitoring-assistant`) at tag `demo-v1` into `input/demo-a/`. Verified `git -C input/demo-a rev-parse --short HEAD` = `6caaf96`. No `.env` in clone.
- [2026-11-12] B1 created `.claude/settings.json` (deny rules per §6.3), `.claude/agents/spec-reviewer.md` (Agent B, tools Read/Grep/Glob), `.claude/skills/write-spec/SKILL.md` (Agent A, skill v1). Farrel confirmed `/permissions` shows the 6 deny rules; `spec-reviewer` agent and `write-spec` skill registered by the harness.
- [2026-11-12] B2 run-01 draft written: `outputs/run-01/spec_draft.md` (skill v1, 80 claims C-01…C-80, 5 open questions Q-01…Q-05, all 11 sections).
- [2026-11-12] B3 run-01 round-1 review saved verbatim to `outputs/run-01/checks_round1.md`: 80 rows, 60 verified / 11 incorrect / 9 unresolved (row tally), 5 omissions.
- [2026-11-12] B4 run-01 revised → `outputs/run-01/spec.md` (18 fixed, 2 moved to Q-06/Q-07, 11 added C-81…C-91; 89 active claims). Round-2 review saved verbatim to `outputs/run-01/checks_round2.md`: 87 verified / 2 incorrect / 0 unresolved (summary line matches row tally).
- [2026-11-12] B5 run-01 errors analyzed into 6 patterns (F1–F6) in `docs/prompt_iterations.md`; v1 text saved there verbatim; `SKILL.md` updated to v2 with rules R1–R8, each tied to run-01 claim IDs.
- [2026-11-12] B6 run-02 draft by fresh subagent (skill v2, 116 claims, all cited) and round-1 review saved verbatim: 112 verified / 2 incorrect / 2 unresolved (summary matches rows). Incorrect rate 13.8% → 1.7%. Result recorded in `docs/prompt_iterations.md`.

## Decisions
- [2026-11-12] Cloned from the GitHub remote, not from the local `E:\Project\Test_Agentic`, so Demo A's own folder is never touched (CLAUDE.md §9.6).
- [2026-11-12] `demo-v1` is an annotated tag: tag object `1aecfaa`, commit `6caaf96`. Line numbers refer to commit `6caaf96`.
- [2026-11-12] Removed vs open-question handling: struck-through claims are not given a verdict (reviewer lists them in Reviewer notes); Open questions use `Q-NN` IDs and are not claims. So "total" in the run log = active claims only.
- [2026-11-12] Skill v1 has no citation rule and no observed/inferred tagging (deliberate "before", §8). The Tools table has one claim ID per row so the reviewer can verdict each tool.
- [2026-11-12] `/agents` wizard is removed in the installed Claude Code version, so it can't list agents. B1 check replaced by: the harness registered `spec-reviewer` (tools Read, Grep, Glob) as an agent type and `write-spec` as a skill after the files were created.
- [2026-11-12] v2 does not add `[observed]`/`[inferred]` tags: only C-02 was purely inferred, already covered by R6. Rule added only for failures seen in run 1 (§8).
- [2026-11-12] Generalizing the loop to other projects is a post-interview goal; not done now, to keep the v1→v2 comparison fair.
- [2026-11-12] Run-02 draft written by a **fresh general-purpose subagent** (no memory of run 1), not the main session, so the v1→v2 comparison isn't biased by the writer knowing run-1 errors. Its prompt: follow SKILL.md for run-02; don't read `outputs/`, `docs/`, `progress.md`, `README.md`; no git. Chosen by Farrel; he wants this setup to be the standard for future runs too. Limit: run-01 was written by the main session, so run-01 and run-02 writers differ in setup (both fresh to the code, but different sessions).
- [2026-11-12] No venv for Demo B (nothing is run or installed). If one is ever needed, use `uv`.

## Open issues
- Clone also contains `pytest.ini` and `requirements-dev.txt`. Neither file is on the in-scope or out-of-scope list in CLAUDE.md §4. Treated as **out of scope** (test tooling) in both prompts unless Farrel decides otherwise.
- `settings.json` deny rules are exactly §6.3; they don't cover `flowchart.svg`, `twd_idr_history.csv`, `pytest.ini`, `requirements-dev.txt`. Those are excluded by the prompt text only.
- run-01 round 1: the reviewer's "Summary counts" line (56 / 13 / 11) does not match its own table (60 verified / 11 incorrect / 9 unresolved; counted with `grep -c "^| C-.. | <verdict> |"`). File kept verbatim per §6.1. Run log uses the row tally. Reviewer prompt not changed (must stay fixed, §8); the main session should recount rows on every review.
- run-01 round 2, still incorrect (not looped again, §6.5):
  - **C-84** (added during revision) — the label wording is right, but it isn't shown on failed turns ("Failed") or on earlier turns with no steps. The revision itself introduced this error.
  - **C-75** — the claim text is identical in draft and revision, yet round 1 said verified and round 2 said incorrect ("removes `<br>`" vs "replaces exact `<br>` with a space; `<br/>`, `<BR>` untouched"). Main-session view: the round-2 reading is stricter but fair; the real finding is **reviewer variance between rounds** on unchanged text. Worth stating in the case study as a limit.
- run-01 round 2 listed 5 new omissions (on_step exceptions become AgentRunError; "[ESC pressed]" message shown on Ctrl+C/EOF off Windows; "n/a" change; history includes tool messages; `cli.ask()`). Not added: loop limit reached.

## Next action
B6 (optional) run-02 revision + round 2, or B7: `docs/case_study.md` + copy finals to `outputs/spec.md`, `outputs/checks.md`. Farrel to choose.

## Run log
| Run | Skill | Round | Claims | Verified | Incorrect | Unresolved | File |
|---|---|---|---|---|---|---|---|
| run-01 | v1 | 1 | 80 | 60 | 11 | 9 | outputs/run-01/checks_round1.md (row tally; reviewer summary line said 56/13/11) |
| run-01 | v1 | 2 | 89 | 87 | 2 | 0 | outputs/run-01/checks_round2.md |
| run-02 | v2 | 1 | 116 | 112 | 2 | 2 | outputs/run-02/checks_round1.md (writer: fresh subagent) |
