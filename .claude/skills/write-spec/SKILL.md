---
name: write-spec
description: Write or revise a product spec by reverse-engineering the code in input/demo-a/. Use when asked to draft a spec for a run (outputs/run-NN/spec_draft.md) or to revise a draft using a spec-reviewer checks file (outputs/run-NN/spec.md).
---

Skill version: v2

You are reverse-engineering an existing product into a spec. The code is the only source of truth. Before writing anything, state in your reply: "Using write-spec skill v2."

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

### v2 rules (added after run 1; see `docs/prompt_iterations.md`)
- **R1 Cite.** Every claim ends with the location it was read from, relative to `input/demo-a/`: `(path:line)` or `(path:start-end)`. Several locations are fine. A claim you can't point to in the code doesn't go in the body (see R5, R6).
- **R2 Scope words.** Don't write "all", "every", "always", "never", "any", or "only" unless you checked every code path it covers. Otherwise say exactly which path it covers (e.g. "while answering a question", "failures caught inside the tool's `try`").
- **R3 Name the surface.** When behavior differs between the terminal and the web page, between Windows and other systems, or between prompt versions, say which one the claim is about. If both behave the same, say "both".
- **R4 Prompt ≠ behavior.** Something that only the system prompt asks for is written as "The system prompt tells the agent to …", never as "The agent does …". Whether the model complies is not visible in code.
- **R5 Code, not comments or outside services.** Comments, docstrings, and how an external API or library behaves at runtime are not evidence of behavior. Put such points in Open questions, and state only what the code itself does (e.g. "sends no API key", not "the API needs no key").
- **R6 No user or intent claims.** Claims about who the users are, the market, or why something was designed a certain way go to Open questions.
- **R7 Self-check before saving.** For each claim, re-open the cited range and read the whole function around it: early returns, `except` branches, fallbacks, and (via search) other callers or a second interface that uses it. If anything makes the claim false or narrower, fix the claim before saving.

### Header
Start the spec with:

```
# Spec — Agentic Financial Monitoring Assistant
- Input repo: input/demo-a/ (Demo A)
- Commit: 6caaf96 (tag demo-v1)
- Skill version: v2
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
3. **R8:** Every fixed or new claim follows R1–R7, including the R7 self-check. A correction or an added claim can overstate just like a draft claim.
4. For each omission the reviewer listed, you may add a new claim. New claims get new IDs after the current highest ID. Never renumber existing claims.
5. Claim IDs are stable: never reuse or renumber an ID.
6. Fill section 11, Change log, with one line per change:
   `- C-07 — fixed | removed | moved to Q-03 | added — reason (reviewer row / omission)`
7. Save to `outputs/run-NN/spec.md`. Do not modify `spec_draft.md` or the checks file.

## Where files go
- `outputs/run-NN/spec_draft.md` — draft
- `outputs/run-NN/checks_round1.md` — review of the draft (saved verbatim by the main session)
- `outputs/run-NN/spec.md` — revision
- `outputs/run-NN/checks_round2.md` — review of the revision (saved verbatim by the main session)
