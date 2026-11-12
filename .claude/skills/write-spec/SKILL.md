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
