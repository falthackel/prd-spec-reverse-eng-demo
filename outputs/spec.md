# Spec — Agentic Financial Monitoring Assistant
- Input repo: input/demo-a/ (Demo A)
- Commit: 6caaf96 (tag demo-v1)
- Skill version: v2
- Run ID: run-02
- Date: 2026-11-13
- Stage: revised

All paths are relative to `input/demo-a/`.

---

## 1. Purpose and target user

- **C-01** The product is a question-answering agent built with LangChain `create_agent` around a Gemini chat model and four tools (`agent.py:12-13`, `agent.py:33-38`, `agent.py:48-50`, `tools.py:431`).
- **C-02** Both the terminal and the web page show the product name "Asset Price Assistant" (terminal: `cli.py:130`; web: `app.py:12`, `app.py:122`, `app.py:135`).
- **C-03** Both system prompt versions tell the agent it helps answer questions about currency exchange rates and asset prices (`prompts.py:12-13`, `prompts.py:30`).
- **C-04** The web page sidebar describes the product as covering "Exchange rates, gold, and the IHSG index in rupiah" (web only, `app.py:123-124`).
- **C-05** The web page sidebar shows the text "Not financial advice." (web only, `app.py:131-133`).

(Target user: not settled by code, see Q-01.)

## 2. User stories / features

- **C-06** A user can type a free-text question and get one answer per turn, in the terminal (`cli.py:137-155`) and on the web page (`app.py:140-143`).
- **C-07** The conversation history from earlier turns is sent to the agent with each new question (both; `agent.py:160`).
- **C-08** The history kept for the next turn includes the tool-call and tool-result messages, not only questions and answers (both; `agent.py:183`).
- **C-09** In the terminal, typing `reset` or `clear` (case-insensitive, after trimming spaces) empties the conversation history and prints "[Conversation reset: earlier context is forgotten]" (`cli.py:144`, `cli.py:150-153`, `agent.py:105-108`).
- **C-10** On the web page, the "🔄 Reset conversation" sidebar button empties that tab's history and its displayed turns (`app.py:129`, `app.py:34-36`).
- **C-11** On the web page, the conversation is stored in `st.session_state.history` and passed to `agent.run_turn`, not in the module-level history used by the terminal (whether `st.session_state` is per tab is Streamlit runtime behavior, see Q-10) (`app.py:44-45`, `app.py:99-100`).
- **C-12** The terminal keeps one conversation in the module-level list `agent._conversation_history`, used through `agent.run_request` (`agent.py:98`, `agent.py:129-139`, `cli.py:62`).
- **C-13** The terminal prints each agent step live while a question is answered: a tool call as `🔧 name(arg=value, …)` and a tool result as `✅`/`❌ name → summary` (`cli.py:43-53`, `cli.py:62`).
- **C-14** The web page shows each step live inside a "Working..." status box, then, when the answer arrives, collapses the box and relabels it via `_steps_label` (tool-call count plus failed count, or "No tool needed") (`app.py:91-95`, `app.py:102`).
- **C-15** The web page sidebar offers four example questions as buttons; clicking one runs it as the next question (`app.py:23-28`, `app.py:40-41`, `app.py:126-127`, `app.py:141-143`).
- **C-16** On the web page, earlier turns are redrawn on each page run; a turn that had steps shows them folded inside an expander (`app.py:70-82`, `app.py:75-78`, `app.py:137-138`).
- **C-17** The web page sidebar shows the active prompt version and the model name (`app.py:131`).
- **C-18** The system prompt tells the agent to use `get_price_table` for a "price update", "market summary", or several assets at once (both prompt versions, `prompts.py:16-17`).
- **C-19** The system prompt tells the agent to use `get_exchange_rate` only for one currency pair and `get_average_exchange_rate` for averages or comparing periods (both prompt versions, `prompts.py:17-19`).
- **C-20** The system prompt tells the agent to try `web_search` if a structured data tool fails (both prompt versions, `prompts.py:20-21`).
- **C-21** The system prompt tells the agent to answer without a tool when the question is unrelated to exchange rates or asset prices (both prompt versions, `prompts.py:22-23`).
- **C-22** Prompt v2 additionally tells the agent to use `web_search` for "why did it move" or news questions and to name the source of each reason (v2 only, `prompts.py:30-36`).
- **C-23** Prompt v2 additionally tells the agent to get a single past date's rate by calling `get_average_exchange_rate` with the same start and end date (v2 only, `prompts.py:37-39`).
- **C-24** Both system prompts include today's date as an ISO string (`prompts.py:8`, `prompts.py:12`).
- **C-124** When the terminal chat starts, it prints a banner with the product name, two example questions, how to quit (ESC or `exit`), how to reset (`reset`), and the log file path (terminal only, `cli.py:129-135`).

