# Workflow and results

The detailed companion to the main [README](../README.md). It covers how the loop works, the rules that keep it honest, and what the runs measured.

## The problem in more detail

Legacy code often has missing or outdated docs. An LLM can read the code and write a spec, but it writes plausible text with confidence: "the app never crashes", "all errors are logged". If nobody checks those sentences against the code, the spec becomes another outdated doc, only faster to produce.

This project asks two questions:
1. Can a second, independent agent catch the writer's errors **with evidence**, not opinion?
2. Can those errors be traced to the writer's prompt, fixed there, and the fix **measured**?

The input is [Demo A](https://github.com/falthackel/agentic-financial-monitoring-assistant): an agentic financial assistant (Python, LangChain + Gemini, 4 tools, terminal and Streamlit UIs), pinned to tag `demo-v1` (commit `6caaf96`). Both agents may read **only the product code** (`*.py`, `requirements.txt`, config). Demo A's README, docs, tests and eval are off-limits, both by permission rules and in the prompts. Otherwise the writer would copy prose instead of reverse-engineering code, and the review would prove nothing.

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

Each claim gets exactly one verdict:
- **verified:** the code shows it's true. The reviewer gives the `path:line`.
- **incorrect:** the code shows it's false or overstated. The reviewer gives the contradicting lines and the correct statement.
- **unresolved:** the code can't settle it, for example runtime behavior of an external API or a claim about users. The reviewer says what evidence is missing.

Rules that keep it honest:
- **Independent review.** The reviewer never sees the writer's reasoning. It treats a citation as a lead, not proof, and searches for counter-evidence.
- **Saved verbatim.** Review tables are saved exactly as returned. The main session recounts the verdict rows itself (the reviewer once miscounted its own summary).
- **One revision per run.** No looping until it looks good.
- **Fixed reviewer.** The reviewer prompt doesn't change between runs, so runs are comparable.
- **Fresh writer.** From run 2 on, the writer is a fresh subagent with no memory of earlier runs.
- **Real rules only.** v2 adds only rules for failures actually seen in run 1. No errors were planted.

## Results

First draft of each run, before any revision. Same code, same reviewer:

| Run | Writer skill | Claims | Verified | Incorrect | Unresolved |
|---|---|---|---|---|---|
| run-01 | v1 (template only) | 80 | 75.0% | **13.8%** | 11.3% |
| run-02 | v2 (+ rules from run-01 errors) | 116 | 96.6% | **1.7%** | 1.7% |
| run-03 | v2, 12-file scope | 138 | 96.4% | **2.9%** | 0.7% |

After one revision and re-review: run-01 reached 87/89 verified, run-02 reached 124/125, and run-03 reached 144/145.

Run-03 read two more files (`pytest.ini`, `requirements-dev.txt`), so it is a repeat of v2 rather than a strict comparison. It shows v2's low error rate holding on a second, independent writer.

**What v2 added** (each rule tied to run-1 claim IDs in [prompt_iterations.md](prompt_iterations.md)):
- cite `path:line` for every claim
- no "all / every / never" unless every code path was checked
- name the interface (terminal vs web), OS, or prompt version a claim is about
- write prompt instructions as instructions, not as observed behavior
- comments and external-API behavior aren't evidence; they go to Open questions
- no claims about users or intent
- a self-check pass that re-opens every cited range

**A recurring blind spot:** run-01 (C-84) and run-03 (C-140) were written by independent writers, yet both added the same wrong claim during revision. Each said a web-page label is always shown, when it isn't shown on failed turns. That's a candidate rule for a future v3.

**The story in 2 minutes:** [case_study.md](case_study.md) follows one error end to end: "the app never crashes" → caught with line evidence → fixed → traced to the prompt → rule added → measured in run 2.

## Why Claude Code, and what about other LLMs?

### Why Claude Code

This repo contains no program, only instructions: a skill (the writer), a subagent definition (the reviewer), and permission rules. Claude Code is the runtime that turns them into working agents:

- **Guarantees are enforced by the tool, not by trust.** The reviewer runs as a subagent with its **own fresh context** and only `Read`, `Grep`, `Glob`, so it *cannot* see the writer's reasoning, edit files, or run commands. `settings.json` blocks edits to `input/` and reading Demo A's docs. Without this, "independent, read-only reviewer" would be a promise in a prompt, not a property of the setup.
- **No glue code, so no bugs of our own.** The model, the file-reading and search tools, the tool-calling loop, and subagent isolation are all built in. Nothing to install, no API keys. The only things we wrote are the parts that matter: what each agent may see and do, the evidence format, and the prompt-iteration loop.

### What if we use another LLM?

The design transfers. The prompts are plain Markdown and the claim/verdict format is model-agnostic. You could run the same loop with GPT, Gemini, or an open model, through the API, an agent framework (e.g. LangChain, as Demo A uses), or another agent CLI. What you'd have to rebuild, and what to watch for:

- **Rebuild the guarantees yourself.** You need read-only file and search tools, a separate conversation for the reviewer (only the spec path and code path, nothing else), and a block on reading out-of-scope files. If any of these leak, for example the reviewer sees the writer's chat history, the review is no longer independent and the results mean little.
- **The numbers here don't transfer.** All runs used Claude as both writer and reviewer. Another model will make different mistakes, so v2's rules may not fit it. Re-run v1 as a baseline and derive rules from *its* errors. Keep the reviewer's model and prompt fixed across runs, or the comparison breaks. One option worth testing: use a **different model as reviewer** than as writer. Two copies of the same model may share blind spots (see C-84 / C-140 above). A different model might catch errors the writer's model misses, but that is an untested idea here.

## Limits

- One run per skill version is a small sample.
- v2 reduced the overstatement pattern but didn't eliminate it.
- The reviewer is not perfectly consistent: twice, unchanged text got different verdicts in round 1 and round 2.
- Run 1 was written by the main session; runs 2 and 3 by fresh subagents.
- Run 3's input had two more files than runs 1 and 2.

## Other documents

| File | What |
|---|---|
| [prompt_iterations.md](prompt_iterations.md) | Error patterns F1–F6, v1 → v2 rules with evidence, measured result, full v1 text |
| [case_study.md](case_study.md) | One caught error, end to end |
| [../progress.md](../progress.md) | Step log, decisions, open issues, run log table |
