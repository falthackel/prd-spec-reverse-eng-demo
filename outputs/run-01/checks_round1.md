| Claim | Verdict | Evidence | Required correction |
|---|---|---|---|
| C-01 | verified | `cli.py:130` — `print("Asset Price Assistant")`; `tools.py:45-52` ASSET_LABELS has USD/SGD/TWD/XAU/XAG/IHSG; `prompts.py:12-13` "questions about currency exchange rates and asset prices" | — |
| C-02 | unresolved | `tools.py:358` prices quoted in IDR; `app.py:123` caption "in rupiah". This shows a design choice, not who the users are. | This is a claim about users. It needs product or user-research evidence, which the code does not contain. |
| C-03 | verified | `cli.py:126-155` terminal `chat()` loop; `app.py:10-143` Streamlit page | — |
| C-04 | verified | `app.py:15` `import agent` and `app.py:99` `agent.run_turn(...)`; `cli.py:62` `agent.run_request(...)`; both use the module-level `agent = build_agent(PROMPT_VERSION)` at `agent.py:53` with `ALL_TOOLS` | — |
| C-05 | unresolved | `tools.py:368-371` the tool emits a table with Asset/Price/Change/Last Update; `prompts.py:16-18` tells the model to use get_price_table for "market summary". The final answer is written by the model. | The tool output is confirmed. Whether the user actually gets one table in the final answer depends on the model at runtime. That needs live run transcripts. |
| C-06 | verified | `tools.py:320` — `def get_price_table(assets: str = "USD,SGD,TWD,XAU,IHSG")` | — |
| C-07 | verified | `tools.py:214-245` get_exchange_rate returns the latest rate for one pair | — |
| C-08 | verified | `tools.py:286-290` — average, `({len(values)} days of data, min ..., max ...)` | — |
| C-09 | verified | `tools.py:272-274`, `tools.py:306-309` — IHSG route returns "Average IHSG close = ... points" | — |
| C-10 | incorrect | `prompts.py:30-36` the "use web_search ... say which source each reason comes from" rule exists only in SYSTEM_PROMPT_V2; `prompts.py:11-24` v1 has no such rule | The instruction to search the web for "why" questions and name each reason's source is only in prompt v2 (the default). Under `PROMPT_VERSION=v1` it is absent. Whether the model follows it is runtime behavior. |
| C-11 | verified | `agent.py:160` — `messages = list(history) + [("human", question)]`; history kept at `agent.py:138` (terminal) and `app.py:99-100` (web) | — |
| C-12 | verified | `cli.py:150-153` — `if question.lower() in ("reset", "clear"): agent.reset_conversation()` | — |
| C-13 | verified | `app.py:121`, `app.py:129` — sidebar `st.button("🔄 Reset conversation", on_click=_start_conversation)` | — |
| C-14 | verified | `app.py:23-28` four EXAMPLES; `app.py:126-127` a button for each sets `pending`; `app.py:141-143` pending is run as the question | — |
| C-15 | verified | `agent.py:165-170` on_step is called per streamed step; `cli.py:51-53` prints each step; `app.py:91-95` `st.status(...)` then `status.markdown(...)` | — |
| C-16 | incorrect | `cli.py:11-15`, `cli.py:98-102` — without msvcrt (not Windows), ESC is not detected and plain `input()` is used | Typing `exit`, `quit`, `bye`, or `esc` quits on every platform. Pressing ESC quits only on Windows (msvcrt). On other systems the user quits with Ctrl+C or EOF instead. |
| C-17 | unresolved | `prompts.py:22-23` — the prompt says "answer normally without calling a tool"; no code enforces it | This is a prompt instruction only. Whether the model complies needs runtime transcripts. |
| C-18 | verified | `currency_agent.py:23-26` — `from cli import chat` ... `chat()` under `__main__` | — |
| C-19 | verified | `currency_agent.py:21` — `sys.stdout.reconfigure(encoding="utf-8", errors="replace")` | — |
| C-20 | verified | `cli.py:137-155` input loop; `cli.py:21-26`, `cli.py:63-68` ERROR_MESSAGES mapping | — |
| C-21 | verified | `agent.py:33-38` ChatGoogleGenerativeAI; `agent.py:48-53` `create_agent(llm, tools=ALL_TOOLS, ...)`; `tools.py:431` 4 tools | — |
| C-22 | verified | `agent.py:98` `_conversation_history = []`; `agent.py:129-139` run_request uses and updates it (called from `cli.py:62`) | — |
| C-23 | verified | `tools.py:55-208` helpers; `tools.py:214-431` 4 `@tool` functions and `ALL_TOOLS` | — |
| C-24 | verified | `prompts.py:43-46` — `PROMPTS = {"v1": ..., "v2": ...}` | — |
| C-25 | verified | `trace_log.py:11-29` — LOG_FILE "agent_log.txt", start_session/log_request/log | — |
| C-26 | verified | `app.py:10-143` Streamlit page | — |
| C-27 | verified | `app.py:34-45` history/turns stored in `st.session_state`; `app.py:99-100` per-session history passed to `run_turn` | — |
| C-28 | verified | `tools.py:233-245` — `f"1 {FROM} = {rate:.6g} {TO} (as of {latest_date})"`; ERROR on empty cross (235) or exception (245); `tools.py:85-115` EUR cross via `API_BASE` v2 | — |
| C-29 | verified | `tools.py:272-294` currency path; `tools.py:297-313` IHSG path via yfinance; ERROR on no data (281-284, 301-305) or exception (294, 313) | — |
| C-30 | incorrect | `tools.py:340-341` — `if not asset_list: return _tool_error(call, "no asset codes given.")` | Each failed asset becomes a row with `-` for price and change and `FAILED: <reason>` in Last Update (`tools.py:366`, `tools.py:374`). The tool returns `ERROR:` when every asset fails (`tools.py:384-387`) and also when no asset codes are given (`tools.py:340-341`). |
| C-31 | verified | `tools.py:412-422` — `DDGS().text(query, max_results=4)`, format `- [{site}] {title}: {body}`, ERROR on no results or exception | — |
| C-32 | incorrect | `tools.py:363-366` — a single asset failure in get_price_table is a `FAILED` row inside a normal table with no `ERROR:` prefix (`tools.py:389-390`); `tools.py:231`, `tools.py:339` — `log(...)` runs outside try, so a log-file error would raise | Failures caught inside tool bodies are returned as `ERROR:` text, except partial get_price_table failures, which appear as `FAILED` rows in a successful-looking table. Calls outside the try blocks (e.g. the first `log()` call) are not protected. |
| C-33 | verified | `tools.py:110-113` — `cross[date] = rate_to / rate_from` | — |
| C-34 | verified | `tools.py:130-132` currencies: last vs second-to-last date; `tools.py:184`, `tools.py:195` IHSG: latest price vs previous daily close; `tools.py:376-377` "n/a" if no previous day | — |
| C-35 | verified | `tools.py:159-161` — pct_change computed from `cross_usd` | — |
| C-36 | verified | `tools.py:35`, `tools.py:156` — `idr_per_gram = idr_per_oz / GRAMS_PER_TROY_OZ` (31.1034768) | — |
| C-37 | verified | `tools.py:379` — `arrow = "🔺" if pct >= 0 else "🔻"` | — |
| C-38 | unresolved | `tools.py:34` URL verified; `tools.py:94-100` no key or auth is sent | The URL and the fact that no key is sent are confirmed. Whether the service accepts keyless requests needs a live API response or the provider's docs. The only in-code statement (`tools.py:5`) is a comment, intent only. |
| C-39 | unresolved | `tools.py:7` — "Frankfurter data is DAILY, not real-time" (comment, intent only); `tools.py:77-79` code formats the date as a daily rate | This is a property of the external API. It needs live API responses or provider docs. |
| C-40 | incorrect | `tools.py:180` `history(period="5d")` daily; `tools.py:190-193` daily fallback; `tools.py:204-205` averages use daily history without 1m interval | IHSG comes from yfinance ticker `^JKSE`. The price table uses 1-minute intraday data for the current price, falls back to daily data, and gets the previous close from 5-day daily data. Date-range averages use daily closes. |
| C-41 | verified | `tools.py:25` `from ddgs import DDGS`; `tools.py:412` | — |
| C-42 | verified | `agent.py:12`, `agent.py:33-34` — `ChatGoogleGenerativeAI(model="gemini-3.5-flash-lite", ...)`; `requirements.txt:4` | — |
| C-43 | unresolved | `requirements.txt:2-11` all `==` pins (verified); `requirements.txt:1` "Python 3.10 — versions pinned to the tested environment" (comment, intent only) | The exact pins are confirmed. "Tested on Python 3.10" comes only from a comment. Confirming it needs CI config or test logs, which are not in scope. |
| C-44 | verified | Grep: `streamlit` is imported only in `app.py:10`; `currency_agent.py`, `cli.py`, `agent.py`, `tools.py`, `prompts.py`, and `trace_log.py` do not import it | — |
| C-45 | incorrect | `agent.py:19-21` — `load_dotenv()` then `os.environ.get("GOOGLE_API_KEY", "")` | `GOOGLE_API_KEY` is required and is read from the process environment. `python-dotenv` loads it from a `.env` file if present, but a real environment variable also works. |
| C-46 | incorrect | `agent.py:21-24` SystemExit with "copy .env.example to .env and add your key"; `app.py:14-18` the web page catches SystemExit and shows `st.error(...)` + `st.stop()` | If the key is missing, empty after strip, or equal to `your-key-here`, the terminal program exits with that message. On the web page the message is shown as an error and the page script stops, but the Streamlit server does not exit. |
| C-47 | verified | `agent.py:43` — `os.environ.get("PROMPT_VERSION", DEFAULT_PROMPT_VERSION)`; `prompts.py:47` `DEFAULT_PROMPT_VERSION = "v2"` | — |
| C-48 | incorrect | `agent.py:44-45` SystemExit listing versions; `app.py:14-18` the web page catches it and shows `st.error` + `st.stop()` | An unknown `PROMPT_VERSION` raises SystemExit with a message listing valid versions. The terminal program exits. The web page shows the message as an error and stops the page script, and the server keeps running. |
| C-49 | verified | `agent.py:36` — `timeout=30,      # seconds per Gemini request` | — |
| C-50 | verified | `agent.py:37` — `max_retries=2` | — |
| C-51 | verified | `agent.py:33-38` — no temperature argument | — |
| C-52 | verified | `tools.py:100` — `requests.get(..., timeout=15)` (the only Frankfurter request) | — |
| C-53 | verified | `tools.py:36`, `tools.py:180`, `tools.py:186`, `tools.py:204-205` — every `history()` call passes `timeout=YF_TIMEOUT` (10) | — |
| C-54 | verified | `.streamlit/config.toml:5` — `address = "127.0.0.1"` | — |
| C-55 | verified | `.streamlit/config.toml:9` — `gatherUsageStats = false` | — |
| C-56 | verified | `trace_log.py:11` — relative path `LOG_FILE = "agent_log.txt"`, opened at `trace_log.py:15`, `trace_log.py:28` | — |
| C-57 | verified | `agent.py:67-76` — keyword match on `f"{type(error).__name__} {error}".lower()` returns quota/timeout/auth/other | — |
| C-58 | verified | `cli.py:21-26` ERROR_MESSAGES; `cli.py:64` and `app.py:105` both use the same dict | — |
| C-59 | incorrect | `cli.py:72-76` and `app.py:109-114` — unexpected exceptions show "Unexpected error: ..." without the "not saved" notice | History is unchanged on any failed turn (`agent.py:182-183`, `app.py:99`). The "not saved" notice is shown only for AgentRunError (`cli.py:67`, `app.py:106`) and terminal Ctrl+C (`cli.py:70`), not for other unexpected exceptions. |
| C-60 | verified | `cli.py:65-66` — `if e.kind == "other": print(f"   Reason: {_summarize(str(e))}")` | — |
| C-61 | verified | `cli.py:69-71` — `except KeyboardInterrupt: print("\n[Cancelled] ...")`; `return None`, and the loop continues | — |
| C-62 | incorrect | `cli.py:72-76`, `app.py:109-114` — the safety nets cover only the turn call; `cli.py:127`, `cli.py:151` (`start_session`/`reset_conversation`) and `app.py:37`, `app.py:70-82` (start/render) are not wrapped | Unexpected exceptions raised while answering a question are caught, logged, and shown as "Unexpected error" in both interfaces. Errors outside the turn (e.g. writing the log at session start or reset, page rendering) are not caught, so "never crashes" is overstated. |
| C-63 | verified | `agent.py:185-189` — `if not answer: ... answer = EMPTY_ANSWER_MESSAGE` (`agent.py:26`) | — |
| C-64 | verified | `agent.py:177-180` — `raise AgentRunError("other", "the agent stopped without a final answer")` | — |
| C-65 | incorrect | `agent.py:124` — a tool result is treated as failed if `message.status == "error"`, a path not produced by `_tool_error` and not logged by the tools | Failures handled inside the tools are logged with `[TOOL FAILED]` (`tools.py:66`, `tools.py:365`). Tool errors produced by the framework (ToolMessage with `status == "error"`, e.g. tool invocation errors) are not logged with `[TOOL FAILED]`. |
| C-66 | verified | `prompts.py:20-21` — "if a structured data tool fails ... Try the web_search tool" (in v1, inherited by v2 at `prompts.py:30`) | — |
| C-67 | verified | `tools.py:186-193` — when `intraday.empty`, uses `daily.iloc[-1]["Close"]` | — |
| C-68 | verified | `tools.py:301-305` — "The market may have been closed on those days (weekend or holiday)." | — |
| C-69 | unresolved | `tools.py:232-245` — no crypto or index routing; any code is sent to Frankfurter; the "Does NOT support crypto or stock indexes" text at `tools.py:219-220` is a docstring (comment, intent only) | The code adds no special handling, so the result depends on Frankfurter's symbol coverage. Confirming it needs live API responses for crypto and index codes. |
| C-70 | verified | `tools.py:358`, `tools.py:123-124`, `tools.py:363-366` — empty data raises ValueError or HTTP error, caught as a `FAILED` row | — |
| C-71 | verified | `trace_log.py:14-15` — `open(LOG_FILE, "w", ...)` at session start | — |
| C-72 | verified | `trace_log.py:11` single fixed path; `app.py:34-37`, `app.py:44-45`, `app.py:129` every new or reset tab conversation calls `trace_log.start_session()` | — |
| C-73 | verified | `cli.py:11-15`, `cli.py:98-102` — no msvcrt, so `input()`; `EOFError`/`KeyboardInterrupt` return None, which quits (`cli.py:140-142`) | — |
| C-74 | verified | `prompts.py:8` `TODAY` set at import; `agent.py:53` agent built once with that prompt | — |
| C-75 | verified | `app.py:54` — `text.replace("$", "\\$").replace("<br>", " ")` | — |
| C-76 | verified | `tools.py:154` — `cross_idr.get(latest_date) or cross_idr[sorted(cross_idr.keys())[-1]]` | — |
| C-77 | verified | No login, user, or auth code in `app.py`, `cli.py`, or `agent.py`; the only key is the Gemini API key (`agent.py:21`) | — |
| C-78 | unresolved | `app.py:133` — the caption "Not financial advice." is the only mention; `prompts.py:11-40` has no rule against giving advice | The code has only a UI disclaimer and nothing that stops the model from giving advice. Actual behavior needs runtime transcripts. |
| C-79 | verified | `agent.py:98` in-memory history; `app.py:35` session_state; no persistence code; `trace_log.py:15` log overwritten at next session | — |
| C-80 | unresolved | `tools.py:85-115` — no crypto-specific source exists, but crypto codes would still be sent to Frankfurter; `tools.py:12-13` "for example crypto" is a comment, intent only | "No dedicated crypto source" is confirmed by the code. "Only through web search" depends on Frankfurter not returning crypto rates, which needs a live API response. |