## 3. Components (files and their responsibility)

- **C-25** `currency_agent.py` is the terminal entry point: run as a script, it calls `cli.chat()` (`currency_agent.py:23-26`).
- **C-26** `currency_agent.py` reconfigures stdout to UTF-8 with `errors="replace"` before importing `cli` (`currency_agent.py:21-23`).
- **C-27** `cli.py` holds the terminal loop, input reading, step formatting, and user-facing error messages (`cli.py:18-155`).
- **C-28** `agent.py` builds the Gemini model and the agent, holds the terminal conversation history, runs a turn, and classifies errors (`agent.py:33-53`, `agent.py:59-76`, `agent.py:98-192`).
- **C-29** `tools.py` defines the four tools, data helpers that call Frankfurter and yfinance, log and time formatting helpers, and a `_site_name` URL formatter; DuckDuckGo is called directly inside `web_search`, with no separate data helper (`tools.py:58-79`, `tools.py:85-208`, `tools.py:214-431`, `tools.py:412`, `tools.py:425-428`).
- **C-30** `prompts.py` holds two system prompt versions, `v1` and `v2`, in the `PROMPTS` dict (`prompts.py:11-46`).
- **C-31** `trace_log.py` writes the text log file `agent_log.txt` (`trace_log.py:11-29`).
- **C-32** `app.py` is a Streamlit web page that imports the same `agent` module and reuses `cli.format_step` and `cli.ERROR_MESSAGES` (`app.py:10`, `app.py:15`, `app.py:20`, `app.py:58`, `app.py:105`).
- **C-33** `cli.ask()` is defined for one-off questions, but no in-scope file calls it (`cli.py:81-86`; search for `ask(` finds only the definition).
- **C-117** At the start of every `run_turn` call, before the model is called, the question is appended to `agent_log.txt` with a `REQUEST: YYYY-MM-DD HH:MM:SS` timestamp, so turns that later fail are logged too (both surfaces; `agent.py:158`, `trace_log.py:20-24`).
- **C-118** When `run_turn` finishes with a final answer, it appends `=== FINAL ANSWER ===` and the answer text to the log; a turn that raises `AgentRunError` does not reach this step (both surfaces; `agent.py:172-180`, `agent.py:190-191`).
- **C-119** Each of the four tools appends `🔧 [TOOL CALLED] name(args)` to the log when it is called (`tools.py:231`, `tools.py:271`, `tools.py:339`, `tools.py:410`).
- **C-120** Each tool appends a `✅ [TOOL OK] …` line to the log on its non-error return paths, including a partly failed price table (`tools.py:242`, `tools.py:291`, `tools.py:310`, `tools.py:389`, `tools.py:419`).
- **C-125** No in-scope code deletes `agent_log.txt` or reads it back; its contents stay on disk after the process ends until the next `start_session()` call overwrites it (`trace_log.py:14-15`, `trace_log.py:27-29`).

## 4. Tools

