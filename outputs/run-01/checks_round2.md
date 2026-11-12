| Claim | Verdict | Evidence | Required correction |
|---|---|---|---|
| C-01 | verified | `cli.py:130` "Asset Price Assistant"; `tools.py:45-52` labels USD/SGD/TWD/XAU/XAG/IHSG; `prompts.py:12-13` "questions about currency exchange rates and asset prices" | — |
| C-03 | verified | `cli.py:126-155` terminal `chat()`; `app.py:10-143` Streamlit page | — |
| C-04 | verified | `app.py:15,99` imports `agent` and calls `agent.run_turn`; `cli.py:7,62` calls `agent.run_request`; both use the same module-level `agent = build_agent(PROMPT_VERSION)` (`agent.py:53`) | — |
| C-05 | verified | `prompts.py:16-18` "Use get_price_table when the user asks for a 'price update', a 'market summary'"; `tools.py:368-381` table with price, % change, last update | — |
| C-06 | verified | `tools.py:320` `assets: str = "USD,SGD,TWD,XAU,IHSG"` | — |
| C-07 | verified | `tools.py:214-243` `get_exchange_rate` returns the latest rate for one pair | — |
| C-08 | verified | `tools.py:288-290` "({len(values)} days of data, min ..., max ...)" | — |
| C-09 | verified | `tools.py:272-274,297-311` `from_currency='IHSG'` goes to `_average_index`, "Average {index_code} close" | — |
| C-10 | verified | `prompts.py:30-36` v2 adds the WHY/web_search/source rule; `prompts.py:11-24` v1 has no such rule; `prompts.py:47` default "v2" | — |
| C-11 | verified | `agent.py:160` `messages = list(history) + [("human", question)]`; `agent.py:138` and `app.py:99-100` keep history across turns | — |
| C-12 | verified | `cli.py:150-153` `if question.lower() in ("reset", "clear"): agent.reset_conversation()` | — |
| C-13 | verified | `app.py:129` `st.button("🔄 Reset conversation", on_click=_start_conversation)` | — |
| C-14 | verified | `app.py:23-28,126-127,141-143` four EXAMPLES buttons set `pending`, which is then run as the question | — |
| C-15 | verified | `agent.py:165-170` `on_step(step)` called during streaming; `cli.py:51-53` prints; `app.py:91-95` `status.markdown(...)` | — |
| C-16 | verified | `cli.py:147` `if question.lower() in ("exit", "quit", "bye", "esc")` | — |
| C-17 | verified | `prompts.py:22-23` "answer normally without calling a tool"; `agent.py:50` all tools always bound, no enforcement code | — |
| C-18 | verified | `currency_agent.py:23-26` `from cli import chat` ... `chat()` | — |
| C-19 | verified | `currency_agent.py:21` `sys.stdout.reconfigure(encoding="utf-8", errors="replace")` before `cli` import | — |
| C-20 | verified | `cli.py:137-155` input loop; `cli.py:21-26,63-64` ERROR_MESSAGES mapping | — |
| C-21 | verified | `agent.py:33-38` `ChatGoogleGenerativeAI(...)`; `agent.py:50` `create_agent(llm, tools=ALL_TOOLS, ...)`; `tools.py:431` 4 tools | — |
| C-22 | verified | `agent.py:98,129-139` `_conversation_history` used by `run_request` (the CLI path) | — |
| C-23 | verified | `tools.py:85-208` helpers; `tools.py:214-422` four `@tool` functions | — |
| C-24 | verified | `prompts.py:43-46` `PROMPTS = {"v1": ..., "v2": ...}` | — |
| C-25 | verified | `trace_log.py:11-29` `LOG_FILE = "agent_log.txt"`, `start_session`/`log` write text | — |
| C-26 | verified | `app.py:10-143` Streamlit page | — |
| C-27 | verified | `app.py:35,99-100` history held in `st.session_state.history`; `agent.py:142-147` `run_turn` does not mutate shared history | — |
| C-28 | verified | `tools.py:229-245` output `1 {FROM} = {rate:.6g} {TO} (as of {latest_date})`, `max(cross.keys())`, `ERROR:` on empty or exception; `tools.py:100,108-113` Frankfurter cross via EUR | — |
| C-29 | verified | `tools.py:272-313` IHSG branch ignores `to_currency`, average close/trading days/min/max; `ERROR:` on empty or exception | — |
| C-30 | verified | `tools.py:320,337-341,347-366,368-387` default assets, header row, Rp/USD-oz/pts formats, `| - | - | FAILED: ... |` rows, `ERROR:` if all fail or no codes | — |
| C-31 | verified | `tools.py:412-418` `DDGS().text(query, max_results=4)`, `- [{site}] {title}: {body}`; `tools.py:413-414,421-422` `ERROR:` | — |
| C-32 | verified | `tools.py:244-245,293-294,312-313,421-422` except -> `_tool_error`; `tools.py:363-366` per-asset failures become FAILED rows | — |
| C-33 | verified | `tools.py:110-113` `cross[date] = rate_to / rate_from` | — |
| C-34 | verified | `tools.py:130-132` (currencies), `tools.py:159-161` (gold), `tools.py:184,195` (IHSG): latest vs previous available value | — |
| C-35 | verified | `tools.py:159-161` `pct_change = (usd_per_oz - prev_usd) / prev_usd * 100` | — |
| C-36 | verified | `tools.py:35,156` `idr_per_gram = idr_per_oz / GRAMS_PER_TROY_OZ` (31.1034768) | — |
| C-37 | verified | `tools.py:379` `arrow = "🔺" if pct >= 0 else "🔻"` | — |
| C-38 | verified | `tools.py:34,94-100` base URL; params only base/quotes/date, no key or headers | — |
| C-40 | verified | `tools.py:39` `{"IHSG": "^JKSE"}`; `tools.py:179` `yf.Ticker("^JKSE")` | — |
| C-41 | verified | `tools.py:25,412` `from ddgs import DDGS` / `DDGS().text(...)` | — |
| C-42 | verified | `agent.py:12,34` `ChatGoogleGenerativeAI(model="gemini-3.5-flash-lite")`; `requirements.txt:4` | — |
| C-43 | verified | `requirements.txt:2-11` every dependency uses `==` | — |
| C-44 | verified | Only `app.py:10` imports streamlit; `currency_agent.py`, `cli.py`, `agent.py`, `tools.py`, `prompts.py`, `trace_log.py` do not | — |
| C-45 | verified | `agent.py:19-21` `load_dotenv()` then `os.environ.get("GOOGLE_API_KEY", "").strip()` | — |
| C-46 | verified | `agent.py:21-24` `.strip()`; `if not API_KEY or API_KEY == "your-key-here": raise SystemExit("... copy .env.example to .env ...")` | — |
| C-47 | verified | `agent.py:43` `os.environ.get("PROMPT_VERSION", DEFAULT_PROMPT_VERSION)`; `prompts.py:47` `"v2"` | — |
| C-48 | verified | `agent.py:44-45` `raise SystemExit(f"Unknown PROMPT_VERSION ... Choose one of: {', '.join(PROMPTS)}")` | — |
| C-49 | verified | `agent.py:36` `timeout=30` | — |
| C-50 | verified | `agent.py:37` `max_retries=2` | — |
| C-51 | verified | `agent.py:33-38` no `temperature` argument | — |
| C-52 | verified | `tools.py:100` `requests.get(..., timeout=15)` | — |
| C-53 | verified | `tools.py:36,180,186,204-205` every `history(...)` call passes `timeout=YF_TIMEOUT` (10) | — |
| C-54 | verified | `.streamlit/config.toml:5` `address = "127.0.0.1"` | — |
| C-55 | verified | `.streamlit/config.toml:9` `gatherUsageStats = false` | — |
| C-56 | verified | `trace_log.py:11` relative path `"agent_log.txt"` | — |
| C-57 | verified | `agent.py:67-76` keyword match on `f"{type(error).__name__} {error}".lower()` -> quota/timeout/auth/other | — |
| C-58 | verified | `cli.py:21-26` ERROR_MESSAGES; `cli.py:64` and `app.py:105` both use this dict | — |
| C-59 | verified | `agent.py:138` and `app.py:99-100` history is assigned only when `run_turn` returns; `agent.py:160` copies history | — |
| C-60 | verified | `cli.py:65-66` `if e.kind == "other": print(f"   Reason: {_summarize(str(e))}")` | — |
| C-61 | verified | `cli.py:69-71` `except KeyboardInterrupt:` prints "[Cancelled]" and returns; the loop continues at `cli.py:137` | — |
| C-62 | verified | `cli.py:72-76` logs "[CLI ERROR]" and prints "Unexpected error: ..."; `app.py:109-114` logs "[UI ERROR]" and shows "Unexpected error: {e}" | — |
| C-63 | verified | `agent.py:185-189` `if not answer: answer = EMPTY_ANSWER_MESSAGE` ("Please try again or rephrase") | — |
| C-64 | verified | `agent.py:177-180` `raise AgentRunError("other", "the agent stopped without a final answer")` | — |
| C-65 | verified | `tools.py:64-67` `_tool_error` logs "❌ [TOOL FAILED]"; `tools.py:365` per-asset failure logs "[TOOL FAILED]" | — |
| C-66 | verified | `prompts.py:20-21` "if a structured data tool fails ... Try the web_search tool" (in v1, so inherited by v2) | — |
| C-67 | verified | `tools.py:187-193` if intraday empty, `price = float(daily.iloc[-1]["Close"])` | — |
| C-68 | verified | `tools.py:301-305` "The market may have been closed on those days (weekend or holiday)." | — |
| C-69 | verified | `tools.py:229-233` passes codes straight to `_fetch_eur_cross_series`; `tools.py:91-96` only uppercases them and handles EUR | — |
| C-70 | verified | `tools.py:357-366` any exception for an asset -> `rows.append((label, None, None, f"FAILED: {e}"))` | — |
| C-71 | verified | `trace_log.py:14-15` `open(LOG_FILE, "w", ...)` in `start_session` | — |
| C-72 | verified | `app.py:34-37,44-45` `_start_conversation` calls `trace_log.start_session()` (mode "w") for each new session or reset; one shared `LOG_FILE` (`trace_log.py:11`) | — |
| C-73 | verified | `cli.py:11-15,98-102` ESC detection only with `msvcrt`; otherwise `input()`, where EOFError/KeyboardInterrupt -> None -> quit | — |
| C-74 | verified | `prompts.py:8,12` `TODAY` computed at import; `agent.py:53` agent built once at import | — |
| C-75 | incorrect | `app.py:54` `text.replace("$", "\\$").replace("<br>", " ")` | The web page escapes `$` and replaces the exact string `<br>` with a space (it does not strip other forms like `<br/>` or `<BR>`) before showing user and model text as Markdown. |
| C-76 | verified | `tools.py:154` `cross_idr.get(latest_date) or cross_idr[sorted(cross_idr.keys())[-1]]` | — |
| C-77 | verified | No login, account, or auth code in any in-scope `.py` (the only grep hits for "auth" are the error kind at `agent.py:74-75`) | — |
| C-78 | verified | `app.py:133` "Not financial advice." | — |
| C-79 | verified | `agent.py:98` in-memory `_conversation_history`; `app.py:35` in-memory session state; nothing reloads history | — |
| C-80 | verified | `tools.py:34,39` only Frankfurter and yfinance `^JKSE`; no crypto source | — |
| C-81 | verified | `prompts.py:37-39` "call get_average_exchange_rate with start_date and end_date both set to that date" | — |
| C-82 | verified | `tools.py:73-79` `f"{date_str} • checked {_now_hm()}"`, local `datetime.now()`; used at `tools.py:353,360` | — |
| C-83 | verified | `agent.py:124` `ok = message.status != "error" and not content.startswith(TOOL_ERROR_PREFIX)` | — |
| C-84 | incorrect | `app.py:102` the label is set from `_steps_label` only on success; `app.py:107,113` failed turns show "Failed" live; `app.py:75-76` an earlier turn with no steps gets no expander, so it shows no "No tool needed" label | The label from `_steps_label` ("No tool needed" / "🔧 N tool call(s)" plus " · ❌ N failed") appears on the live status box of a successful turn and on the expander of earlier turns that had steps. A failed turn's live status reads "Failed", and earlier turns with no steps show no label. |
| C-85 | verified | `app.py:131` `st.caption(f"Prompt {agent.PROMPT_VERSION} · {agent.MODEL_NAME}...")` | — |
| C-86 | verified | `tools.py:32` `logging.getLogger("yfinance").setLevel(logging.CRITICAL)` | — |
| C-87 | verified | `app.py:14-18` `except SystemExit as e: st.error(str(e)); st.stop()` | — |
| C-88 | verified | `tools.py:186-188` `history(period="1d", interval="1m")`, `price = float(intraday.iloc[-1]["Close"])` | — |
| C-89 | verified | `tools.py:180,184,195` `period="5d"`, `prev_close = daily.iloc[-2]["Close"]` | — |
| C-90 | verified | `tools.py:199-208` returns `history["Close"]` values from daily history | — |
| C-91 | verified | `cli.py:67,70` notice for AgentRunError and KeyboardInterrupt; `cli.py:75` none for other exceptions; `app.py:106` notice vs `app.py:112` none | — |

