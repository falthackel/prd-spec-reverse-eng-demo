# PRD / Spec Reverse-Engineering with Multi-Agent Validation

## 1. Project background

Legacy code often has missing or outdated docs. This project uses one AI agent to reverse-engineer an existing codebase into a spec (PRD), and a second agent with **read-only access and a separate context** to check every claim in that spec against the code, with file-and-line evidence. The reviewer's findings fix the spec, and they also improve the writer's prompt, which is then measured on a new run.

The codebase being reverse-engineered is [Demo A](https://github.com/falthackel/agentic-financial-monitoring-assistant), an agentic financial assistant, pinned to tag `demo-v1` (commit `6caaf96`).

Workflow, results and limits: see **[docs/README.md](docs/README.md)**.

## 2. Project directory

```
.
├── .claude/
│   ├── skills/write-spec/SKILL.md   # Agent A (writer): how to write and revise a spec
│   ├── agents/spec-reviewer.md      # Agent B (reviewer): read-only, checks every claim
│   └── settings.json                # Blocks edits to input/ and reading Demo A's docs/tests
├── input/demo-a/                    # Demo A source code (you clone it; not in this repo)
├── outputs/
│   ├── run-01/ … run-03/            # Each run: spec_draft, checks_round1, spec, checks_round2
│   ├── spec.md                      # Latest final spec
│   └── checks.md                    # Latest final review
├── docs/
│   ├── README.md                    # Workflow and results
│   ├── prompt_iterations.md         # Writer prompt v1 → v2: errors found, rules added
│   └── case_study.md                # One caught error, end to end
└── progress.md                      # Step log and run log table
```

## 3. Environment setup

Requirements:
- [Claude Code](https://claude.com/claude-code)
- Git

No Python environment, API keys or package installs are needed. Demo A is only read, never run.

```bash
git clone https://github.com/falthackel/prd-spec-reverse-eng-demo.git
cd prd-spec-reverse-eng-demo
git clone --branch demo-v1 https://github.com/falthackel/agentic-financial-monitoring-assistant.git input/demo-a
git -C input/demo-a rev-parse --short HEAD   # must print 6caaf96
```

## 4. Steps to run

Open Claude Code in the project folder and send these prompts one at a time (replace `run-04` with the next run number):

1. **Write the draft:** *"Start run-04: have a fresh subagent follow the write-spec skill and write `outputs/run-04/spec_draft.md`."*
2. **Review it:** *"Call spec-reviewer on `outputs/run-04/spec_draft.md` and save its output verbatim to `outputs/run-04/checks_round1.md`."*
3. **Revise:** *"Have a fresh subagent revise run-04 using `checks_round1.md`, saving to `outputs/run-04/spec.md`."*
4. **Re-review:** *"Call spec-reviewer on `outputs/run-04/spec.md` and save its output verbatim to `outputs/run-04/checks_round2.md`."*

The result is a spec where every claim is checked against the code. Each checks file ends with summary counts (verified / incorrect / unresolved).
