| Claim | Verdict | Evidence | Required correction |
|---|---|---|---|
| C-01 | verified | `cli.py:130`, `app.py:12`, `app.py:122`, `app.py:135`: "Asset Price Assistant" in banner, page_title, sidebar header, st.title | — |
| C-02 | verified | `agent.py:33-38`, `agent.py:50`: `create_agent(llm, tools=ALL_TOOLS, ...)`; `tools.py:431` lists 4 tools | — |
| C-03 | verified | `currency_agent.py:23-26`: `from cli import chat` / `chat()`; `app.py:10`: `import streamlit as st` | — |
| C-04 | verified | `cli.py:62`: `agent.run_request(...)`; `app.py:99-100`: `agent.run_turn(...)`; both use module-level `agent = build_agent(PROMPT_VERSION)` at `agent.py:53` | — |
| C-05 | verified | `tools.py:320`: `def get_price_table(assets: str = "USD,SGD,TWD,XAU,IHSG")` | — |
| C-06 | verified | `tools.py:233-243`: fetches cross series, returns rate for latest date | — |
| C-07 | verified | `tools.py:276-292`: cross series with `{"from": start_date, "to": end_date}`, mean computed | — |
| C-08 | verified | `tools.py:272-274`: `if index_code in INDEX_TICKERS: return _average_index(...)`; `tools.py:306-309` mean of closes | — |
| C-09 | verified | `tools.py:412`: `DDGS().text(query, max_results=4)` | — |
| C-10 | verified | `agent.py:160`: `messages = list(history) + [("human", question)]`; `agent.py:183` full state messages kept; `agent.py:138`, `app.py:99-100` store it | — |
| C-11 | verified | `agent.py:165-170` calls `on_step(step)` during stream; `cli.py:62` passes `print_step`; `app.py:93-95,100` passes `on_step` | — |
| C-12 | verified | `cli.py:144` strip; `cli.py:150-153`: `question.lower() in ("reset", "clear")` → `agent.reset_conversation()` and prints notice; `agent.py:108` clears history | — |
| C-13 | verified | `app.py:129`: button `on_click=_start_conversation`; `app.py:35-36` reset `history` and `turns` | — |
| C-14 | verified | `app.py:23-28` four EXAMPLES; `app.py:41` sets `pending`; `app.py:141`: `question = typed or st.session_state.pop("pending", None)` | — |
| C-15 | verified | `cli.py:108-110` ESC returns None; `cli.py:140-142` quits; `cli.py:147`: `question.lower() in ("exit", "quit", "bye", "esc")` | — |
| C-16 | verified | `trace_log.py:11` LOG_FILE; `agent.py:158` log_request; `tools.py:231`, `tools.py:242`, `tools.py:66` tool lines; `agent.py:190-191` final answer | — |
| C-17 | verified | `cli.py:131` examples line; `cli.py:134` prints `trace_log.LOG_FILE` | — |
| C-18 | verified | `currency_agent.py:21`: `sys.stdout.reconfigure(encoding="utf-8", errors="replace")` unconditionally, before import at line 23 | — |
| C-19 | verified | `currency_agent.py:25-26`: `if __name__ == "__main__": chat()` | — |
| C-20 | verified | `cli.py:21-26`, `cli.py:43-78`, `cli.py:92-155`: error texts, step printing, answer printing, input loop | — |
| C-21 | verified | `agent.py:19-24`, `agent.py:33-53`, `agent.py:67-76`, `agent.py:98-192` | — |
| C-22 | verified | `tools.py:85-208` helpers; `tools.py:214-428` tools; `tools.py:431` ALL_TOOLS | — |
| C-23 | verified | `prompts.py:43-46`: `PROMPTS = {"v1": ..., "v2": ...}` | — |
| C-24 | verified | `trace_log.py:15` mode "w"; `trace_log.py:21-24`, `trace_log.py:28` mode "a" | — |
| C-25 | verified | `app.py:10-143`: module-level script, no `__main__` block | — |
| C-26 | incorrect | `app.py:58` wraps `cli.format_step` in backticks and replaces backticks; `app.py:105-106` appends " This question was not saved to the conversation." vs terminal `cli.py:67` "(This question was not saved to the conversation. Details are in the log.)"; `app.py:112` full error vs `cli.py:75` summary | `app.py` imports `cli` to reuse `format_step` and `ERROR_MESSAGES`; the per-kind base messages and step text are shared, but surrounding error wording differs (different "not saved" note, no `Reason:` line for `other`, full vs summarized unexpected-error text) and step text is wrapped in backticks with backticks replaced by `'`. |
| C-27 | verified | `.streamlit/config.toml:1-9`: `[server]` and `[browser]` sections | — |
| C-28 | verified | `requirements.txt:2-11`: 10 lines, all `==`, versions as stated | — |
| C-29 | verified | `requirements-dev.txt:2-4`: `-r requirements.txt`, `pytest==9.2.0`, `PyYAML==6.0.3` | — |
| C-30 | verified | `pytest.ini:2-3`: `testpaths = tests`, `pythonpath = .` | — |
| C-31 | verified | `tools.py:233` calls helper without date_or_range; `tools.py:234-238` empty → error suggesting web_search; `tools.py:241` format; `tools.py:244-245` exception text | — |
| C-32 | verified | `tools.py:276-279`, `tools.py:281-294`, `tools.py:299-313`, `tools.py:204-205`: sources, outputs and error texts as stated | — |
| C-33 | verified | `tools.py:347-360` sources; `tools.py:369` header columns; `tools.py:363-366` FAILED row; `tools.py:384-387` all-failed ERROR | — |
| C-34 | unresolved | `tools.py:412`: `DDGS().text(query, max_results=4)`; `tools.py:417-418` one line per returned result, no cap in code | Format `- [site] title: body` per result and error texts are verified; "up to 4 lines" depends on the `ddgs` library honoring `max_results` (and bodies containing no newlines), which is not enforced in in-scope code; needs ddgs library source/runtime output. |
| C-35 | verified | `tools.py:231`, `tools.py:271`, `tools.py:339`, `tools.py:410`: `log(f"🔧 [TOOL CALLED] {call}")` before fetch | — |
| C-36 | verified | `tools.py:64-67`: logs `❌ [TOOL FAILED] {call} -> {reason}`, returns `f"{TOOL_ERROR_PREFIX} {reason}"` | — |
| C-37 | verified | `agent.py:124`: `ok = message.status != "error" and not content.startswith(TOOL_ERROR_PREFIX)` | — |
| C-38 | verified | `tools.py:431`; `agent.py:50` | — |
| C-39 | verified | `tools.py:91-96`: upper, `sorted(set([from_c, to_c]) - {"EUR"})`, `quotes` only if non-empty | — |
| C-40 | verified | `tools.py:110-113`: `1.0 if ... == "EUR"`, `rate_to / rate_from` | — |
| C-41 | verified | `tools.py:112`: `if rate_from and rate_to:` (None or 0 skipped) | — |
| C-42 | verified | `tools.py:91`: `from_currency.upper(), to_currency.upper()` only | — |
| C-43 | verified | `tools.py:239`: `latest_date = max(cross.keys())` | — |
| C-44 | verified | `tools.py:241`: `{rate:.6g}` | — |
| C-45 | verified | `tools.py:39` `INDEX_TICKERS = {"IHSG": "^JKSE"}`; `tools.py:272-273` | — |
| C-46 | verified | `tools.py:286-287`: `sum(values) / len(values)` | — |
| C-47 | verified | `tools.py:276-279`: dates passed straight into params | — |
| C-48 | incorrect | `tools.py:268-270` builds `call` including `to_currency=...`; `call` is passed to `_average_index` (`tools.py:274`) and logged via `_tool_error` (`tools.py:302`, `tools.py:313`) | On the IHSG path, `to_currency` does not affect the data fetched or the result text; it appears only in the logged call string. |
| C-49 | verified | `tools.py:202`, `tools.py:204-205`: `end=end_plus_one` | — |
| C-50 | verified | `tools.py:202-203` `fromisoformat` on both; called inside `try` at `tools.py:299-300`; `tools.py:312-313` error text | — |
| C-51 | verified | `tools.py:206-207` returns `[]`; `tools.py:301-305` "The market may have been closed..." | — |
| C-52 | verified | `tools.py:307-309`: `{avg:.2f} points`, `trading days`, min/max `.2f` | — |
| C-53 | verified | `tools.py:337`: `[a.strip().upper() for a in assets.split(",") if a.strip()]` | — |
| C-54 | verified | `tools.py:340-341`: `_tool_error(call, "no asset codes given.")` | — |
| C-55 | verified | `tools.py:337`, `tools.py:344`: plain list, loop over every entry | — |
| C-56 | verified | `tools.py:45-52`; `tools.py:345`: `ASSET_LABELS.get(asset, asset)` | — |
| C-57 | verified | `tools.py:349-351`: `${...:,.2f}/oz`, `Rp{...:,.0f}/g`, `Rp{...:,.0f}/0.01g` | — |
| C-58 | verified | `tools.py:35`, `tools.py:156`: `idr_per_oz / GRAMS_PER_TROY_OZ` | — |
| C-59 | verified | `tools.py:159-161`: pct from `cross_usd` | — |
| C-60 | verified | `tools.py:154`: `cross_idr.get(latest_date) or cross_idr[sorted(...)[-1]]` | — |
| C-61 | verified | `tools.py:356`: `f"{price:,.2f} pts"` | — |
| C-62 | verified | `tools.py:186-193`: intraday `period="1d", interval="1m"`, else `daily.iloc[-1]` from `period="5d"` | — |
| C-63 | verified | `tools.py:184`: `daily.iloc[-2]` if `len(daily) >= 2` else None; `tools.py:195` | — |
| C-64 | verified | `tools.py:189`, `tools.py:193`: `index[-1].strftime("%Y-%m-%d %H:%M")` | — |
| C-65 | verified | `tools.py:357-359`: `_get_price_and_change(asset, "IDR")`, `f"Rp{price:,.2f}"` | — |
| C-66 | verified | `tools.py:120-121`, `tools.py:144-146` `{"from": week_ago}`; `tools.py:130-132`, `tools.py:159-161` compare last two dates | — |
| C-67 | verified | `tools.py:73-79`: `f"{date_str} • checked {_now_hm()}"` with `datetime.now()` | — |
| C-68 | verified | `tools.py:376-380`: `n/a`, `🔺` if `pct >= 0` else `🔻`, `{pct:+.2f}%` | — |
| C-69 | verified | `tools.py:363-366` appends `(label, None, None, f"FAILED: {e}")`; `tools.py:373-374` `- | -` | — |
| C-70 | verified | `tools.py:385-387`: `f"could not fetch any of {asset_list}.\n{table}"` via `_tool_error` | — |
| C-71 | verified | `tools.py:389-390`: `return table` | — |
| C-72 | verified | `tools.py:412` | — |
| C-73 | incorrect | `tools.py:427`: `host = urlparse(url).netloc.lower()` | The site name is the URL's `netloc` (host plus any port or userinfo), lowercased, with a leading `www.` removed; an empty netloc (including a missing `href`) becomes `unknown site`. |
| C-74 | verified | `tools.py:34`, `tools.py:100`: `f"{API_BASE}/rates"` | — |
| C-75 | verified | `tools.py:100`: `timeout=15` | — |
| C-76 | verified | `tools.py:101-102` | — |
| C-77 | verified | `tools.py:105-106`: `row["date"]`, `row["quote"]`, `row["rate"]` | — |
| C-78 | verified | `tools.py:94-100`: params only `base`, `quotes`, date range; no headers | — |
| C-79 | verified | `tools.py:39`, `tools.py:179`, `tools.py:204` | — |
| C-80 | verified | `tools.py:180`, `tools.py:186`, `tools.py:205`: `timeout=YF_TIMEOUT` (10) | — |
| C-81 | verified | `tools.py:32` | — |
| C-82 | verified | `tools.py:412`: `DDGS()` with no arguments | — |
| C-83 | verified | `agent.py:34` | — |
| C-84 | verified | `agent.py:36-37` | — |
| C-85 | verified | `agent.py:33-38`: no temperature argument | — |
| C-86 | verified | Grep over `*.py`: only `app.py:10` imports streamlit | — |
| C-87 | verified | `agent.py:19`: module-level `load_dotenv()` | — |
| C-88 | verified | `agent.py:21`: `os.environ.get("GOOGLE_API_KEY", "").strip()` | — |
| C-89 | verified | `agent.py:22-24` | — |
| C-90 | verified | `.env.example:3` | — |
| C-91 | verified | `agent.py:43`; `prompts.py:47`: `DEFAULT_PROMPT_VERSION = "v2"` | — |
| C-92 | verified | `agent.py:44-45`: "Choose one of: {', '.join(PROMPTS)}" | — |
| C-93 | verified | `agent.py:43`, `agent.py:53`: module-level, built once | — |
| C-94 | verified | `agent.py:34`: literal `model="gemini-3.5-flash-lite"` | — |
| C-95 | verified | `prompts.py:30-40`: `SYSTEM_PROMPT_V1 + ("\n\n" ... two paragraphs)` | — |
| C-96 | verified | `prompts.py:8` module-level `TODAY`; `prompts.py:12` `f"Today is {TODAY}. ..."` | — |
| C-97 | verified | `prompts.py:13-15` | — |
| C-98 | verified | `prompts.py:16-19` | — |
| C-99 | verified | `prompts.py:20-21` | — |
| C-100 | verified | `prompts.py:22-23` | — |
| C-101 | verified | `prompts.py:32-36` | — |
| C-102 | verified | `prompts.py:37-39` | — |
| C-103 | verified | `.streamlit/config.toml:1`, `.streamlit/config.toml:5` | — |
| C-104 | verified | `.streamlit/config.toml:7`, `.streamlit/config.toml:9` | — |
| C-105 | verified | `trace_log.py:11`: `LOG_FILE = "agent_log.txt"` | — |
| C-106 | verified | `agent.py:163-175`: `except Exception`, logs `[AGENT ERROR] {kind}: {type(e).__name__}: {e}`, raises `AgentRunError(kind, ...)` | — |
| C-107 | verified | `agent.py:69-76`: lowercased type name + message, checks in stated order | — |
| C-108 | verified | `agent.py:177-180` | — |
| C-109 | verified | `agent.py:138` and `app.py:99-100` assign history only when `run_turn` returns | — |
| C-110 | verified | `agent.py:26`, `agent.py:186-189` | — |
| C-111 | verified | `cli.py:21-26` | — |
| C-112 | verified | `cli.py:64`, `cli.py:67` | — |
| C-113 | verified | `cli.py:65-66`: `if e.kind == "other": print(f"   Reason: {_summarize(str(e))}")` | — |
| C-114 | verified | `app.py:105-108`: no reason line | — |
| C-115 | verified | `cli.py:69-71` | — |
| C-116 | verified | `cli.py:72-76`: logs `[CLI ERROR]`, prints `Unexpected error: {_summarize(str(e))}` | — |
| C-117 | verified | `app.py:109-114`: logs `[UI ERROR]`, `turn["error"] = f"Unexpected error: {e}"` | — |
| C-118 | verified | `app.py:115` appends turn in all cases; `app.py:79-80` `st.error(turn["error"])`; `app.py:137-138` | — |
| C-119 | verified | `app.py:14-18`: `except SystemExit as e: st.error(str(e)); st.stop()` | — |
| C-120 | verified | Grep: `SystemExit` caught only at `app.py:16`; `cli.py:7` and `currency_agent.py:23` are bare imports | — |
| C-121 | verified | `tools.py:389-390` table without prefix; `agent.py:124` ok check; `cli.py:47` ✅ mark; `app.py:58` reuses it | — |
| C-122 | verified | `tools.py:133-134`, `tools.py:162-163` `pct_change = None`; `tools.py:376-377` `n/a` | — |
| C-123 | verified | `prompts.py:8` fixed at import; `agent.py:53` agent built once | — |
| C-124 | verified | `trace_log.py:15` mode "w"; calls at `cli.py:127`, `cli.py:151`→`agent.py:110`, `app.py:44-45`→`app.py:37`, `app.py:129`→`app.py:37` | — |
| C-125 | incorrect | `trace_log.py:11`: `LOG_FILE = "agent_log.txt"` is a relative path, resolved against each process's working directory | Within one Streamlit server process, all web sessions share the same log file, so a new session or Reset empties it for the others; the terminal process shares that file only when it runs from the same working directory. |
| C-126 | verified | `cli.py:98-102` returns None on `EOFError`/`KeyboardInterrupt`; `cli.py:140-142` prints "[ESC pressed] Closing the program. Goodbye!" | — |
| C-127 | verified | `cli.py:107-120` | — |
| C-128 | verified | `app.py:54` `_md`; applied at `app.py:73`, `82`, `88`, `103`; errors raw at `app.py:80`, `108`, `114` | — |
| C-129 | verified | `cli.py:32-40`, `cli.py:48`; `app.py:58` reuses `format_step` | — |
| C-130 | verified | `app.py:141`: `typed or ...pop(...)` short-circuits, pending not popped when typed | — |
| C-131 | verified | `cli.py:83-84`; Grep `ask\(` in `*.py` finds only the definition `cli.py:81` | — |
| C-132 | verified | `tools.py:179`: `yf.Ticker("^JKSE")` | — |
| C-133 | verified | `tools.py:337` strip/upper; `tools.py:354`: `elif asset == "IHSG":` | — |
| C-134 | verified | `app.py:131`: `f"Prompt {agent.PROMPT_VERSION} · {agent.MODEL_NAME}"` | — |
| C-135 | verified | Grep: only `requests.get` at `tools.py:100`, `yf.Ticker` at `tools.py:179`/`204`, `DDGS` at `tools.py:412` | — |
| C-136 | verified | `agent.py:98`, `app.py:35` in-memory; Grep `open(` only at `trace_log.py:15`, `trace_log.py:28` | — |
| C-137 | verified | `app.py:10-143`: no login or auth code | — |
| C-138 | verified | `tools.py:100` GET; `tools.py:180`, `186`, `204-205` history reads; `tools.py:412` search | — |