| Claim | Name | Inputs | Output | Data source | Failure behavior |
|---|---|---|---|---|---|
| **C-34** | `get_exchange_rate` | `from_currency`, `to_currency` (strings) | `"1 FROM = rate TO (as of DATE)"`, rate to 6 significant digits, using the latest date in the response (`tools.py:239-241`) | Frankfurter `/v2/rates` with `base=EUR`, no date parameter (`tools.py:233`, `tools.py:94-100`) | Empty result → `ERROR: no rate found … try web_search.`; exception inside its `try` → `ERROR: could not fetch exchange rate: …` (`tools.py:234-238`, `tools.py:244-245`) |
| **C-35** | `get_average_exchange_rate` (currency path) | `from_currency`, `to_currency`, `start_date`, `end_date` | `"Average 1 FROM = avg TO from START to END (N days of data, min …, max …)"`, 6 significant digits (`tools.py:286-290`) | Frankfurter `/v2/rates` with `from`/`to` range (`tools.py:276-279`) | Empty result → `ERROR: no FROM to TO rate data between …`; exception inside its `try` → `ERROR: could not fetch historical rates: …` (`tools.py:280-284`, `tools.py:293-294`) |
| **C-36** | `get_average_exchange_rate` (index path, `from_currency` = `IHSG` after strip/upper) | same; `to_currency` is not used (`tools.py:272-274`, `tools.py:297-313`) | `"Average IHSG close = avg points from START to END (N trading days, min …, max …)"`, 2 decimals (`tools.py:306-309`) | yfinance `^JKSE` daily history, end date + 1 day (`tools.py:199-208`, `tools.py:39`) | No rows → `ERROR: no IHSG data … market may have been closed …`; exception → `ERROR: could not fetch IHSG history: …` (`tools.py:301-305`, `tools.py:312-313`) |
| **C-37** | `get_price_table` | `assets`: comma-separated codes, default `"USD,SGD,TWD,XAU,IHSG"` (`tools.py:320`) | Markdown table `Asset | Price | Change (vs previous day) | Last Update`, prices in IDR except IHSG in points and gold also in USD (`tools.py:343-390`) | Frankfurter for currencies and XAU; yfinance `^JKSE` for IHSG (`tools.py:347-360`) | A failing asset becomes a row with `-` and `FAILED: …`; only when every asset fails is the result prefixed `ERROR:`; empty list → `ERROR: no asset codes given.` (`tools.py:340-341`, `tools.py:363-366`, `tools.py:384-387`) |
| **C-38** | `web_search` | `query` (string) | One `- [site] title: body` entry per result the library returns, joined with newlines; the code requests `max_results=4` but does not cut the returned list itself (`tools.py:412`, `tools.py:416-418`; library behavior: Q-15) | DuckDuckGo via `ddgs.DDGS().text` (`tools.py:25`, `tools.py:412`) | No results → `ERROR: no search results found.`; exception inside its `try` → `ERROR: search failed: …` (`tools.py:413-414`, `tools.py:421-422`) |

- **C-39** The agent is created with exactly these four tools: `get_exchange_rate`, `get_average_exchange_rate`, `get_price_table`, `web_search` (`tools.py:431`, `agent.py:50`).
- **C-40** Currency cross rates are computed by dividing two EUR-based rates: `rate_to / rate_from`, with EUR itself counted as 1.0 (`tools.py:108-113`).
- **C-41** A date is left out of the cross-rate series when either currency's rate is missing or zero for that date (`tools.py:112`).
- **C-42** Currency codes are upper-cased before the Frankfurter request (`tools.py:91`).
- **C-43** `get_price_table` upper-cases and trims each asset code and drops empty entries (`tools.py:337`).
- **C-44** In `get_price_table`, a currency's price is the latest IDR cross rate within the last 7 days, shown as `Rp` with 2 decimals (`tools.py:118-128`, `tools.py:358-359`).
- **C-45** In `get_price_table`, a currency's % change compares the latest date with the second-latest date in that 7-day series, which is not necessarily the previous calendar day (`tools.py:126-132`).
- **C-46** In `get_price_table`, gold (XAU) is shown as USD per troy ounce plus Rp per gram and Rp per 0.01 gram, using 31.1034768 grams per troy ounce (`tools.py:35`, `tools.py:156-157`, `tools.py:349-351`).
- **C-47** Gold's % change is computed from the USD-per-ounce series, not the IDR series (`tools.py:159-161`).
- **C-48** If the IDR gold series has no value for the latest USD date, the latest IDR date's value is used instead, while the row still shows the USD date (`tools.py:152-154`, `tools.py:170`).
- **C-49** In `get_price_table`, XAG has no special branch: it is priced like a currency, as Rp per troy ounce, with the label "Silver (XAG, per troy oz)" (`tools.py:50`, `tools.py:357-359`).
- **C-50** In `get_price_table`, IHSG's price is the last close of a `period="1d", interval="1m"` history if that data is not empty, otherwise the last daily close from a 5-day history (`tools.py:180`, `tools.py:186-193`).
- **C-51** IHSG's % change compares that price with the second-to-last row of the 5-day daily history (`tools.py:184`, `tools.py:195`).
- **C-52** For Frankfurter-based rows (currencies and XAU), the "Last Update" column shows the rate date plus the local time of the check (`DATE • checked HH:MM`) (`tools.py:77-79`, `tools.py:353`, `tools.py:360`).
- **C-53** For the IHSG row, the "Last Update" column shows the timestamp of the last yfinance row used (`tools.py:189`, `tools.py:193`).
- **C-54** In the price table, a change ≥ 0 is shown with 🔺 and a negative change with 🔻, signed to 2 decimals; a missing change is shown as `n/a` (`tools.py:376-380`).
- **C-55** `web_search` labels each result with the host name of its URL, lower-cased and with a leading `www.` removed, or "unknown site" if there is no host (`tools.py:416`, `tools.py:425-428`).
- **C-56** `get_average_exchange_rate` averages every date in the returned cross series with equal weight (`tools.py:286-287`).

