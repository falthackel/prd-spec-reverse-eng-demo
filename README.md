# PRD / Spec Reverse-Engineering with Multi-Agent Validation

An AI agent reverse-engineers an existing codebase into a spec. A second agent, with **read-only access and a separate context**, checks every claim in that spec against the code, with file-and-line evidence. Its findings are used twice: to fix the spec, and to fix the **prompt** that wrote it. The improvement is then measured on a fresh run.

Built with Claude Code (main session + a skill + subagents). No API keys, nothing to install.

---

## The problem

Legacy code often has missing or outdated docs. An LLM can read the code and write a spec, but it writes plausible text with confidence: "the app never crashes", "all errors are logged". If nobody checks those sentences against the code, the spec becomes another outdated doc, only faster to produce.

So this project asks two questions:
1. Can a second, independent agent catch the writer's errors **with evidence**, not opinion?
2. Can those errors be traced to the writer's prompt, fixed there, and the fix **measured**?

## The input

The "legacy product" is [Demo A](https://github.com/falthackel/agentic-financial-monitoring-assistant): an agentic financial assistant (Python, LangChain + Gemini, 4 tools, terminal and Streamlit UIs), pinned to tag `demo-v1` (commit `6caaf96`).

Both agents may read **only the product code** (`*.py`, `requirements.txt`, config). Demo A's README, docs, tests and eval are off-limits, both by permission rules and in the prompts. Otherwise the writer would copy prose instead of reverse-engineering code, and the review would prove nothing.

## Workflow

```
input/demo-a/  (read-only, commit 6caaf96)
      │
      ▼
[Agent A: writer]   skill .claude/skills/write-spec/SKILL.md
      │   spec_draft.md: atomic numbered claims C-01, C-02, …
      ▼
[Agent B: reviewer] subagent .claude/agents/spec-reviewer.md
      │   tools: Read, Grep, Glob only. Gets only the spec path and the code path.
      │   one row per claim: verified / incorrect / unresolved + path:line evidence
      ▼
[Agent A revises]   fix, remove, or move to Open questions; every change logged
      ▼
[Agent B re-reviews]
      ▼
analyze error patterns → add rules to the writer's skill (v1 → v2) → new run
```

Rules that keep it honest:
- **Independent review.** The reviewer never sees the writer's reasoning. It treats a citation as a lead, not proof, and searches for counter-evidence.
- **Saved verbatim.** Review tables are saved exactly as returned. The main session recounts the verdict rows itself (the reviewer once miscounted its own summary).
- **One revision per run.** No looping until it looks good.
- **Fixed reviewer.** The reviewer prompt doesn't change between runs, so runs are comparable.
- **Fresh writer.** Run 2 was written by a fresh subagent with no memory of run 1.
- **Real rules only.** v2 adds only rules for failures actually seen in run 1. No errors were planted.

## Results

First draft of each run, before any revision. Same code, same reviewer:

| Run | Writer skill | Claims | Verified | Incorrect | Unresolved |
|---|---|---|---|---|---|
| run-01 | v1 (template only) | 80 | 75.0% | **13.8%** | 11.3% |
| run-02 | v2 (+ rules from run-01 errors) | 116 | 96.6% | **1.7%** | 1.7% |

After one revision and re-review: run-01 reached 87/89 verified, run-02 reached 124/125.

**Run-03 (repeat of v2, wider input):** same skill v2, with two more files in scope (`pytest.ini`, `requirements-dev.txt`). First draft: 138 claims, 4 incorrect (2.9%), 1 unresolved (0.7%). After revision: 144/145 verified. Not strictly comparable with run-01/02 because the input differs. It does show v2's low error rate repeating on a second run.

A recurring blind spot: run-01 (C-84) and run-03 (C-140) were written by independent writers, yet both added the same wrong claim during revision. Each said a web-page label is always shown, when it isn't shown on failed turns. That's a candidate rule for a future v3.

**What v2 added** (each rule tied to run-1 claim IDs in [docs/prompt_iterations.md](docs/prompt_iterations.md)):
- cite `path:line` for every claim
- no "all / every / never" unless every code path was checked
- name the interface (terminal vs web), OS, or prompt version a claim is about
- write prompt instructions as instructions, not as observed behavior
- comments and external-API behavior aren't evidence; they go to Open questions
- no claims about users or intent
- a self-check pass that re-opens every cited range

**The story in 2 minutes:** [docs/case_study.md](docs/case_study.md) follows one error end to end: "the app never crashes" → caught with line evidence → fixed → traced to the prompt → rule added → measured in run 2.

**Limits:**
- One run per skill version is a small sample.
- v2 reduced the overstatement pattern but didn't eliminate it.
- The reviewer is not perfectly consistent: twice, unchanged text got different verdicts in round 1 and round 2.
- Run 1 and run 2 used different writer setups (main session vs fresh subagent).

## Repository layout

| Path | What |
|---|---|
| `.claude/skills/write-spec/SKILL.md` | Agent A: writer procedure (currently v2) |
| `.claude/agents/spec-reviewer.md` | Agent B: read-only reviewer |
| `.claude/settings.json` | Deny rules: no edits to `input/`, no reading Demo A's docs/tests/eval |
| `outputs/run-01/` … `outputs/run-03/` | Every run kept: `spec_draft.md`, `checks_round1.md`, `spec.md`, `checks_round2.md` |
| `outputs/spec.md`, `outputs/checks.md` | Latest final spec and review (run-03) |
| `docs/prompt_iterations.md` | Error patterns, v1 → v2 rules with evidence, full v1 text |
| `docs/case_study.md` | One caught error, end to end |
| `progress.md` | Step log, decisions, open issues, and the run log table |

## How to run it

Requires [Claude Code](https://claude.com/claude-code). Demo A's code isn't in this repo (`input/demo-a/` is git-ignored), so fetch it first:

```bash
git clone https://github.com/falthackel/prd-spec-reverse-eng-demo.git
cd prd-spec-reverse-eng-demo
git clone --branch demo-v1 https://github.com/falthackel/agentic-financial-monitoring-assistant.git input/demo-a
git -C input/demo-a rev-parse --short HEAD   # must print 6caaf96
```

Then open Claude Code in the folder and ask, step by step:

1. *"Start run-04: have a fresh subagent follow the write-spec skill and write `outputs/run-04/spec_draft.md`."*
2. *"Call spec-reviewer on `outputs/run-04/spec_draft.md` and save its output verbatim to `checks_round1.md`."*
3. *"Revise run-04 using the checks with a fresh subagent, then re-review."*

Nothing from Demo A is executed. The work is static analysis only.

Note: from run-03 onward, the in-scope files also include `pytest.ini` and `requirements-dev.txt`, so run-03 and later are not directly comparable to run-01/02.
