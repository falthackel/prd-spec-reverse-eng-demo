# Case study: "the app never crashes"

One error, followed end to end: the writer makes a claim, the reviewer catches it with line evidence, the spec gets fixed, the cause is traced to the writer's prompt, the prompt gets a rule, and the next run is measured.

Input: Demo A (an agentic financial assistant: LangChain + Gemini, 4 tools, terminal and Streamlit UIs) at commit `6caaf96`. Both agents read only the product code, never its docs.

---

## 1. The claim (run 1, skill v1)

The writer, following skill v1, wrote this in the Error handling section:

> **C-62** Unexpected exceptions are caught and logged, so neither the terminal chat nor the web page ever crashes.

It sounds right. Both interfaces do have a "last safety net" `except Exception` block, and the code comments even say "never crash the chat" and "never break the page."

## 2. The catch (reviewer, round 1)

The reviewer is a separate subagent with read-only tools (Read, Grep, Glob) and no access to the writer's reasoning. It opened the safety nets, then searched for code paths outside them:

| Claim | Verdict | Evidence |
|---|---|---|
| C-62 | **incorrect** | `cli.py:72-76`, `app.py:109-114`: the safety nets cover only the turn call. `cli.py:127`, `cli.py:151` (`start_session` / `reset_conversation`) and `app.py:37`, `app.py:70-82` (start / render) are not wrapped. |

**Correction:** "Unexpected exceptions raised while answering a question are caught, logged, and shown as 'Unexpected error' in both interfaces. Errors outside the turn (e.g. writing the log at session start or reset, page rendering) are not caught, so 'never crashes' is overstated."

Why this matters: a PM reading "never crashes" would skip error handling at session start and reset. The code comments said "never crash", and the writer repeated them. Only the code itself showed the gap.

## 3. The fix (revision, round 2)

The claim was narrowed to what the code shows:

> **C-62** Unexpected exceptions raised while answering a question are caught, logged, and shown as "Unexpected error: …" in both interfaces.

Round 2: **verified** (`cli.py:72-76` logs `[CLI ERROR]`; `app.py:109-114` logs `[UI ERROR]`).

That fixes this one spec. It doesn't stop the next spec from making the same mistake.

## 4. The cause (pattern across run 1)

C-62 was not alone. Of the 11 incorrect claims in run 1, three made the same kind of mistake, an overstated scope word:

| Claim | Overstated word | Reality |
|---|---|---|
| C-32 | "**All** tool failures are returned as `ERROR:`" | Partial `get_price_table` failures come back as `FAILED` rows inside a normal table |
| C-62 | "**never** crashes" | Only the per-question call is protected |
| C-65 | "**Every** tool failure is logged" | Framework-level tool errors are not |

The revision then made the same mistake once more: C-84, a claim *added* during the fix, said a label is always shown when it isn't. That's the evidence that the cause is **in the writer's instructions, not in one sentence**. Skill v1 never told the writer to check scope words against every code path, and never asked for a citation it would have to re-open.

## 5. The prompt change (skill v2)

Two rules were added to the writer's skill, both pointing at these claim IDs (full list in `docs/prompt_iterations.md`):

- **R2 Scope words:** don't write "all", "every", "always", "never", "any", or "only" unless you checked every code path it covers; otherwise say exactly which path it covers.
- **R7 Self-check:** before saving, re-open each cited range and read the whole function around it: early returns, `except` branches, fallbacks, and other callers.

Plus **R1** (cite `path:line` for every claim), so the self-check has something to re-open.

## 6. The result (run 2, skill v2)

Run 2 was written by a **fresh subagent with no memory of run 1**, against the same code, and checked by the same unchanged reviewer. The same behavior now reads:

> **C-88** In the terminal, any other `Exception` **during a turn** is logged as `[CLI ERROR] …` and shown as "⚠️ Unexpected error: …" without ending the chat loop (`cli.py:72-76`). → **verified**
>
> **C-91** On the web page, any other `Exception` is logged as `[UI ERROR] …` and shown as "Unexpected error: …" with the full message (`app.py:109-114`). → **verified**

It also found a detail run 1 never stated: an exception inside the live-display callback is caught one level earlier and becomes an agent error (C-80, `agent.py:163-175`, verified).

Note what changed and what didn't. Run 2 still uses scope words ("any", "every": 5 uses, about the same as run 1). But each one is now tied to a stated scope ("during a turn", "while streaming"), so it holds. "Never" no longer appears.

**Across the whole spec (first draft, before any revision):**

| Run | Skill | Claims | Incorrect | Unresolved |
|---|---|---|---|---|
| run-01 | v1 | 80 | 11 (13.8%) | 9 (11.3%) |
| run-02 | v2 | 116 | 2 (1.7%) | 2 (1.7%) |

## 7. What this does not prove

- **One run per version.** LLM output varies; a second v1 run could score better or worse.
- **v2 didn't eliminate the pattern.** Run 2's C-113 still overstated ("conversations are not persisted", while every question and answer is written to a log file on disk).
- **The reviewer varies too.** Twice, an unchanged claim was verified in round 1 and marked incorrect in round 2 (run-01 C-75, run-02 C-84). In run 1 the reviewer also miscounted its own summary (said 56/13/11; the rows were 60/11/9). That's why the main session recounts every table.
- **Writer setup differed.** Run 1 was written by the main session, run 2 by a fresh subagent.

The honest one-line version: *a read-only second agent caught a confident, plausible, wrong claim that came straight from code comments. The error pattern traced to one gap in the writer's prompt, and closing it cut the first-draft error rate from 13.8% to 1.7% in one comparison run.*