## 5. External data sources and dependencies

- **C-57** Frankfurter is called at `https://api.frankfurter.dev/v2/rates` (`tools.py:34`, `tools.py:100`).
- **C-58** The Frankfurter request sends only `base`, optional `quotes`, and optional date parameters; the code sends no API key (`tools.py:94-100`).
- **C-59** The Frankfurter request uses a 15-second timeout and raises on HTTP error status (`tools.py:100-101`).
- **C-60** yfinance is used only for ticker `^JKSE` (IHSG) (`tools.py:39`, `tools.py:179`, `tools.py:204`).
- **C-61** Each yfinance `history` call passes a 10-second timeout (`tools.py:36`, `tools.py:180`, `tools.py:186`, `tools.py:205`).
- **C-62** The `yfinance` logger level is set to CRITICAL (`tools.py:32`).
- **C-63** `web_search` asks DuckDuckGo for at most 4 results (`tools.py:412`).
- **C-64** The chat model is `ChatGoogleGenerativeAI` with model `"gemini-3.5-flash-lite"` (`agent.py:12`, `agent.py:33-34`).
- **C-65** `requirements.txt` pins exact versions of langchain, langchain-core, langchain-google-genai, langgraph, google-genai, yfinance, ddgs, requests, python-dotenv, and streamlit (`requirements.txt:2-11`).
- **C-66** The terminal path (`currency_agent.py`, `cli.py`, `agent.py`, `tools.py`, `prompts.py`, `trace_log.py`) does not import streamlit; only `app.py` does (`app.py:10`; imports at `cli.py:7-15`, `agent.py:8-17`, `tools.py:18-28`).

## 6. Configuration (keys, env vars, settings)

- **C-67** `agent.py` calls `load_dotenv()` and then reads `GOOGLE_API_KEY` from the environment, trimmed of spaces (`agent.py:19-21`).
- **C-68** If `GOOGLE_API_KEY` is empty or equals `your-key-here`, importing `agent.py` raises `SystemExit` with "GOOGLE_API_KEY not set — copy .env.example to .env and add your key." (`agent.py:22-24`).
- **C-69** `.env.example` contains `GOOGLE_API_KEY=your-key-here`, the same placeholder `agent.py` rejects (`.env.example:3`, `agent.py:22`).
- **C-70** The env var `PROMPT_VERSION` picks the system prompt; the default is `v2` (`agent.py:43`, `prompts.py:47`).
- **C-71** A `PROMPT_VERSION` not in `PROMPTS` makes importing `agent.py` raise `SystemExit` listing the valid versions (`agent.py:44-45`).
- **C-72** The Gemini model is configured with `timeout=30` and `max_retries=2`, and no temperature is set (`agent.py:33-38`).
- **C-73** The model name, timeouts (15 s Frankfurter, 10 s yfinance, 30 s Gemini), and result count (4) are hard-coded, not read from config (`agent.py:34-37`, `tools.py:36`, `tools.py:100`, `tools.py:412`).
- **C-74** The log file name is the relative path `agent_log.txt` (`trace_log.py:11`).
- **C-75** `.streamlit/config.toml` sets `server.address = "127.0.0.1"` and `browser.gatherUsageStats = false` (`.streamlit/config.toml:1-9`).
- **C-76** The terminal truncates the first line of a tool result to 100 characters, adding `...`, and appends `(+N lines)` when the result has more lines (`cli.py:18`, `cli.py:32-40`).
- **C-122** The web page uses the same 100-character / `(+N lines)` shortening for tool-result steps, because `_step_md` calls `cli.format_step`, which calls `_summarize` (web; `app.py:57-58`, `cli.py:48`).

## 7. Error handling