**Summary counts:** total: 89, verified: 87, incorrect: 2, unresolved: 0

**Omissions (max 5):**
- An exception raised inside the `on_step` display callback (terminal print or web `status.markdown`) happens inside the agent stream's `try`. It is turned into an `AgentRunError` and classified, so it never reaches the "Unexpected error" safety net — `agent.py:165-175`
- On non-Windows systems, Ctrl+C or end-of-input at the prompt prints "[ESC pressed] Closing the program." even though ESC was not pressed. On Windows, the input prompt reads Ctrl+C as an ordinary character rather than quitting — `cli.py:98-102,107-120,140-141`
- In the price table, the change column shows "n/a" when there is only one data point (no previous day) — `tools.py:376-377`
- The stored conversation history includes all tool-call and tool-result messages, not just questions and answers — `agent.py:183`
- `cli.ask()` sends a single question outside chat mode, and starts a new log session only if there is no history yet — `cli.py:81-86`

**Reviewer notes:**
- C-02 and C-39: removed, not checked.
- Claims were checked in numeric ID order, which differs from the spec's display order (C-81 to C-91 are spread through the body).
- C-27, C-49, C-50: verified only as what the code sets up (Streamlit `session_state`, and the `timeout` and `max_retries` arguments). That Streamlit isolates sessions per tab, and the exact retry and timeout semantics, are library behavior outside the in-scope code.
- C-57: the "stopped without a final answer" case gets `other` directly rather than by keyword matching (that case is covered by C-64). C-57 is treated as describing exceptions raised during the stream.
- C-62: the handlers call `trace_log.log` themselves, so a failure to write the log file inside a handler would not be caught. This edge case was not counted against the claim.
- Grep with glob `*.py` returned hits only from in-scope files. No out-of-scope files were opened.
