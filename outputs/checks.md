| Claim | Verdict | Evidence | Required correction |
|---|---|---|---|
| C-01 | verified | `cli.py:130`; `app.py:12`; `app.py:122`; `app.py:135`: "Asset Price Assistant" in the banner, page_title, sidebar header and st.title | — |
| C-02 | verified | `agent.py:33-38`; `agent.py:50`; `tools.py:431`: ChatGoogleGenerativeAI passed to create_agent with ALL_TOOLS (4 tools) | — |
| C-03 | verified | `currency_agent.py:23-26`: `from cli import chat` ... `chat()`; `app.py:10` `import streamlit as st` | — |
| C-04 | verified | `agent.py:53` `agent = build_agent(PROMPT_VERSION)`, used by run_turn (`agent.py:165`); cli calls `agent.run_request` (`cli.py:62`), app calls `agent.run_turn` (`app.py:99-100`) | — |
| C-05 | verified | `tools.py:320`: `def get_price_table(assets: str = "USD,SGD,TWD,XAU,IHSG")` | — |
| C-06 | verified | `tools.py:214-243`: get_exchange_rate returns the latest rate for one pair | — |
| C-07 | verified | `tools.py:276-290`: Frankfurter range from/to, mean of values | — |
| C-08 | verified | `tools.py:272-274`; `tools.py:297-311`: IHSG branch goes to _average_index, which averages closes | — |
| C-09 | verified | `tools.py:396-420`: `DDGS().text(query, max_results=4)` | — |
| C-10 | verified | `agent.py:160` `list(history) + [("human", question)]`; `agent.py:183` full state messages stored; `agent.py:138`; `app.py:99-100` | — |
| C-11 | verified | `agent.py:165-170` calls on_step for each new message's steps during streaming; `cli.py:62` print_step; `app.py:93-95` | — |
| C-12 | verified | `cli.py:144` strip; `cli.py:150-152` `question.lower() in ("reset", "clear")` → reset_conversation + notice; `agent.py:105-110` | — |
| C-13 | verified | `app.py:129` on_click=_start_conversation; `app.py:35-36` resets history and turns | — |
| C-14 | verified | `app.py:23-28` four EXAMPLES; `app.py:126-127` buttons; `app.py:40-41` sets pending; `app.py:141` `typed or st.session_state.pop("pending", None)` | — |
| C-15 | verified | `cli.py:108-110` ESC returns None; `cli.py:140-142`; `cli.py:147-149` `("exit", "quit", "bye", "esc")` after `.lower()` | — |
| C-16 | verified | `trace_log.py:11` LOG_FILE; `agent.py:158` log_request; `tools.py:231` / `tools.py:242` / `tools.py:66` call and result lines; `agent.py:190-191` final answer | — |
| C-17 | verified | `cli.py:131` examples line; `cli.py:134` prints `trace_log.LOG_FILE` | — |
| C-18 | verified | `currency_agent.py:21` `sys.stdout.reconfigure(encoding="utf-8", errors="replace")` at module level, before `currency_agent.py:23` import | — |
| C-19 | verified | `currency_agent.py:25-26`: `if __name__ == "__main__": chat()` | — |
| C-20 | verified | `cli.py:21-26`, `cli.py:43-78`, `cli.py:92-155`: messages, step printing, _answer, input reading, chat loop | — |
| C-21 | verified | `agent.py:19-24` load_dotenv + key check; `agent.py:33-53` model and agent; `agent.py:98` history; `agent.py:129-192` turns; `agent.py:67-76` classify | — |
| C-22 | verified | `tools.py:85-208` helpers; `tools.py:214-428` tools; `tools.py:431` ALL_TOOLS | — |
| C-23 | verified | `prompts.py:43-46`: `PROMPTS = {"v1": ..., "v2": ...}` | — |
| C-24 | verified | `trace_log.py:15` mode "w"; `trace_log.py:21-24` log_request uses log; `trace_log.py:28` mode "a" | — |
| C-25 | verified | `app.py:10-143`: module-level script, no `__main__` guard (Grep finds none) | — |
| C-26 | verified | `app.py:58` backtick wrap with `.replace("`", "'")`; `app.py:105-106` different note and no Reason; `app.py:112` `f"Unexpected error: {e}"` vs `cli.py:75` `_summarize(str(e))`; `cli.py:65-67` | — |
| C-27 | verified | `.streamlit/config.toml:1-9`: [server] and [browser] sections | — |
| C-28 | verified | `requirements.txt:2-11`: 10 `==` pins incl. langchain==1.3.14, langchain-google-genai==4.3.2, yfinance==1.5.2, ddgs==9.14.4, streamlit==1.65.0 | — |
| C-29 | verified | `requirements-dev.txt:2-4`: `-r requirements.txt`, pytest==9.2.0, PyYAML==6.0.3 | — |
| C-30 | verified | `pytest.ini:2-3`: `testpaths = tests`, `pythonpath = .` | — |
| C-31 | verified | `tools.py:233` no date_or_range; `tools.py:241` output format; `tools.py:234-238` empty → web_search hint; `tools.py:244-245` exception text | — |
| C-32 | verified | `tools.py:276-279`; `tools.py:204-205`; `tools.py:281-284`; `tools.py:293-294`; `tools.py:301-305`; `tools.py:312-313` `f"could not fetch {index_code} history: {e}"` | — |
| C-33 | verified | `tools.py:347-360` sources; `tools.py:368-370` headers; `tools.py:366` FAILED row; `tools.py:385-387` all failed → ERROR + table | — |
| C-34 | verified | `tools.py:412-418`: one `- [site] title: body` per result, joined by "\n", no slicing; `tools.py:413-414`; `tools.py:421-422` | — |
| C-35 | verified | `tools.py:231`, `tools.py:271`, `tools.py:339`, `tools.py:410`: `log(f"🔧 [TOOL CALLED] {call}")` before any fetch | — |
| C-36 | verified | `tools.py:64-67`: logs `❌ [TOOL FAILED] {call} -> {reason}` and returns `f"{TOOL_ERROR_PREFIX} {reason}"` | — |
| C-37 | verified | `agent.py:124`: `ok = message.status != "error" and not content.startswith(TOOL_ERROR_PREFIX)` | — |
| C-38 | verified | `tools.py:431` order matches; `agent.py:50` `tools=ALL_TOOLS` | — |
| C-39 | verified | `tools.py:91-96`: upper(), `sorted(set([from_c, to_c]) - {"EUR"})`, quotes added only if non-empty | — |
| C-40 | verified | `tools.py:110-113`: `cross[date] = rate_to / rate_from`, 1.0 for EUR | — |
| C-41 | verified | `tools.py:112`: `if rate_from and rate_to:` (None or 0 skips the date) | — |
| C-42 | verified | `tools.py:91`: only `.upper()`, no strip | — |
| C-43 | verified | `tools.py:239-240`: `latest_date = max(cross.keys())` | — |
| C-44 | verified | `tools.py:241`: `{rate:.6g}` | — |
| C-45 | verified | `tools.py:39` `INDEX_TICKERS = {"IHSG": "^JKSE"}`; `tools.py:272-273` | — |
| C-46 | verified | `tools.py:286-287`: `sum(values) / len(values)` | — |
| C-47 | verified | `tools.py:276-279`: dates passed straight into params with no validation | — |
| C-48 | verified | `tools.py:268-271` call includes to_currency; `tools.py:297-313` _average_index never uses to_currency except via `call` passed to _tool_error (log only) | — |
| C-49 | verified | `tools.py:202-205`: end_plus_one = end_date + 1 day passed as `end` | — |
| C-50 | verified | `tools.py:202-203` fromisoformat on both; inside the try at `tools.py:299-300`; `tools.py:312-313` | — |
| C-51 | verified | `tools.py:206-207` returns []; `tools.py:301-305` "The market may have been closed..." | — |
| C-52 | verified | `tools.py:307-309`: `{avg:.2f} points`, `trading days`, min/max `.2f` | — |
| C-53 | verified | `tools.py:337`: `[a.strip().upper() for a in assets.split(",") if a.strip()]` | — |
| C-54 | verified | `tools.py:340-341`: `_tool_error(call, "no asset codes given.")` | — |
| C-55 | verified | `tools.py:337`, `tools.py:344`: list is iterated with no dedup | — |
| C-56 | verified | `tools.py:45-52`; `tools.py:345` `ASSET_LABELS.get(asset, asset)` | — |
| C-57 | verified | `tools.py:349-351`: `${...:,.2f}/oz`, `Rp{...:,.0f}/g`, `Rp{...:,.0f}/0.01g` | — |
| C-58 | verified | `tools.py:35`; `tools.py:156`: `idr_per_oz / GRAMS_PER_TROY_OZ` | — |
| C-59 | verified | `tools.py:159-161`: pct from cross_usd | — |
| C-60 | verified | `tools.py:154`: `cross_idr.get(latest_date) or cross_idr[sorted(cross_idr.keys())[-1]]` | — |
| C-61 | verified | `tools.py:356`: `f"{price:,.2f} pts"` | — |
| C-62 | verified | `tools.py:186-193`: intraday 1d/1m last close, else daily last close; `tools.py:180` period="5d" | — |
| C-63 | verified | `tools.py:184` `daily.iloc[-2]` if len ≥ 2 else None; `tools.py:195` | — |
| C-64 | verified | `tools.py:189`; `tools.py:193`: `.strftime("%Y-%m-%d %H:%M")` on the index of the row used | — |
| C-65 | verified | `tools.py:357-359`: else branch `_get_price_and_change(asset, "IDR")`, `f"Rp{price:,.2f}"` | — |
| C-66 | verified | `tools.py:120-121`; `tools.py:144-146` from week_ago; `tools.py:126-132`; `tools.py:159-161` previous sorted date | — |
| C-67 | verified | `tools.py:73-79`: `f"{date_str} • checked {_now_hm()}"` using `datetime.datetime.now()` | — |
| C-68 | verified | `tools.py:376-380`: n/a; `"🔺" if pct >= 0 else "🔻"`; `{pct:+.2f}%` | — |
| C-69 | verified | `tools.py:363-366` except per asset; `tools.py:373-374` `| {label} | - | - | {last_update} |` | — |
| C-70 | verified | `tools.py:385-387`: `f"could not fetch any of {asset_list}.\n{table}"` via _tool_error | — |
| C-71 | verified | `tools.py:389-390`: `return table` | — |
| C-72 | verified | `tools.py:412` | — |
| C-73 | verified | `tools.py:416` `r.get("href", "")`; `tools.py:427-428` `urlparse(url).netloc.lower()`, `removeprefix("www.") or "unknown site"` | — |
| C-74 | verified | `tools.py:34`; `tools.py:100` `f"{API_BASE}/rates"` | — |
| C-75 | verified | `tools.py:100`: `timeout=15` | — |
| C-76 | verified | `tools.py:101-102`: `resp.raise_for_status()` before `resp.json()` | — |
| C-77 | verified | `tools.py:105-106`: `row["date"]`, `row["quote"]`, `row["rate"]` | — |
| C-78 | verified | `tools.py:94-100`: params only base/quotes/date range; no headers argument | — |
| C-79 | verified | `tools.py:39`; `tools.py:179`; `tools.py:204` | — |
| C-80 | verified | `tools.py:180`, `tools.py:186`, `tools.py:205`: `timeout=YF_TIMEOUT` (10) | — |
| C-81 | verified | `tools.py:32`: `logging.getLogger("yfinance").setLevel(logging.CRITICAL)` | — |
| C-82 | verified | `tools.py:25`; `tools.py:412`: `DDGS()` with no key | — |
| C-83 | verified | `agent.py:34`: `model="gemini-3.5-flash-lite"` | — |
| C-84 | verified | `agent.py:36-37`: `timeout=30`, `max_retries=2` | — |
| C-85 | verified | `agent.py:33-38`: no temperature kwarg | — |
| C-86 | verified | `app.py:10`: Grep over *.py finds `import streamlit` only in app.py | — |
| C-87 | verified | `agent.py:19`: module-level `load_dotenv()` | — |
| C-88 | verified | `agent.py:21`: `os.environ.get("GOOGLE_API_KEY", "").strip()` | — |
| C-89 | verified | `agent.py:22-24`: exact message text | — |
| C-90 | verified | `.env.example:3`: `GOOGLE_API_KEY=your-key-here` | — |
| C-91 | verified | `agent.py:43`; `prompts.py:47` `DEFAULT_PROMPT_VERSION = "v2"` | — |
| C-92 | verified | `agent.py:44-45`: SystemExit "Choose one of: {', '.join(PROMPTS)}" | — |
| C-93 | verified | `agent.py:43`; `agent.py:53`: module-level, executed on import | — |
| C-94 | verified | `agent.py:34`: hard-coded string | — |
| C-95 | verified | `prompts.py:30-40`: `SYSTEM_PROMPT_V1 + ("\n\n" ... "\n\n" ...)` two paragraphs | — |
| C-96 | verified | `prompts.py:8` TODAY at import; `prompts.py:12` `f"Today is {TODAY}. ..."`; v2 builds on v1 | — |
| C-97 | verified | `prompts.py:13-15`: "ALWAYS use the available tools to get numbers. Never guess or use data from your memory." | — |
| C-98 | verified | `prompts.py:16-19` | — |
| C-99 | verified | `prompts.py:20-21` | — |
| C-100 | verified | `prompts.py:22-23` | — |
| C-101 | verified | `prompts.py:32-36` | — |
| C-102 | verified | `prompts.py:37-39` | — |
| C-103 | verified | `.streamlit/config.toml:1`, `.streamlit/config.toml:5` | — |
| C-104 | verified | `.streamlit/config.toml:7`, `.streamlit/config.toml:9` | — |
| C-105 | verified | `trace_log.py:11`: `LOG_FILE = "agent_log.txt"` | — |
| C-106 | verified | `agent.py:172-175`: `except Exception`, log `[AGENT ERROR] {kind}: {type(e).__name__}: {e}`, raise AgentRunError(kind, ...) | — |
| C-107 | verified | `agent.py:69-76`: keyword lists checked in order quota, timeout, auth, else other | — |
| C-108 | verified | `agent.py:177-180` | — |
| C-109 | verified | `agent.py:138` assignment only on return; `agent.py:182-183`; `app.py:99-100` tuple assignment skipped when run_turn raises | — |
| C-110 | verified | `agent.py:26`; `agent.py:186-189` | — |
| C-111 | verified | `cli.py:21-26`; `cli.py:22` quota text matches | — |
| C-112 | verified | `cli.py:63-67`: `⚠️  {ERROR_MESSAGES...}` and "(This question was not saved to the conversation. Details are in the log.)" | — |
| C-113 | verified | `cli.py:65-66`: `if e.kind == "other": print(f"   Reason: {_summarize(str(e))}")` | — |
| C-114 | verified | `app.py:104-108`: message + " This question was not saved to the conversation.", `status.update(label="Failed", ...)`, no reason | — |
| C-115 | verified | `cli.py:69-71` | — |
| C-116 | verified | `cli.py:72-76`: `[CLI ERROR]` log, `Unexpected error: {_summarize(str(e))}` | — |
| C-117 | verified | `app.py:109-114`: `[UI ERROR]` log, `f"Unexpected error: {e}"`, st.error | — |
| C-118 | verified | `app.py:115` append after try/except; `app.py:79-80` st.error for error turns; `app.py:137-138` | — |
| C-119 | verified | `app.py:14-18`: `except SystemExit as e: st.error(str(e)); st.stop()` | — |
| C-120 | verified | `cli.py:7` bare `import agent`; `currency_agent.py:23` bare import; no try around either | — |
| C-121 | verified | `tools.py:389-390` table returned without prefix; `agent.py:124`; `cli.py:47` ✅ mark; `app.py:58` reuses format_step | — |
| C-122 | verified | `tools.py:130-134` / `tools.py:159-163` pct None if < 2 dates; `tools.py:376-377` "n/a" | — |
| C-123 | verified | `prompts.py:8` TODAY computed once; `agent.py:53` agent built once with that prompt | — |
| C-124 | verified | `trace_log.py:15` mode "w"; call sites `cli.py:127`, `cli.py:151`→`agent.py:110`, `app.py:44-45`→`app.py:37`, `app.py:129` | — |
| C-125 | verified | `trace_log.py:11` relative path; `trace_log.py:15` "w"; `trace_log.py:28` "a"; claim states its process-sharing part as inferred/conditional | — |
| C-126 | verified | `cli.py:98-102` returns None on EOFError/KeyboardInterrupt; `cli.py:140-142` prints "[ESC pressed] Closing the program. Goodbye!" | — |
| C-127 | verified | `cli.py:104-120`: getwch loop; `\x1b` quits, `\r`/`\n` ends, `\x08` pops, else append | — |
| C-128 | verified | `app.py:54` escapes `$` and replaces `<br>`; applied at `app.py:73`, `app.py:82`, `app.py:88`, `app.py:103`; not at `app.py:80`, `app.py:108`, `app.py:114` | — |
| C-129 | verified | `cli.py:32-40` first line, `[:100] + "..."`, `(+N lines)`; `cli.py:48`; `app.py:58` | — |
| C-130 | verified | `app.py:141`: `typed or st.session_state.pop("pending", None)` | — |
| C-131 | verified | `cli.py:83-84`: start_session only if `not agent.has_history()`; Grep for `ask(` in *.py finds only the definition at `cli.py:81` | — |
| C-132 | verified | `tools.py:179`: `yf.Ticker("^JKSE")` | — |
| C-133 | verified | `tools.py:337` strip/upper; `tools.py:354` `elif asset == "IHSG":` | — |
| C-134 | verified | `app.py:131`: `st.caption(f"Prompt {agent.PROMPT_VERSION} · {agent.MODEL_NAME}...")` | — |
| C-135 | verified | Only network calls: `tools.py:100` (Frankfurter), `tools.py:179`/`tools.py:204` (yfinance), `tools.py:412` (DDGS) | — |
| C-136 | verified | `agent.py:98` module list; `app.py:35` session_state; Grep for `open(`/`.write(` finds only `trace_log.py:15-16` and `trace_log.py:28-29` | — |
| C-137 | verified | `app.py:10-143`: no login or auth code | — |
| C-138 | verified | `tools.py:100` requests.get; `tools.py:180`, `tools.py:186`, `tools.py:204-205` history reads; `tools.py:412` text search; no write calls | — |
| C-139 | verified | `agent.py:84-95`: str returned as-is; list keeps str blocks and dicts with `type == "text"`, `"".join(parts).strip()`; else `str(content)` | — |
| C-140 | incorrect | `app.py:102` the label is applied to the live status box only on success; `app.py:107` and `app.py:113` a failed turn's live box is labelled "Failed"; `app.py:75-76` when earlier turns are redrawn, the expander (with this label) appears only if `turn["steps"]` is non-empty | The `_steps_label` text ("No tool needed", or "🔧 N tool call(s)" plus " · ❌ M failed" when M > 0) is used for the live status box of a successful turn, and for the expander of a redrawn earlier turn that has steps. A failed turn's live box is labelled "Failed". A redrawn turn with no steps shows no step box at all. |
| C-141 | verified | `cli.py:144-146`: `question.strip()`; `if not question: continue` | — |
| C-142 | verified | `tools.py:361`: `log(f"✅ [{asset}] {price_str} (as of {last_update})")` | — |
| C-143 | verified | `tools.py:389`: `log(f"✅ [TOOL OK] table with {len(rows)} assets ({failed} failed)")` before `return table` | — |
| C-144 | verified | `trace_log.py:16-17`: `=== NEW SESSION STARTED: %Y-%m-%d %H:%M:%S ===` with `datetime.now()` | — |
| C-145 | verified | `trace_log.py:21-24`: `"\n" + "="*60`, REQUEST timestamp, Question, `"-"*60`; called at `agent.py:158` | — |