- **C-77** The failure strings the four tools return (empty results and exceptions caught inside their `try` blocks) are built by `_tool_error` and start with `ERROR:`; the per-asset `FAILED:` rows inside a partly successful price table are the exception (`tools.py:43`, `tools.py:64-67`, `tools.py:366`, `tools.py:384-390`).
- **C-78** `_tool_error` also writes `❌ [TOOL FAILED] call -> reason` to the log (`tools.py:66`).
- **C-79** A tool result step is marked failed (❌) when the ToolMessage status is `"error"` or its content starts with `ERROR:` (both surfaces; `agent.py:122-125`, `cli.py:47`).
- **C-80** In `run_turn`, any `Exception` raised while streaming the agent, including one raised by the `on_step` callback, is logged as `[AGENT ERROR] kind: …` and re-raised as `AgentRunError` (`agent.py:163-175`).
- **C-81** `_classify_error` picks the kind by keyword search in the error type name and message, checked in order: `quota` ("429", "resource_exhausted", "quota", "rate limit"), `timeout` ("timeout", "timed out", "deadline"), `auth` ("api key", "api_key", "permission_denied", "unauthenticated"), else `other` (`agent.py:67-76`).
- **C-82** If the stream yields no state or the last message is not an `AIMessage`, `run_turn` raises `AgentRunError("other", "the agent stopped without a final answer")` (`agent.py:177-180`).
- **C-83** When a turn raises, the stored history is not updated: in the terminal `run_request` assigns history only after `run_turn` returns (`agent.py:138`), and on the web page the tuple assignment to `st.session_state.history` happens only on return (`app.py:99-100`).
- **C-84** If the final answer has no text, `run_turn` logs `[AGENT WARNING] final answer had no text` and returns "(The model returned no text answer. Please try again or rephrase the question.)" (`agent.py:26`, `agent.py:185-189`).
- **C-121** When the final message content is a list, the answer keeps only plain-string blocks and dict blocks with `type == "text"`, joins them, and strips surrounding whitespace; other blocks are dropped. String content is returned unchanged, and any other type is converted with `str()` (both surfaces; `agent.py:82-95`, `agent.py:185`).
- **C-85** In the terminal, an `AgentRunError` prints the message for its kind from `ERROR_MESSAGES` (falling back to the "other" message) and "(This question was not saved to the conversation. Details are in the log.)" (`cli.py:21-26`, `cli.py:63-68`).
- **C-86** In the terminal, for kind `other` the reason is also printed, shortened by `_summarize` (`cli.py:65-66`).
- **C-87** In the terminal, Ctrl+C while a question is being answered prints "[Cancelled] This question was not saved to the conversation." and returns to the prompt (`cli.py:69-71`).
- **C-88** In the terminal, any other `Exception` during a turn is logged as `[CLI ERROR] …` and shown as "⚠️ Unexpected error: …" without ending the chat loop (`cli.py:72-76`).
- **C-89** On the web page, an `AgentRunError` shows the same `ERROR_MESSAGES` text plus " This question was not saved to the conversation." in an error box, and the status box is labeled "Failed" (`app.py:104-108`).
- **C-90** On the web page, the reason of an `other` error is not shown to the user (`app.py:104-108`).
- **C-91** On the web page, any other `Exception` is logged as `[UI ERROR] …` and shown as "Unexpected error: …" with the full message (`app.py:109-114`).
- **C-92** On the web page, failed turns are stored in `st.session_state.turns` and redrawn with their error on later page runs (`app.py:115`, `app.py:79-80`).
- **C-93** On the web page, a `SystemExit` while importing `agent` (missing key or bad `PROMPT_VERSION`) is shown with `st.error` and the page stops (`app.py:14-18`).
- **C-94** In the terminal, the `SystemExit` from importing `agent` is not caught, so the program exits with that message (`currency_agent.py:23`, `cli.py:7`, `agent.py:22-24`).

## 8. Edge cases and known limits