**Summary counts:** total: 80, verified: 56, incorrect: 13, unresolved: 11

**Omissions (max 5):**
- Prompt v2 tells the agent to answer a single past-date rate by calling get_average_exchange_rate with start_date = end_date, because get_exchange_rate only gives the latest rate — `prompts.py:37-39`
- For Frankfurter assets (currencies and gold), "Last Update" shows the rate date plus the local time of the check (`<date> • checked HH:MM`), not a market timestamp — `tools.py:77-79`
- A tool step counts as failed when its result starts with `ERROR:` or has ToolMessage status "error". The web page uses this to label each turn ("No tool needed", "N tool calls · ❌ N failed") — `agent.py:124`, `app.py:61-67`
- The web sidebar shows the active prompt version and the model name — `app.py:131`
- yfinance logging is set to CRITICAL so "possibly delisted" warnings for non-trading days are hidden — `tools.py:32`

**Reviewer notes:**
- No claims were marked removed (no strikethrough). Q-01 to Q-05 are open questions and were not given verdicts.
- C-50: verified from the literal `max_retries=2` setting. Whether that means 2 retries or 2 total attempts is defined in `langchain-google-genai`, which is outside scope.
- C-46 and C-48 are marked incorrect only because of the web-page path (SystemExit is caught in `app.py:14-18`). For the terminal they are accurate.
- C-65's counter-evidence relies on `agent.py:124` handling `status == "error"` ToolMessages. When LangChain actually produces these is library behavior and is not visible in scope.
- C-69 and C-80 are unresolved, not incorrect: the code has no crypto or index handling, so the outcome depends on Frankfurter's live coverage.
- Spec file reviewed: e:\Project\Spec_ReverseEng\outputs\run-01\spec_draft.md. Code read: e:\Project\Spec_ReverseEng\input\demo-a\ (in-scope files only).
