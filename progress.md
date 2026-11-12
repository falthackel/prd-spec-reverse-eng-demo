# Progress — Demo B
Input: Demo A @ demo-v1 (6caaf96)

## Completed steps
- [2026-11-12] B0 folder setup: `git init`, `.gitignore` (`input/demo-a/`), cloned Demo A from GitHub (`falthackel/agentic-financial-monitoring-assistant`) at tag `demo-v1` into `input/demo-a/`. Verified `git -C input/demo-a rev-parse --short HEAD` = `6caaf96`. No `.env` in clone.
- [2026-11-12] B1 created `.claude/settings.json` (deny rules per §6.3), `.claude/agents/spec-reviewer.md` (Agent B, tools Read/Grep/Glob), `.claude/skills/write-spec/SKILL.md` (Agent A, skill v1). Pending Farrel: read both prompts, confirm `/permissions` and `/agents`.

## Decisions
- [2026-11-12] Cloned from the GitHub remote, not from the local `E:\Project\Test_Agentic`, so Demo A's own folder is never touched (CLAUDE.md §9.6).
- [2026-11-12] `demo-v1` is an annotated tag: tag object `1aecfaa`, commit `6caaf96`. Line numbers refer to commit `6caaf96`.
- [2026-11-12] Removed vs open-question handling: struck-through claims are not given a verdict (reviewer lists them in Reviewer notes); Open questions use `Q-NN` IDs and are not claims. So "total" in the run log = active claims only.
- [2026-11-12] Skill v1 has no citation rule and no observed/inferred tagging (deliberate "before", §8). The Tools table has one claim ID per row so the reviewer can verdict each tool.
- [2026-11-12] No venv for Demo B (nothing is run or installed). If one is ever needed, use `uv`.

## Open issues
- Clone also contains `pytest.ini` and `requirements-dev.txt`. Neither file is on the in-scope or out-of-scope list in CLAUDE.md §4. Treated as **out of scope** (test tooling) in both prompts unless Farrel decides otherwise.
- `settings.json` deny rules are exactly §6.3; they don't cover `flowchart.svg`, `twd_idr_history.csv`, `pytest.ini`, `requirements-dev.txt`. Those are excluded by the prompt text only.

## Next action
Farrel: run `/permissions` and `/agents` to confirm B1. Then B2: run 1 draft with skill v1 → `outputs/run-01/spec_draft.md`.

## Run log
| Run | Skill | Round | Claims | Verified | Incorrect | Unresolved | File |
|---|---|---|---|---|---|---|---|