- **C-95** A partly failed price table is not prefixed `ERROR:`, so its tool-result step is shown as ✅ and is not counted in the web page's "❌ N failed" label (`tools.py:384-390`, `agent.py:124`, `app.py:61-67`).
- **C-96** An unknown code in `get_price_table` (for example a crypto ticker) goes to the currency branch and becomes a `FAILED: no data for CODE` row if Frankfurter returns no rate for it (`tools.py:357-358`, `tools.py:123-124`, `tools.py:363-366`).
- **C-97** In `get_price_table`, only the literal code `IHSG` uses yfinance; `get_average_exchange_rate` routes to yfinance for codes in `INDEX_TICKERS`, which contains only `IHSG` (`tools.py:39`, `tools.py:273`, `tools.py:354`).
- **C-98** The IHSG average path checks both dates with `date.fromisoformat`; a bad date becomes `ERROR: could not fetch IHSG history: …` (`tools.py:202-203`, `tools.py:312-313`).
- **C-99** The currency average path does not validate date formats in code; dates are passed straight to Frankfurter (`tools.py:276-279`).
- **C-100** The date in the system prompt is computed once when `prompts.py` is first imported, so a process running past midnight keeps the earlier date (`prompts.py:8`, `prompts.py:11-12`).
- **C-101** The 7-day window used by `get_price_table` for currencies and gold is computed from the local date at call time (`tools.py:120`, `tools.py:144`).
- **C-102** `trace_log.start_session()` opens the log in write mode, erasing earlier content, then writes a `=== NEW SESSION STARTED: … ===` line (`trace_log.py:14-17`).
- **C-103** `start_session()` is called when the terminal chat starts, on terminal `reset`/`clear`, and each time a web session starts or the web reset button is pressed (`cli.py:127`, `agent.py:109-110`, `app.py:34-37`, `app.py:44-45`, `app.py:129`).
- **C-104** All web sessions and the terminal write to the same `agent_log.txt` path, so a new web session clears the log of any other session using that working directory (`trace_log.py:11`, `trace_log.py:14-15`, `app.py:37`).
- **C-105** In the terminal on Windows (when `msvcrt` imports), pressing ESC at the prompt quits; on other systems, EOF or Ctrl+C at the prompt quits (`cli.py:11-15`, `cli.py:98-110`).
- **C-106** On both platforms, quitting from the prompt via `_read_line_with_esc` returning `None` prints "[ESC pressed] Closing the program. Goodbye!", even when the cause was Ctrl+C or EOF on non-Windows (`cli.py:100-102`, `cli.py:140-142`).
- **C-107** In the terminal, typing `exit`, `quit`, `bye`, or `esc` (case-insensitive) quits with "Goodbye!" (`cli.py:147-149`).
- **C-108** In the terminal, an empty or whitespace-only line is ignored (`cli.py:144-146`).
- **C-109** The Windows input reader handles only Enter, ESC, and backspace specially; every other character read by `msvcrt.getwch()` is added to the line (`cli.py:106-120`).
- **C-110** On the web page, question and answer text has `$` escaped and `<br>` replaced by a space before `st.markdown`; error texts are not passed through this (`app.py:51-54`, `app.py:73`, `app.py:82`, `app.py:88`, `app.py:103`, `app.py:108`, `app.py:114`).
- **C-111** On the web page, step text is shown as inline code with backticks replaced by `'` (`app.py:57-58`).
- **C-112** The web step label reads "No tool needed" when there were no tool calls (`app.py:61-66`).
- **C-123** On the web page, if a page run has both typed text and a pending example, the typed text is run and `st.session_state.pop("pending")` is not evaluated, so the example stays in `st.session_state.pending` until a later page run with no typed text pops it, or another example click replaces it (web only, `app.py:40-41`, `app.py:140-143`).
- **C-113** The conversation history sent to the model is held only in memory, in `agent._conversation_history` (terminal) or `st.session_state.history` (web), and no in-scope code loads it back from disk; `trace_log.py` only opens the log file for writing or appending (`agent.py:98`, `agent.py:138`, `app.py:35`, `app.py:99-100`, `trace_log.py:15`, `trace_log.py:28`). The questions and answers written to the log file are covered by C-117, C-118, and C-125.

## 9. Out of scope / non-goals

- **C-114** The tools have no crypto-specific code path or data source: every code given to `get_exchange_rate`, a code other than `XAU` or `IHSG` in `get_price_table`, and a code other than `IHSG` in `get_average_exchange_rate` is sent to Frankfurter like a currency; the only stock index with a yfinance mapping is `IHSG` (`tools.py:39`, `tools.py:233`, `tools.py:273-279`, `tools.py:347-358`; whether Frankfurter serves any crypto code: Q-16).
- **C-115** The code has no user login or access control on the web page; access is limited only by the `127.0.0.1` bind address in config (`app.py:1-143`, `.streamlit/config.toml:5`).
- **C-116** No in-scope code places trades, sends alerts, or schedules background checks; tools only read prices and search (`tools.py:214-431`, `cli.py:137-155`, `app.py:140-143`).

