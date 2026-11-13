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
- [2026-11-13] B6 run-02 revision by a fresh subagent (4 fixed, 9 added C-117…C-125, Q-15/Q-16 added; 125 active claims) → `outputs/run-02/spec.md`. Round-2 review saved verbatim: 124 verified / 1 incorrect / 0 unresolved (summary matches rows).
- [2026-11-13] B7 `docs/case_study.md` written (C-62 "never crashes": claim → reviewer evidence → fix → pattern C-32/C-62/C-65/C-84 → rules R1/R2/R7 → run-02 C-80/C-88/C-91 verified; limits stated). Finals copied: `outputs/spec.md` = run-02 `spec.md`, `outputs/checks.md` = run-02 `checks_round2.md` (byte-identical, checked with `cmp`).
- [2026-11-13] B8 `README.md` written; pushed to https://github.com/falthackel/prd-spec-reverse-eng-demo (branch `main`). Checked before push: 0 files under `input/` tracked, no secrets.

## Decisions
- [2026-11-12] Cloned from the GitHub remote, not from the local `E:\Project\Test_Agentic`, so Demo A's own folder is never touched (CLAUDE.md §9.6).
- [2026-11-12] `demo-v1` is an annotated tag: tag object `1aecfaa`, commit `6caaf96`. Line numbers refer to commit `6caaf96`.
- [2026-11-12] Removed vs open-question handling: struck-through claims are not given a verdict (reviewer lists them in Reviewer notes); Open questions use `Q-NN` IDs and are not claims. So "total" in the run log = active claims only.
- [2026-11-12] Skill v1 has no citation rule and no observed/inferred tagging (deliberate "before", §8). The Tools table has one claim ID per row so the reviewer can verdict each tool.
- [2026-11-12] `/agents` wizard is removed in the installed Claude Code version, so it can't list agents. B1 check replaced by: the harness registered `spec-reviewer` (tools Read, Grep, Glob) as an agent type and `write-spec` as a skill after the files were created.
- [2026-11-12] v2 does not add `[observed]`/`[inferred]` tags: only C-02 was purely inferred, already covered by R6. Rule added only for failures seen in run 1 (§8).
- [2026-11-12] Generalizing the loop to other projects is a post-interview goal; not done now, to keep the v1→v2 comparison fair.
- [2026-11-12] Run-02 draft written by a **fresh general-purpose subagent** (no memory of run 1), not the main session, so the v1→v2 comparison isn't biased by the writer knowing run-1 errors. Its prompt: follow SKILL.md for run-02; don't read `outputs/`, `docs/`, `progress.md`, `README.md`; no git. Chosen by Farrel; he wants this setup to be the standard for future runs too. Limit: run-01 was written by the main session, so run-01 and run-02 writers differ in setup (both fresh to the code, but different sessions).
- [2026-11-13] Run-02 revision was also done by a fresh subagent (run-01 revision was done by the main session). Fits Farrel's "fresh writer per run" setup.
- [2026-11-13] Farrel: add `pytest.ini` and `requirements-dev.txt` to the in-scope files. Applies from **run-03** onward; run-01 and run-02 stay measured on the original 10 files so they remain comparable.
- [2026-11-13] Scope change edited in SKILL.md (lines 16, 18), spec-reviewer.md (lines 19, 21), CLAUDE.md §4.3. The reviewer prompt changed only in its file list, not in how it judges. §8 says a reviewer change needs a re-review of run 1; not done, because run-01/run-02 stay on the 10-file scope and are not compared against run-03 with the new list. If run-03 is compared to them, say the scope differs by these 2 files.
- [2026-11-12] No venv for Demo B (nothing is run or installed). If one is ever needed, use `uv`.

## Open issues
- ~~Clone also contains `pytest.ini` and `requirements-dev.txt`, not on either scope list.~~ Resolved 2026-11-13: Farrel put them in scope from run-03 (see Decisions).
- `settings.json` deny rules are exactly §6.3; they don't cover `flowchart.svg`, `twd_idr_history.csv`. Those are excluded by the prompt text only.
- run-01 round 1: the reviewer's "Summary counts" line (56 / 13 / 11) does not match its own table (60 verified / 11 incorrect / 9 unresolved; counted with `grep -c "^| C-.. | <verdict> |"`). File kept verbatim per §6.1. Run log uses the row tally. Reviewer prompt not changed (must stay fixed, §8); the main session should recount rows on every review.
- run-01 round 2, still incorrect (not looped again, §6.5):
  - **C-84** (added during revision) — the label wording is right, but it isn't shown on failed turns ("Failed") or on earlier turns with no steps. The revision itself introduced this error.
  - **C-75** — the claim text is identical in draft and revision, yet round 1 said verified and round 2 said incorrect ("removes `<br>`" vs "replaces exact `<br>` with a space; `<br/>`, `<BR>` untouched"). Main-session view: the round-2 reading is stricter but fair; the real finding is **reviewer variance between rounds** on unchanged text. Worth stating in the case study as a limit.
- run-02 round 2, still incorrect (not looped again, §6.5): **C-84** — whitespace-only string replies are not stripped, so they skip the empty-answer placeholder. Text is **identical** in draft and revision; round 1 said verified. Second case of reviewer variance on unchanged text (after run-01 C-75): round 2 tends to be stricter.
- run-02 round 2 listed 5 new omissions (per-asset log lines in price table; IHSG fails if 5-day history empty; all-failed ERROR includes table; codes not stripped in get_exchange_rate; on_step only for new messages). Not added: loop limit reached.
- run-01 round 2 listed 5 new omissions (on_step exceptions become AgentRunError; "[ESC pressed]" message shown on Ctrl+C/EOF off Windows; "n/a" change; history includes tool messages; `cli.ask()`). Not added: loop limit reached.

## Next action
Done through B8. Optional next: run-03 with the 12-file scope, or generalize the loop for other projects (post-interview).

## Run log
| Run | Skill | Round | Claims | Verified | Incorrect | Unresolved | File |
|---|---|---|---|---|---|---|---|
| run-01 | v1 | 1 | 80 | 60 | 11 | 9 | outputs/run-01/checks_round1.md (row tally; reviewer summary line said 56/13/11) |
| run-01 | v1 | 2 | 89 | 87 | 2 | 0 | outputs/run-01/checks_round2.md |
| run-02 | v2 | 1 | 116 | 112 | 2 | 2 | outputs/run-02/checks_round1.md (writer: fresh subagent) |
| run-02 | v2 | 2 | 125 | 124 | 1 | 0 | outputs/run-02/checks_round2.md |
