# Prompt iterations — `write-spec` skill

The reviewer prompt (`.claude/agents/spec-reviewer.md`) stays fixed across runs, so only the writer's skill changes. Each v2 rule must point to a failure that actually appeared in run 1.

## v1 → v2

### What run 1 showed (skill v1)

Source: `outputs/run-01/checks_round1.md`. Of 80 claims, 60 were verified, 11 incorrect, and 9 unresolved (counted from the table rows; the reviewer's own summary line said 56/13/11).

All 20 non-verified claims fall into six patterns:

| # | Failure pattern | Run-1 claims | Example |
|---|---|---|---|
| F1 | **Overstated scope words** ("all", "every", "never") that don't hold on every code path | C-32, C-62, C-65 (and C-84, added in the revision, incorrect in round 2) | C-62: "neither the terminal chat nor the web page ever crashes", but only the per-question call is wrapped (`cli.py:72-76`, `app.py:109-114`) |
| F2 | **One surface stated as general**: true for the terminal but not the web, for Windows but not other systems, or for prompt v2 but not v1 | C-10, C-16, C-46, C-48 | C-46: "the program stops", but `app.py:14-18` catches the SystemExit and the page script stops instead |
| F3 | **Missed branch next to the read code**: the claim matches the main path but misses an early return, fallback, or second path a few lines away | C-30, C-40, C-45, C-59 | C-30 missed `tools.py:340-341` (`ERROR:` when no asset codes are given); C-40 missed that averages use daily data (`tools.py:204-205`) |
| F4 | **Prompt instruction stated as behavior**: "the agent does X" when only the system prompt asks for X | C-05, C-10, C-17, C-78 | C-17: "the agent answers normally without calling a tool"; only `prompts.py:22-23` says so, and no code enforces it |
| F5 | **Comment, docstring, or external API property stated as fact** | C-38, C-39, C-43, C-69, C-80 | C-39: "Frankfurter data is daily", supported only by the comment at `tools.py:7` |
| F6 | **Claim about users or intent** | C-02 | C-02: "the target user is someone in Indonesia" |

What v1 didn't require, and what this shows: v1 has **no citation rule and no self-check**. None of the 80 claims named the file or line it came from. F3 errors are exactly what re-opening the cited lines and reading the surrounding branches would catch.

**Not a skill problem:** C-75 was verified in round 1 and incorrect in round 2, with identical text. That is reviewer variance, not a writer error, so no rule was added for it.

### v2 rules added

Each rule answers exactly one pattern above. No other changes were made.

| Rule | Text (summary) | Fixes | Claim IDs |
|---|---|---|---|
| R1 | Every claim cites `path:line` or `path:start-end` where it was read | F3, and makes every claim checkable | all 20 non-verified claims (none had a citation) |
| R2 | No "all / every / always / never / any / only" unless every code path was checked; otherwise name the scope | F1 | C-32, C-62, C-65, C-84 |
| R3 | When behavior differs by interface (terminal / web), OS, or prompt version, name which one the claim is about | F2 | C-10, C-16, C-46, C-48 |
| R4 | Write prompt instructions as "the system prompt tells the agent to …", never as "the agent does …" | F4 | C-05, C-10, C-17, C-78 |
| R5 | Comments, docstrings, and properties of external services or libraries are not evidence of behavior; such points go to Open questions | F5 | C-38, C-39, C-43, C-69, C-80 |
| R6 | Claims about users, market, or intent go to Open questions | F6 | C-02 |
| R7 | Self-check before saving: re-open each cited range, read the whole function (early returns, `except`, fallbacks, other callers via search), and fix or narrow the claim | F3 (and catches F1) | C-30, C-40, C-45, C-59 |
| R8 | (revision) New claims added during a revision follow R1–R7 too | F1 in the revision | C-84 |

**Considered, not added:** `[observed]` / `[inferred]` tags. The only purely inferred claim in run 1 was C-02, and R6 already covers it. Adding a tag rule with no matching failure would break the "only rules seen in run 1" principle (CLAUDE.md §8).

### Measured result

*To be filled after run 2 (B6): run-01 vs run-02 round-1 counts.*

| Run | Skill | Round-1 claims | Verified | Incorrect | Unresolved |
|---|---|---|---|---|---|
| run-01 | v1 | 80 | 60 | 11 | 9 |
| run-02 | v2 | | | | |

Limit: one run per version is a small sample. LLM output varies between runs, and the reviewer itself varied on C-75.

---

## Appendix: full text of skill v1

Exact copy of `.claude/skills/write-spec/SKILL.md` before the v2 changes (also in git history, commit `1aca952`).

~~~~markdown
---
name: write-spec
description: Write or revise a product spec by reverse-engineering the code in input/demo-a/. Use when asked to draft a spec for a run (outputs/run-NN/spec_draft.md) or to revise a draft using a spec-reviewer checks file (outputs/run-NN/spec.md).
---

Skill version: v1

You are reverse-engineering an existing product into a spec. The code is the only source of truth. Before writing anything, state in your reply: "Using write-spec skill v1."

## Input

Code root: `input/demo-a/` (Demo A at tag `demo-v1`, commit `6caaf96`). Read-only: never edit anything under `input/` and never run the code.

Read these in-scope files:

`currency_agent.py`, `cli.py`, `agent.py`, `tools.py`, `prompts.py`, `trace_log.py`, `app.py`, `requirements.txt`, `.env.example`, `.streamlit/config.toml`

Do not read anything else in `input/demo-a/` (no `README.md`, `docs/`, `demo_script.md`, `eval/`, `tests/`, `flowchart.svg`, `twd_idr_history.csv`, `pytest.ini`, `requirements-dev.txt`).

## Task: write a draft

Read the in-scope files and describe what the product does.

### Claim rules
- The spec is made of atomic, numbered claims: `C-01`, `C-02`, … One checkable fact per claim. "The tool retries on timeout and logs the error" is two claims.
- Number claims in order through the whole document, across sections.
- Open questions are not claims. Number them `Q-01`, `Q-02`, …

### Header
Start the spec with:

```
# Spec — Agentic Financial Monitoring Assistant
- Input repo: input/demo-a/ (Demo A)
- Commit: 6caaf96 (tag demo-v1)
- Skill version: v1
- Run ID: run-NN
- Date: YYYY-MM-DD
- Stage: draft | revised
```

### Sections (all 11, in this order)
1. Purpose and target user
2. User stories / features
3. Components (files and their responsibility)
4. Tools (table: `Claim | Name | Inputs | Output | Data source | Failure behavior`, one claim ID per row)
5. External data sources and dependencies
6. Configuration (keys, env vars, settings)
7. Error handling
8. Edge cases and known limits
9. Out of scope / non-goals
10. Open questions (things the code doesn't settle)
11. Change log (leave as "— (draft)" in a draft)

Save the draft to `outputs/run-NN/spec_draft.md`.

## Task: revise a draft

Inputs: `outputs/run-NN/spec_draft.md` and the reviewer's `outputs/run-NN/checks_round1.md`.

1. Copy the draft and set `Stage: revised`.
2. For every row whose verdict is not `verified`, re-read the relevant code yourself, then do exactly one of:
   - **Fix**: rewrite the claim to the reviewer's correction, if you agree after re-reading the code (or to your own accurate wording if you find the correction is also off; say why in the log).
   - **Remove**: keep the ID, strike the text through (`~~C-07 …~~`), give the reason in the change log.
   - **Move to Open questions**: strike the claim in the body and add a new `Q-` item that refers to the old claim ID.
3. For each omission the reviewer listed, you may add a new claim. New claims get new IDs after the current highest ID. Never renumber existing claims.
4. Claim IDs are stable: never reuse or renumber an ID.
5. Fill section 11, Change log, with one line per change:
   `- C-07 — fixed | removed | moved to Q-03 | added — reason (reviewer row / omission)`
6. Save to `outputs/run-NN/spec.md`. Do not modify `spec_draft.md` or the checks file.

## Where files go
- `outputs/run-NN/spec_draft.md` — draft
- `outputs/run-NN/checks_round1.md` — review of the draft (saved verbatim by the main session)
- `outputs/run-NN/spec.md` — revision
- `outputs/run-NN/checks_round2.md` — review of the revision (saved verbatim by the main session)
~~~~