## 10. Open questions (things the code doesn't settle)

- **Q-01** Who is the target user (e.g. an Indonesian individual tracking IDR prices)? The code shows rupiah as the price-table base (`tools.py:358`) and the sidebar mentions rupiah (`app.py:123`), but users are not visible in code.
- **Q-02** Is Frankfurter free and keyless, and is its data daily rather than real-time? Only the docstring says so (`tools.py:4-9`); C-58 states only that no key is sent.
- **Q-03** With no date parameter, does Frankfurter return only the latest date? C-34 takes the max date of whatever comes back; the response shape is runtime behavior.
- **Q-04** How many dates does Frankfurter return for weekends/holidays? This decides "N days of data" (C-35) and what "previous day" means in C-45.
- **Q-05** Is the yfinance 1-minute data actually the real market time, and does the daily history include today's row during trading? This decides C-50/C-51; docstrings claim real time (`tools.py:10-11`, `tools.py:176`).
- **Q-06** Does the model follow the system prompt rules (C-18–C-23), e.g. fall back to `web_search` after a tool error? Not visible in code.
- **Q-07** Does `max_retries=2` retry on 429 and timeouts, and does the model really ignore temperature? Only a comment says so (`agent.py:31-32`, `agent.py:37`).
- **Q-08** Do real Gemini error messages contain the keywords in `_classify_error` (C-81)? Needs live error samples.
- **Q-09** Which `.env` does `load_dotenv()` find (the comment says "this folder", `agent.py:19`)? Library search behavior, not in code.
- **Q-10** Is `st.session_state` separate per browser tab (C-11), and is it safe for several web tabs to use the one module-level `agent` object and log file at once? Runtime concurrency is not shown in code.
- **Q-11** Does binding to `127.0.0.1` take effect and block other machines? Runtime Streamlit behavior; the intent is only stated in a comment (`.streamlit/config.toml:2-4`).
- **Q-12** Is `cli.ask()` (C-33) used by out-of-scope code such as the eval? Out-of-scope files were not read.
- **Q-13** Which Python version is required? Only a comment says "Python 3.10" (`requirements.txt:1`).
- **Q-14** What does `msvcrt.getwch()` return for arrow keys or Ctrl+C at the Windows prompt (C-109)? Runtime behavior.
- **Q-15** (from C-38) Does `ddgs` `DDGS().text(query, max_results=4)` ever return more than 4 items, and can a result's title or body contain newlines (which would add lines to the output)? Library and runtime behavior; the code does not cut the list or strip newlines (`tools.py:412-418`).
- **Q-16** (from C-114) Does Frankfurter serve any crypto code? The code sends such codes to Frankfurter like currencies (C-114); the supported code list is external API behavior. Only docstrings say crypto needs `web_search` (`tools.py:12-13`, `tools.py:219-220`).

## 11. Change log

- C-29 — fixed — reviewer row C-29 (incorrect): DuckDuckGo has no data helper; it is called directly inside `web_search`. Wording also names the log/time helpers (`tools.py:58-79`), which the reviewer's correction left out.
- C-38 — fixed — reviewer row C-38 (unresolved): output now states only what the code does (one entry per returned result, requests 4, no slicing); the library question moved to new Q-15.
- C-113 — fixed — reviewer row C-113 (incorrect): narrowed to the in-memory conversation history sent to the model; the on-disk log of questions and answers is now stated in C-117, C-118, C-125.
- C-114 — fixed — reviewer row C-114 (unresolved): rewritten to the code routing only (no crypto-specific path; non-special codes go to Frankfurter; only IHSG maps to yfinance); whether Frankfurter serves crypto moved to new Q-16.
- C-117 — added — omission 1 (each turn's question and timestamp logged).
- C-118 — added — omission 1 (final answer logged on success).
- C-119 — added — omission 1 (`[TOOL CALLED]` log line).
- C-120 — added — omission 1 (`[TOOL OK]` log line).
- C-121 — added — omission 2 (`extract_text` keeps only text blocks).
- C-122 — added — omission 3 (web page reuses `_summarize` truncation).
- C-123 — added — omission 4 (typed text wins over a pending example).
- C-124 — added — omission 5 (terminal banner contents).
- C-125 — added — omission 1 / reviewer row C-113 (log file stays on disk until the next session start overwrites it).
- Q-15 — added — from C-38.
- Q-16 — added — from C-114.
