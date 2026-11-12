# Progress — Demo B
Input: Demo A @ demo-v1 (6caaf96)

## Completed steps
- [2026-11-12] B0 folder setup: `git init`, `.gitignore` (`input/demo-a/`), cloned Demo A from GitHub (`falthackel/agentic-financial-monitoring-assistant`) at tag `demo-v1` into `input/demo-a/`. Verified `git -C input/demo-a rev-parse --short HEAD` = `6caaf96`. No `.env` in clone.
- [2026-11-12] B1 created `.claude/settings.json` (deny rules per §6.3), `.claude/agents/spec-reviewer.md` (Agent B, tools Read/Grep/Glob), `.claude/skills/write-spec/SKILL.md` (Agent A, skill v1). Farrel confirmed `/permissions` shows the 6 deny rules; `spec-reviewer` agent and `write-spec` skill registered by the harness.
- [2026-11-12] B2 run-01 draft written: `outputs/run-01/spec_draft.md` (skill v1, 80 claims C-01…C-80, 5 open questions Q-01…Q-05, all 11 sections).
- [2026-11-12] B3 run-01 round-1 review saved verbatim to `outputs/run-01/checks_round1.md`: 80 rows, 60 verified / 11 incorrect / 9 unresolved (row tally), 5 omissions.

## Decisions
- [2026-11-12] Cloned from the GitHub remote, not from the local `E:\Project\Test_Agentic`, so Demo A's own folder is never touched (CLAUDE.md §9.6).
- [2026-11-12] `demo-v1` is an annotated tag: tag object `1aecfaa`, commit `6caaf96`. Line numbers refer to commit `6caaf96`.
- [2026-11-12] Removed vs open-question handling: struck-through claims are not given a verdict (reviewer lists them in Reviewer notes); Open questions use `Q-NN` IDs and are not claims. So "total" in the run log = active claims only.
- [2026-11-12] Skill v1 has no citation rule and no observed/inferred tagging (deliberate "before", §8). The Tools table has one claim ID per row so the reviewer can verdict each tool.
- [2026-11-12] `/agents` wizard is removed in the installed Claude Code version, so it can't list agents. B1 check replaced by: the harness registered `spec-reviewer` (tools Read, Grep, Glob) as an agent type and `write-spec` as a skill after the files were created.
- [2026-11-12] No venv for Demo B (nothing is run or installed). If one is ever needed, use `uv`.

## Open issues
- Clone also contains `pytest.ini` and `requirements-dev.txt`. Neither file is on the in-scope or out-of-scope list in CLAUDE.md §4. Treated as **out of scope** (test tooling) in both prompts unless Farrel decides otherwise.
- `settings.json` deny rules are exactly §6.3; they don't cover `flowchart.svg`, `twd_idr_history.csv`, `pytest.ini`, `requirements-dev.txt`. Those are excluded by the prompt text only.
- run-01 round 1: the reviewer's "Summary counts" line (56 / 13 / 11) does not match its own table (60 verified / 11 incorrect / 9 unresolved; counted with `grep -c "^| C-.. | <verdict> |"`). File kept verbatim per §6.1. Run log uses the row tally. Reviewer prompt not changed (must stay fixed, §8); the main session should recount rows on every review.

## Next action
B4: revise `outputs/run-01/spec_draft.md` against `checks_round1.md` (20 non-verified rows) → `outputs/run-01/spec.md` with change log; then spec-reviewer round 2 → `checks_round2.md`.

## Run log
| Run | Skill | Round | Claims | Verified | Incorrect | Unresolved | File |
|---|---|---|---|---|---|---|---|
| run-01 | v1 | 1 | 80 | 60 | 11 | 9 | outputs/run-01/checks_round1.md (row tally; reviewer summary line said 56/13/11) |
