# Progress — Demo B
Input: Demo A @ demo-v1 (6caaf96)

## Completed steps
- [2026-11-12] B0 folder setup: `git init`, `.gitignore` (`input/demo-a/`), cloned Demo A from GitHub (`falthackel/agentic-financial-monitoring-assistant`) at tag `demo-v1` into `input/demo-a/`. Verified `git -C input/demo-a rev-parse --short HEAD` = `6caaf96`. No `.env` in clone.

## Decisions
- [2026-11-12] Cloned from the GitHub remote, not from the local `E:\Project\Test_Agentic`, so Demo A's own folder is never touched (CLAUDE.md §9.6).
- [2026-11-12] `demo-v1` is an annotated tag: tag object `1aecfaa`, commit `6caaf96`. Line numbers refer to commit `6caaf96`.

## Open issues
- Clone also contains `pytest.ini` and `requirements-dev.txt`. Neither file is on the in-scope or out-of-scope list in CLAUDE.md §4. Agent A will treat them as **out of scope** (they are test tooling, close to `tests/`) unless Farrel decides otherwise.

## Next action
B1: create `.claude/settings.json`, `.claude/agents/spec-reviewer.md`, `.claude/skills/write-spec/SKILL.md` (v1).

## Run log
| Run | Skill | Round | Claims | Verified | Incorrect | Unresolved | File |
|---|---|---|---|---|---|---|---|