**Summary counts:** total: 138, verified: 133, incorrect: 4, unresolved: 1

**Omissions (max 5):**
- `extract_text` keeps only string blocks and `type == "text"` dict blocks when the model content is a list and joins them; anything else is dropped. `agent.py:82-95`
- The web page labels each turn's step box from tool counts: "No tool needed", "🔧 N tool call(s)", and "· ❌ M failed". `app.py:61-67`
- `get_price_table` logs a `✅ [ASSET] price (as of ...)` line for each asset and a summary line `table with N assets (M failed)`. `tools.py:361`, `tools.py:389`
- In the terminal, blank input (after trimming) is skipped without sending a request. `cli.py:145-146`
- Each log session starts with a `=== NEW SESSION STARTED: <timestamp> ===` header, and each request block is logged with a separator, a timestamp, and the question. `trace_log.py:16-17`, `trace_log.py:21-24`

**Reviewer notes:**
- No claims are struck through, and Q-01 to Q-13 were not given verdicts.
- C-04: I read "the same agent object" as "the same module-level agent definition". The terminal and the web page run as separate processes, and each process reads `PROMPT_VERSION` from its own environment. So the prompt version matches only if both environments match.
- C-48 and C-73: I marked these incorrect only because the wording is broader than the code (`to_currency` still appears in the logged call text, and the code uses netloc rather than the bare host). The data behavior they describe is otherwise correct.
- C-34: everything except "up to 4 lines" is verified. I marked it unresolved because the code itself never caps the number of results.
- Grep used the glob `*.py` under `input/demo-a/`. Every hit was in an in-scope file, and I opened no out-of-scope files.
- Absolute paths of the files reviewed: e:\Project\Spec_ReverseEng\outputs\run-03\spec_draft.md; code in e:\Project\Spec_ReverseEng\input\demo-a\
