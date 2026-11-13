# CLAUDE.md — Demo B: Spec Reverse-Engineering with Multi-Agent Validation

This file explains the project to Claude Code and lists the work. Read all of it before making changes.

---

## 1. Context

- **Owner:** Farrel. He's a Smart MHI master's student at NTU and a biomedical engineer, not a CS major. He's strongest at *specifying behavior* and reads code well enough to follow it. When you change a file he already has, tell him which lines change and why. Don't rewrite whole files.
- **Purpose:** He'll demo this at a Jubo Health interview for the **"AI Agent Product Intern"** role on **Nov 17, 2026**. The role reverse-engineers existing codebases into specs/PRDs and builds a second agent that validates the spec against the original. This project is a small, honest version of that work.
- **Input "legacy product":** Demo A, the Agentic Financial Monitoring Assistant (Python, LangChain `create_agent` + Gemini, 4 tools), pinned to git tag `demo-v1`.
- **Interview story this project must support:**
  > "An agent reverse-engineered my own product into a spec. A second agent, with read-only access and a separate context, checked every claim against the code with file-and-line evidence. It caught real errors, I traced them to the spec-writing prompt, changed the prompt, and measured the drop."
- **Priority:** the core loop (write → review → revise → re-review) working end to end with real outputs comes first. Polish comes after. Cut scope, not honesty.

## 2. The workflow

```
input/demo-a/ (read-only, tag demo-v1)
      │
      ▼
[Agent A: spec writer]  main session + skill `write-spec`
      │   writes outputs/run-NN/spec_draft.md
      ▼
[Agent B: spec reviewer]  subagent `spec-reviewer`, tools: Read, Grep, Glob only, fresh context
      │   returns the claim table → main session saves it verbatim to outputs/run-NN/checks_round1.md
      ▼
[Agent A revises]  fixes / removes / moves to Open Questions every claim not "verified"
      │   writes outputs/run-NN/spec.md (with a change log)
      ▼
[Agent B re-reviews]  → outputs/run-NN/checks_round2.md
      ▼
copy final spec.md + checks_round2.md → outputs/spec.md, outputs/checks.md
update progress.md
```

Two kinds of fixes, and the demo needs both:
- **Revision** fixes *this* spec (the instance).
- **Prompt change** to the `write-spec` skill fixes *the cause*, so the next run makes fewer errors. Measure it with a second run.

## 3. Folder layout

```
<this folder>/
├── CLAUDE.md                        # this file
├── README.md                        # problem → workflow → how to run → results (written last)
├── progress.md                      # handoff file between sessions (see §7)
├── .gitignore                       # ignores input/demo-a/
├── .claude/
│   ├── settings.json                # permission rules (see §6)
│   ├── skills/write-spec/SKILL.md   # Agent A procedure (versioned, see §8)
│   └── agents/spec-reviewer.md      # Agent B definition
├── input/
│   └── demo-a/                      # clone of Demo A at tag demo-v1. NEVER edit.
├── outputs/
│   ├── spec.md                      # latest final spec
│   ├── checks.md                    # latest final review
│   └── run-01/, run-02/ …           # every run kept: spec_draft, checks_round1, spec, checks_round2
└── docs/
    ├── prompt_iterations.md         # skill v1 → v2: failure, evidence, change, measured result
    └── case_study.md                # the one error Agent B caught, told end to end (demo material)
```

## 4. Input rules (Demo A)

1. **Read-only.** Never edit, create, or delete anything under `input/`. Never run Demo A code (no `python`, no `pip`, no Gemini calls). This is static analysis only. Demo A has no `.env` in the clone and must not get one.
2. **Pinned version.** The clone is at tag `demo-v1` (commit `6caaf96`). Every line number in a spec or review refers to that commit. If the clone is ever re-made, check `git -C input/demo-a rev-parse --short HEAD` still prints `6caaf96`. Record the commit in `progress.md`.
3. **In scope (the "product code"):**
   `currency_agent.py`, `cli.py`, `agent.py`, `tools.py`, `prompts.py`, `trace_log.py`, `app.py`, `requirements.txt`, `.env.example`, `.streamlit/config.toml`, `pytest.ini`, `requirements-dev.txt`.
   (`pytest.ini` and `requirements-dev.txt` added 2026-11-13, from run-03 onward. run-01 and run-02 used the first 10 files only.)
4. **Out of scope, do not read:** `README.md`, `docs/`, `demo_script.md`, `eval/`, `tests/`, `flowchart.svg`, `twd_idr_history.csv`.
   Why: Jubo's real case is legacy code with missing or outdated docs. If Agent A reads Demo A's README, it copies prose instead of reverse-engineering code, and the review proves nothing. Code is the only source of truth for both agents.

## 5. Evidence and output formats (binding for both agents)