**Summary counts:** total: 145, verified: 144, incorrect: 1, unresolved: 0

**Omissions (max 5):**
- Web page: if the user types a question while an example question is pending, `typed or ...pop(...)` short-circuits, so the pending example is not removed and runs on a later rerun when nothing is typed (`app.py:141`)
- An unknown `AgentRunError.kind` falls back to the `other` message in both interfaces through `.get(e.kind, ERROR_MESSAGES['other'])` (`cli.py:64`; `app.py:105`)
- `reset_conversation(new_log=False)` clears the history without restarting the log file (`agent.py:105-110`)
- Web sidebar shows a legend ("🔧 tool chosen · ✅ result OK · ❌ tool failed") and a caption naming the data sources plus "Not financial advice." (`app.py:123-124`; `app.py:130-133`)
- Web live status box: collapses (`expanded=False`) on success but stays expanded on failure (`app.py:102`; `app.py:107`; `app.py:113`)

**Reviewer notes:**
- No claims are struck through. C-139 to C-145 appear inside section 8, before C-135 to C-138 in section 9. The table lists every row in numeric ID order.
- C-04: "same agent object" is read as both interfaces using the module-level `agent` built in `agent.py`. The terminal and the web page run as separate processes, so `PROMPT_VERSION` could differ between them if their environments differ.
- C-121: the ✅ result also depends on LangChain giving `ToolMessage.status` a value other than `"error"` for a tool that returns normally. That is library behavior and not visible in in-scope code (compare the spec's Q-07). It is marked verified on the in-scope logic only.
- C-131 and C-136 rely on Grep over `*.py` in `input/demo-a/`. All hits were in in-scope files.