### 5.1 Claims in the spec
- The spec is made of **atomic, numbered claims**: `C-01`, `C-02`, … One checkable fact per claim. "The tool retries on timeout and logs the error" is two claims.
- IDs are stable across revisions. A removed claim keeps its ID with ~~strikethrough~~ and a reason in the change log. New claims get new IDs at the end of the sequence.
- Section structure of `spec.md`:
  1. Purpose and target user
  2. User stories / features
  3. Components (files and their responsibility)
  4. Tools (table: name | inputs | output | data source | failure behavior)
  5. External data sources and dependencies
  6. Configuration (keys, env vars, settings)
  7. Error handling
  8. Edge cases and known limits
  9. Out of scope / non-goals
  10. Open questions (things the code doesn't settle)
  11. Change log (filled on revision)
- Header of every spec: input repo, commit, skill version, run ID, date.

### 5.2 Review table (`checks*.md`)
Every claim in the spec gets exactly one row. No claim may be skipped.

| Claim | Verdict | Evidence | Required correction |
|---|---|---|---|
| C-07 | verified / incorrect / unresolved | `tools.py:214-250` + short quote or description | what to change, or "—" |

Verdict definitions:
- **verified**: the cited (or found) code shows the claim is true as written. Evidence = `path:line` or `path:start-end` relative to `input/demo-a/`, forward slashes, plus a quote of ≤ 1 line or a one-line description.
- **incorrect**: the code shows the claim is false, or true only in a narrower/different way. Evidence = the lines that contradict it. Correction = the accurate statement.
- **unresolved**: the code in scope can't confirm or deny it (e.g. runtime behavior of an external API, a claim about users). Must state **what evidence is missing** and where it would come from (e.g. "needs a live API response; not visible in code").

Below the table, the review always ends with:
- **Summary counts:** total, verified, incorrect, unresolved.
- **Omissions (max 5):** important behaviors in the code that the spec doesn't mention, each with `path:line`.
- **Reviewer notes:** anything ambiguous about the review itself.

Docs and comments are not evidence for *behavior*. A code comment can support a claim about intent only, and the row must say so.

## 6. Harness rules (Claude Code)

1. **Agent B is a subagent:** `.claude/agents/spec-reviewer.md`, frontmatter `tools: Read, Grep, Glob`. No Edit, Write, or Bash. It can't save files, so it **returns** the table and the main session saves it **verbatim** (no edits, no re-ordering, no "cleanup"). If the main session disagrees with a verdict, it writes that in `progress.md` → Open issues, not in the checks file.
2. **Separate context for real:** when calling the reviewer, pass only (a) the path to the spec file and (b) the path `input/demo-a/`. Don't pass Agent A's reasoning, summaries, or hints about which claims are shaky. The point is an independent check.
3. **Permissions** in `.claude/settings.json`:
   ```json
   {
     "permissions": {
       "deny": [
         "Edit(./input/**)",
         "Read(./input/demo-a/README.md)",
         "Read(./input/demo-a/demo_script.md)",
         "Read(./input/demo-a/docs/**)",
         "Read(./input/demo-a/eval/**)",
         "Read(./input/demo-a/tests/**)"
       ]
     }
   }
   ```
   After creating it, ask Farrel to run `/permissions` in Claude Code to confirm the rules loaded. Deny rules are a backstop. The rules in §4 still apply even where a tool (e.g. Bash `cat`) could get around them.
4. **Agent A is the main session following the `write-spec` skill.** Before writing, it states which skill version it is using.
5. **Loop limit:** one revision and one re-review per run. If round 2 still has `incorrect` rows, record them in `progress.md` → Open issues. Don't loop again. That's also an honest demo point.
6. **No Gemini, no API keys.** Demo B uses Claude Code itself as both agents. There's no `.env` and nothing to install.

## 7. `progress.md` (handoff file)

Update it at the end of every step, and read it first at the start of every session. Fixed sections:

```markdown
# Progress — Demo B
Input: Demo A @ demo-v1 (6caaf96)

## Completed steps
- [date] B2 run-01 draft written (skill v1, 34 claims)

## Decisions
- [date] Docs excluded from input — reason: …

## Open issues
- …

## Next action
One concrete step, e.g. "B4: call spec-reviewer on outputs/run-01/spec_draft.md"

## Run log
| Run | Skill | Round | Claims | Verified | Incorrect | Unresolved | File |
|---|---|---|---|---|---|---|---|
```

The run log table is the main evidence for the interview. Fill it from the summary counts, never from memory.

## 8. Skill versioning and the prompt-iteration story

- **Skill v1** (baseline): the section template from §5.1, atomic numbered claims, "read the in-scope files and describe what the product does." **No** requirement to cite lines, no rule for separating observed vs inferred behavior. This is deliberate, and it's the "before".
- Run 1 uses v1. Agent B's round-1 review shows which errors it makes.
- **Skill v2**: add only rules that fix errors actually seen in run 1. Likely candidates, but add a rule **only if run 1 shows the failure**:
  - each claim cites `path:line` where it was read
  - tag each claim `[observed]` (read directly in code) or `[inferred]` (reasoned from code); inferred claims need the reasoning in one line
  - a claim with no code location goes to Open questions, not the body
  - before finishing, re-open each cited range and check the claim still matches (self-check pass)
  - no claims about runtime behavior of external APIs beyond what the code handles
- Run 2 uses v2, same input, same reviewer. Compare round-1 counts of run 1 vs run 2 in `docs/prompt_iterations.md`.
- Keep the full text of v1 in `docs/prompt_iterations.md` before overwriting `SKILL.md` with v2 (also visible in git history).
- **Honesty rule:** never plant or invent an error to make the story better. If v1 makes no incorrect claims, record that, use the unresolved/omission findings as the story, and say so in the case study. LLM runs vary, so one run per version is a small sample. State that limit.
- The reviewer prompt stays **fixed** across runs so the comparison is fair. If it must change, record why and re-review run 1 with the new version.

## 9. Working rules

1. **Explain before editing.** Say which file and section changes and why. Small steps.
2. **One task at a time** from §10. After each task: update `progress.md`, show Farrel the result, wait for his OK before the next task.
3. **Git:** `git init` at B0. One commit per task, message like `B3: run-01 round-1 review (34 claims: 27/4/3)`. Never commit `input/demo-a/`.
4. **Language:** everything in English (spec, checks, docs, README).
5. **Windows notes:**
   - Claude Code on Windows runs shell commands in Git Bash. Use forward slashes in paths and evidence (`input/demo-a/tools.py:214`).
   - Git may check files out with CRLF. Line numbers don't change, but don't "fix" line endings in `input/`.
   - Save every file as UTF-8.
6. **Don't touch Demo A's own folder** (`E:\Project\Test_Agentic`). Only the clone in `input/` is used, and only for reading.

## 10. Task list (Friday Nov 13)

| ID | Task | Est. | Done when |
|---|---|---|---|
| B0 | Folder setup: `git init`, `.gitignore` (`input/demo-a/`), clone Demo A at `demo-v1` into `input/demo-a`, verify commit `6caaf96`, create `progress.md` | 15 min | `git status` doesn't list `input/demo-a/`; commit recorded |
| B1 | Create `.claude/settings.json`, `.claude/agents/spec-reviewer.md`, `.claude/skills/write-spec/SKILL.md` (v1) | 30 min | Farrel has read both prompts; `/permissions` shows deny rules; `/agents` lists spec-reviewer |
| B2 | Run 1, Agent A: `outputs/run-01/spec_draft.md` with skill v1 | 15 min | All 11 sections present, claims numbered |
| B3 | Run 1, Agent B round 1 → `checks_round1.md` | 20 min | One row per claim; counts added to run log |
| B4 | Agent A revises → `run-01/spec.md` with change log; Agent B round 2 → `checks_round2.md` | 25 min | Every non-verified claim from round 1 handled |
| B5 | Analyze run-1 errors; write skill v2 in `docs/prompt_iterations.md` + `SKILL.md` | 20 min | Each new rule points to a run-1 claim ID |
| B6 | Run 2 (skill v2): draft + round-1 review (+ revise/round 2 if time) | 30 min | Run log row for run-02 round 1 |
| B7 | `docs/case_study.md`: one caught error end to end (claim → evidence → correction → prompt rule → result in run 2); copy finals to `outputs/spec.md`, `outputs/checks.md` | 25 min | Readable in 2 minutes |
| B8 | `README.md` + push to GitHub (new repo) | 25 min | Fresh reader understands problem, workflow, results |

Total ≈ 3.5 h. **If short on time, cut in this order:** B6 round 2 → README polish → B6 entirely (then the story becomes "here's the error, here's the prompt rule I'd add, I haven't measured it yet"; say that honestly in the interview). Never cut B3 or B7. They are the demo.

## 11. Agent prompts — required content

### `spec-reviewer.md` (Agent B)
Frontmatter: `name: spec-reviewer`, a `description` saying it verifies a spec against source code and is read-only, `tools: Read, Grep, Glob`.
The body must tell it to:
- read the spec file path it's given, list all claim IDs first, then check them one by one
- for each claim, open the cited code if a citation exists, **and** search (Grep) for counter-evidence. A citation is a lead, not proof.
- use only in-scope files (§4). Docs and comments are not behavior evidence.
- output exactly the §5.2 format, nothing else before the table
- be strict: "mostly right" is `incorrect` with a correction. Wording that overstates (e.g. "always", "all errors") counts.
- not suggest new features or style edits. Only accuracy and missing evidence.

### `write-spec/SKILL.md` (Agent A)
Frontmatter: `name: write-spec`, `description` saying when to use it (writing or revising a spec from the code in `input/`).
The body must contain: version line (`Skill version: v1`), the in-scope file list, the §5.1 template and claim rules, the revision procedure (for each non-verified row: fix to the reviewer's correction if Agent A agrees after re-reading the code, or remove, or move to Open questions; log every change with the claim ID), and where to save files (§3). v2 rules get added per §8 only.
