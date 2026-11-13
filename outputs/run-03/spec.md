# Spec — Agentic Financial Monitoring Assistant
- Input repo: input/demo-a/ (Demo A)
- Commit: 6caaf96 (tag demo-v1)
- Skill version: v2
- Run ID: run-03
- Date: 2026-11-13
- Stage: revised

All locations are relative to `input/demo-a/`.

---

## 1. Purpose and target user

- **C-01** The product is a chat assistant named "Asset Price Assistant" in both the terminal banner and the web page title, sidebar header, and page heading (cli.py:130; app.py:12, app.py:122, app.py:135).
- **C-02** It answers questions with a Gemini chat model (`ChatGoogleGenerativeAI`) wrapped in a LangChain agent built by `create_agent` with four tools (agent.py:33-38; agent.py:48-50; tools.py:431).
- **C-03** It has two interfaces: a terminal chat started by running `currency_agent.py`, and a Streamlit web page in `app.py` (currency_agent.py:23-26; app.py:10).
- **C-04** Both interfaces use the same agent object from `agent.py`, so the same model, tools, and system prompt version (cli.py:62; app.py:99-100; agent.py:53).

The target user is not stated in code; see Q-01.

## 2. User stories / features

- **C-05** The user can get a table of several assets in one tool call; with no `assets` argument the table covers USD, SGD, TWD, XAU, and IHSG (tools.py:319-320).
- **C-06** The user can get the latest exchange rate for one currency pair (tools.py:214-243).
- **C-07** The user can get the average exchange rate between two currencies over a start/end date range (tools.py:251-292).
- **C-08** The user can get the average daily closing value of the IHSG index over a date range, through `get_average_exchange_rate` with `from_currency='IHSG'` (tools.py:272-274; tools.py:297-311).
- **C-09** The agent can run a DuckDuckGo web search (tools.py:396-420).
- **C-10** Both interfaces send the earlier messages of the current conversation (including tool calls and tool results) with each new question (agent.py:160; agent.py:165; agent.py:183; agent.py:138; app.py:99-100).
- **C-11** Both interfaces show each tool call and each tool result as a step while the turn is running, via the `on_step` callback (agent.py:165-171; cli.py:51-53; cli.py:62; app.py:93-95; app.py:100).
- **C-12** Terminal: typing `reset` or `clear` (case-insensitive, after trimming spaces) clears the conversation history and prints a reset notice (cli.py:144; cli.py:150-153; agent.py:105-110).
- **C-13** Web page: the "🔄 Reset conversation" sidebar button clears the page's conversation history and its list of shown turns (app.py:129; app.py:34-36).
- **C-14** Web page: the sidebar shows four example questions as buttons; clicking one stores it in `st.session_state.pending`, which the page script runs as the question when nothing was typed (app.py:23-28; app.py:40-41; app.py:126-127; app.py:141-143).
- **C-15** Terminal: the user quits by pressing ESC (Windows input path) or by typing `exit`, `quit`, `bye`, or `esc` (case-insensitive) (cli.py:107-110; cli.py:140-149).
- **C-16** Both interfaces write a log file, `agent_log.txt`, that receives each question, each tool call and result line, and the final answer (trace_log.py:11; agent.py:158; agent.py:190-191; tools.py:66; tools.py:231; tools.py:242).
- **C-17** Terminal: the start banner prints the log file name and example questions (cli.py:129-135).

## 3. Components (files and their responsibility)

- **C-18** `currency_agent.py` reconfigures `sys.stdout` to UTF-8 with `errors="replace"` before importing `cli`; the call is not inside an OS check, so it runs on all systems (currency_agent.py:21-23).
- **C-19** `currency_agent.py` calls `cli.chat()` when run as a script (currency_agent.py:25-26).
- **C-20** `cli.py` holds the terminal chat: line input, step printing, answer printing, and terminal error messages (cli.py:21-26; cli.py:43-78; cli.py:92-155).
- **C-21** `agent.py` loads the `.env` file, checks the API key, builds the model and agent, holds the terminal's conversation history, runs turns, and classifies errors (agent.py:19-24; agent.py:33-53; agent.py:67-76; agent.py:98-192).
- **C-22** `tools.py` defines the four tools and their data helpers, and exports them as `ALL_TOOLS` (tools.py:85-208; tools.py:214-428; tools.py:431).
- **C-23** `prompts.py` defines two system prompt versions, `v1` and `v2`, in a `PROMPTS` dict (prompts.py:11-46).
- **C-24** `trace_log.py` writes the log file: `start_session` overwrites it, `log_request` and `log` append to it (trace_log.py:14-29).
- **C-25** `app.py` is the Streamlit web page; it has no `if __name__ == "__main__"` block and runs top to bottom (app.py:10-143).
- **C-26** `app.py` imports `cli` to reuse `format_step` and `ERROR_MESSAGES`: the step text and the per-kind base error messages are shared with the terminal, but the web page wraps each step in backticks (replacing any backtick inside it with `'`), and the wording around errors differs: a different "not saved to the conversation" note, no `Reason:` line for kind `other`, and the full rather than summarized text for unexpected errors (app.py:20; app.py:58; app.py:105-106; app.py:112; cli.py:64-67; cli.py:75).
- **C-27** `.streamlit/config.toml` holds Streamlit server and browser settings (.streamlit/config.toml:1-9).
- **C-28** `requirements.txt` pins 10 packages to exact versions (`==`), including langchain 1.3.14, langchain-google-genai 4.3.2, yfinance 1.5.2, ddgs 9.14.4, and streamlit 1.65.0 (requirements.txt:2-11).
- **C-29** `requirements-dev.txt` includes `requirements.txt` and adds pytest 9.2.0 and PyYAML 6.0.3 (requirements-dev.txt:2-4).
- **C-30** `pytest.ini` sets `testpaths = tests` and `pythonpath = .` (pytest.ini:1-3).

## 4. Tools

| Claim | Name | Inputs | Output | Data source | Failure behavior |
|---|---|---|---|---|---|
| C-31 | `get_exchange_rate` (tools.py:214-245) | `from_currency`, `to_currency` (str) | `"1 FROM = rate TO (as of DATE)"` (tools.py:241) | Frankfurter `/rates`, no date parameter (tools.py:233; tools.py:94-100) | empty data → `ERROR:` text suggesting web_search; exception inside the `try` → `ERROR: could not fetch exchange rate: …` (tools.py:234-238; tools.py:244-245) |
| C-32 | `get_average_exchange_rate` (tools.py:251-313) | `from_currency`, `to_currency`, `start_date`, `end_date` (str) | average, number of days, min, max (tools.py:288-290; tools.py:307-309) | Frankfurter `/rates` with `from`/`to`; yfinance `^JKSE` when `from_currency` is IHSG (tools.py:276-279; tools.py:204-205) | empty data → `ERROR:` text; exception inside the `try` → `ERROR: could not fetch historical rates: …` or `ERROR: could not fetch IHSG history: …` (tools.py:281-284; tools.py:293-294; tools.py:301-305; tools.py:312-313) |
| C-33 | `get_price_table` (tools.py:319-390) | `assets` (str, comma-separated; default `"USD,SGD,TWD,XAU,IHSG"`) | Markdown table: Asset, Price, Change (vs previous day), Last Update (tools.py:368-382) | Frankfurter (currencies, XAU, XAG); yfinance `^JKSE` (IHSG) (tools.py:347-360) | one asset failing → that row shows `FAILED: …`; all assets failing → `ERROR:` text plus the table (tools.py:363-366; tools.py:384-387) |
| C-34 | `web_search` (tools.py:396-422) | `query` (str) | one `- [site] title: body` entry per result returned by `DDGS().text(query, max_results=4)`, joined with newlines; the code itself does not cap the count (see Q-14) (tools.py:412; tools.py:416-418) | DuckDuckGo via `ddgs.DDGS` (tools.py:25; tools.py:412) | no results → `ERROR: no search results found.`; exception inside the `try` → `ERROR: search failed: …` (tools.py:413-414; tools.py:421-422) |

### Shared tool behavior
- **C-35** Each of the four tools writes a `🔧 [TOOL CALLED] name(args)` line to the log before fetching data (tools.py:231; tools.py:271; tools.py:339; tools.py:410; tools.py:58-61).
- **C-36** Tool failures built by `_tool_error` return text starting with `ERROR: ` and write a `❌ [TOOL FAILED] call -> reason` log line (tools.py:43; tools.py:64-67).
- **C-37** The agent module marks a tool result as failed when the tool message status is `"error"` or its content starts with `ERROR:` (agent.py:122-125).
- **C-38** `ALL_TOOLS` lists the four tools in this order: `get_exchange_rate`, `get_average_exchange_rate`, `get_price_table`, `web_search`, and is passed to `create_agent` (tools.py:431; agent.py:50).

### Frankfurter cross-rate helper
- **C-39** `_fetch_eur_cross_series` uppercases both codes and requests rates with `base=EUR` and `quotes` set to the sorted non-EUR codes; when both codes are EUR it sends no `quotes` parameter (tools.py:91-96).
- **C-40** The cross rate for each date is `rate_to / rate_from`, using 1.0 for EUR (tools.py:108-113).
- **C-41** A date is left out of the cross series when either of the two rates is missing or zero for that date (tools.py:110-113).
- **C-42** The helper does not strip spaces from currency codes; it only uppercases them (tools.py:91).

### `get_exchange_rate`
- **C-43** It returns the rate for the latest date found in the response (tools.py:239-240).
- **C-44** The rate is formatted with 6 significant digits (`.6g`) (tools.py:241).

### `get_average_exchange_rate`
- **C-45** The IHSG path is taken when `from_currency`, after stripping spaces and uppercasing, is a key of `INDEX_TICKERS` (only `IHSG`) (tools.py:39; tools.py:272-274).
- **C-46** On the currency path, the average is an unweighted mean of the per-date cross rates returned (tools.py:286-287).
- **C-47** On the currency path, the code does not check the date format before calling the API; dates are passed as given (tools.py:276-279).
- **C-48** On the IHSG path, `to_currency` does not affect the data fetched or the returned text; it appears only in the `call` string that is written to the log (tools.py:268-270; tools.py:274; tools.py:297-313; tools.py:64-67).
- **C-49** On the IHSG path, yfinance is asked for history up to `end_date` plus one day, so `end_date` is included (tools.py:201-205).
- **C-50** On the IHSG path, both dates are parsed with `date.fromisoformat`; a bad date raises inside the `try` and returns `ERROR: could not fetch IHSG history: …` (tools.py:202-203; tools.py:299-300; tools.py:312-313).
- **C-51** On the IHSG path, an empty history returns an `ERROR:` text saying the market may have been closed (tools.py:206-207; tools.py:301-305).
- **C-52** On the IHSG path, the average, min, and max are formatted with 2 decimals and labelled "points", with the count labelled "trading days" (tools.py:306-309).

### `get_price_table`
- **C-53** The `assets` text is split on commas; each code is stripped and uppercased, and empty codes are dropped (tools.py:337).
- **C-54** If no codes remain, it returns `ERROR: no asset codes given.` (tools.py:340-341).
- **C-55** Duplicate codes are not removed; each one produces its own row (tools.py:337; tools.py:344-362).
- **C-56** The row label comes from `ASSET_LABELS` (USD, SGD, TWD, "Gold (XAU)", "Silver (XAG, per troy oz)", IHSG); other codes are shown as the code itself (tools.py:45-52; tools.py:345).
- **C-57** The XAU row shows USD per troy ounce (2 decimals), rupiah per gram, and rupiah per 0.01 gram (0 decimals) (tools.py:347-351).
- **C-58** Gold rupiah per gram is the IDR per troy ounce price divided by 31.1034768 (tools.py:35; tools.py:156-157).
- **C-59** The XAU % change is computed from the USD-per-ounce series, not the IDR series (tools.py:159-161).
- **C-60** For XAU, the IDR price uses the IDR rate on the latest USD-series date, and falls back to the latest IDR-series date if that date has no IDR value (tools.py:151-154).
- **C-61** The IHSG row shows the index in points with 2 decimals (tools.py:355-356).
- **C-62** The IHSG price is the last close in the 1-minute data requested with `period="1d", interval="1m"`; if that data is empty, it is the last close in the `period="5d"` daily data (tools.py:180; tools.py:186-193).
- **C-63** The IHSG % change compares the price with the second-to-last close in the 5-day daily data; with fewer than 2 daily rows it is `None` (tools.py:184; tools.py:195).
- **C-64** The IHSG "Last Update" is the timestamp of the yfinance row used, formatted `YYYY-MM-DD HH:MM` (tools.py:189; tools.py:193).
- **C-65** Every code other than XAU and IHSG (including XAG) is priced in IDR from Frankfurter and shown as `Rp` with 2 decimals (tools.py:357-359).
- **C-66** For Frankfurter-based rows (XAU and the other codes), data is requested from 7 days before today, and the % change compares the latest date with the previous date that has data in that window (tools.py:120-121; tools.py:126-132; tools.py:144-146; tools.py:159-161).
- **C-67** For Frankfurter-based rows, "Last Update" is the rate date plus `• checked HH:MM`, using the local clock (tools.py:73-79; tools.py:353; tools.py:360).
- **C-68** A `None` % change is shown as `n/a`; otherwise as 🔺 (for ≥ 0) or 🔻 (for < 0) followed by a signed value with 2 decimals (tools.py:376-380).
- **C-69** An exception for one asset is caught; that row shows `-` for price and change and `FAILED: <error>` as Last Update, and the other assets are still processed (tools.py:363-366; tools.py:372-375).
- **C-70** When every row failed, the result is `ERROR: could not fetch any of [...]` followed by the table (tools.py:384-387).
- **C-71** When at least one row succeeded, the table is returned without the `ERROR:` prefix (tools.py:384-390).

### `web_search`
- **C-72** It calls `DDGS().text(query, max_results=4)` (tools.py:412).
- **C-73** The site name in each line is the URL's `netloc` (host plus any port or user info), lowercased, with a leading `www.` removed; an empty `netloc`, including when the result has no `href` key, becomes `unknown site` (tools.py:416; tools.py:425-428).

## 5. External data sources and dependencies

- **C-74** Frankfurter requests go to `https://api.frankfurter.dev/v2/rates` (tools.py:34; tools.py:100).
- **C-75** Frankfurter requests use a 15-second timeout (tools.py:100).
- **C-76** The code calls `raise_for_status()` on the Frankfurter response before reading its JSON (tools.py:101-102).
- **C-77** The code reads the Frankfurter JSON as a list of rows with `date`, `quote`, and `rate` fields (tools.py:102-106).
- **C-78** The code sends no API key or auth header to Frankfurter (tools.py:94-100).
- **C-79** yfinance is used for IHSG with ticker `^JKSE` (tools.py:39; tools.py:179; tools.py:204).
- **C-80** Each yfinance `history` call passes a 10-second timeout (tools.py:36; tools.py:180; tools.py:186; tools.py:204-205).
- **C-81** The `yfinance` logger level is set to CRITICAL at import (tools.py:32).
- **C-82** DuckDuckGo search uses `ddgs.DDGS` with no API key passed (tools.py:25; tools.py:412).
- **C-83** The model is `gemini-3.5-flash-lite` (agent.py:34).
- **C-84** The model is created with `timeout=30` and `max_retries=2` (agent.py:36-37).
- **C-85** No temperature is passed to the model (agent.py:33-38).
- **C-86** `streamlit` is imported in `app.py` and in no other in-scope Python file (app.py:10).
- **C-87** `load_dotenv()` runs when `agent.py` is imported (agent.py:10; agent.py:19).

## 6. Configuration (keys, env vars, settings)

- **C-88** `GOOGLE_API_KEY` is read from the environment and stripped of surrounding spaces (agent.py:21).
- **C-89** If the key is empty or equals `your-key-here`, importing `agent.py` raises `SystemExit` with the message "GOOGLE_API_KEY not set — copy .env.example to .env and add your key." (agent.py:22-24).
- **C-90** `.env.example` contains `GOOGLE_API_KEY=your-key-here` (.env.example:3).
- **C-91** `PROMPT_VERSION` is read from the environment; the default is `v2` (agent.py:43; prompts.py:47).
- **C-92** An unknown `PROMPT_VERSION` raises `SystemExit` that lists the valid versions (agent.py:44-45).
- **C-93** The system prompt version is chosen once, when `agent.py` is imported (agent.py:43; agent.py:53).
- **C-94** The model name is set in code, not read from an environment variable (agent.py:34).
- **C-95** Prompt `v2` is prompt `v1` plus two added paragraphs (prompts.py:30-40).
- **C-96** The system prompt (both versions) starts with "Today is <date>", where the date is computed once when `prompts.py` is imported (prompts.py:8; prompts.py:12; prompts.py:30).
- **C-97** The system prompt (both versions) tells the agent to always use the tools to get numbers and not to guess or use memory (prompts.py:12-15).
- **C-98** The system prompt (both versions) tells the agent to use `get_price_table` for a price update, market summary, or several assets; `get_exchange_rate` only for one pair; and `get_average_exchange_rate` for averages or comparing periods (prompts.py:16-19).
- **C-99** The system prompt (both versions) tells the agent to try `web_search` if a structured data tool fails (prompts.py:20-21).
- **C-100** The system prompt (both versions) tells the agent to answer questions unrelated to exchange rates or asset prices without calling a tool (prompts.py:22-23).
- **C-101** Prompt `v2` also tells the agent to use `web_search` for "why" and news questions, to give only reasons found in the search results with their source, and to say so if the results don't explain the move (prompts.py:32-36).
- **C-102** Prompt `v2` also tells the agent to answer a single past date by calling `get_average_exchange_rate` with start and end set to that date (prompts.py:37-39).
- **C-103** `.streamlit/config.toml` sets `server.address = "127.0.0.1"` (.streamlit/config.toml:1; .streamlit/config.toml:5).
- **C-104** `.streamlit/config.toml` sets `browser.gatherUsageStats = false` (.streamlit/config.toml:7; .streamlit/config.toml:9).
- **C-105** The log file name is the relative path `agent_log.txt` (trace_log.py:11).

## 7. Error handling

- **C-106** Exceptions raised while the agent streams a turn are caught, logged as `[AGENT ERROR] kind: Type: message`, and re-raised as `AgentRunError` with a `kind` (agent.py:163-175).
- **C-107** The `kind` is chosen by substring match on the lowercased error type name plus message, in this order: `quota` (429, resource_exhausted, quota, rate limit), `timeout` (timeout, timed out, deadline), `auth` (api key, api_key, permission_denied, unauthenticated), else `other` (agent.py:67-76).
- **C-108** If the stream yields no state, or the last message is not an AI message, the turn raises `AgentRunError("other", "the agent stopped without a final answer")` (agent.py:177-180).
- **C-109** When a turn raises `AgentRunError`, the conversation history is left unchanged in both interfaces (agent.py:137-138; agent.py:182-183; app.py:99-100).
- **C-110** If the final answer has no text, the answer is replaced by "(The model returned no text answer. Please try again or rephrase the question.)" and a `[AGENT WARNING]` line is logged (agent.py:26; agent.py:185-189).
- **C-111** `ERROR_MESSAGES` maps the four kinds to user texts; for example, `quota` is "Gemini quota or rate limit reached (429). Wait a minute, then try again." (cli.py:21-26).
- **C-112** Terminal: on `AgentRunError`, it prints `⚠️` with the message for that kind and a note that the question was not saved to the conversation (cli.py:63-68).
- **C-113** Terminal: for kind `other` only, it also prints `Reason:` with a one-line summary of the error (cli.py:65-66).
- **C-114** Web page: on `AgentRunError`, it shows `st.error` with the message for that kind plus " This question was not saved to the conversation." and sets the status box to "Failed"; it shows no reason text for kind `other` (app.py:104-108).
- **C-115** Terminal: a `KeyboardInterrupt` during a turn prints "[Cancelled] This question was not saved to the conversation." (cli.py:69-71).
- **C-116** Terminal: any other `Exception` during a turn is logged as `[CLI ERROR]` and printed as "Unexpected error:" with a one-line summary (cli.py:72-76).
- **C-117** Web page: any other `Exception` during a turn is logged as `[UI ERROR]` and shown with `st.error` as "Unexpected error: <full error text>" (app.py:109-114).
- **C-118** Web page: a failed turn is still added to the list of shown turns, with its error text, and `_show_turn` draws it with `st.error` when earlier turns are shown (app.py:115; app.py:79-80; app.py:137-138).
- **C-119** Web page: if importing `agent` raises `SystemExit` (missing key or unknown prompt version), the page shows the message with `st.error` and stops (app.py:14-18; agent.py:22-24; agent.py:44-45).
- **C-120** Terminal: neither `currency_agent.py` nor `cli.py` catches the `SystemExit` raised when `agent` is imported (cli.py:7; currency_agent.py:23).

## 8. Edge cases and known limits

- **C-121** Because a partly failed price table has no `ERROR:` prefix, its step is shown as a success (✅) in both interfaces (tools.py:389-390; agent.py:124; cli.py:47; app.py:58).
- **C-122** If only one date has data in the 7-day window, the Frankfurter-based row shows `n/a` as change (tools.py:130-134; tools.py:159-163; tools.py:376-377).
- **C-123** The date in the system prompt does not update while the process keeps running (prompts.py:8; agent.py:53).
- **C-124** `start_session` opens the log file in write mode, which empties it; it is called when the terminal chat starts, on terminal `reset`/`clear`, when a web session starts, and on the web Reset button (trace_log.py:15; cli.py:127; cli.py:151; agent.py:110; app.py:37; app.py:44-45; app.py:129).
- **C-125** Every log write uses the same relative path `agent_log.txt`, resolved against the running process's working directory; so sessions that run with the same working directory (web sessions in one Streamlit server process, or a terminal chat started from that same folder) write to one file, and a `start_session` from any of them (new session or reset) empties it for the others. A terminal started from a different folder writes to a different file. Inferred from the shared constant and write mode; whether web sessions share one process is framework behavior (see Q-08) (trace_log.py:11; trace_log.py:15; trace_log.py:28; app.py:37).
- **C-126** Terminal, non-Windows: EOF or Ctrl+C at the prompt ends the chat, and the program still prints "[ESC pressed] Closing the program. Goodbye!" (cli.py:98-102; cli.py:140-142).
- **C-127** Terminal, Windows: input is read one character at a time; ESC quits, Enter ends the line, `\x08` deletes the last character, and every other character is added to the line (cli.py:104-120).
- **C-128** Web page: `$` is escaped and `<br>` is replaced by a space in the shown question and answer, but not in error texts (app.py:51-54; app.py:73; app.py:82; app.py:88; app.py:103; app.py:80; app.py:108; app.py:114).
- **C-129** Step text shows a tool result as its first line, cut to 100 characters plus `...`, with `(+N lines)` when there are more lines; both interfaces use this format (cli.py:18; cli.py:32-40; cli.py:43-48; app.py:58).
- **C-130** Web page: a pending example question is taken only when nothing was typed in that run (app.py:141).
- **C-131** `cli.ask()` starts a log session only if there is no history, and no in-scope file calls it (cli.py:81-86).
- **C-132** `_get_ihsg_data` uses the hard-coded ticker `"^JKSE"` rather than the `INDEX_TICKERS` mapping (tools.py:179).
- **C-133** In `get_price_table`, IHSG is recognized only by the exact code `IHSG` after uppercasing (tools.py:337; tools.py:354).
- **C-134** The web page shows the active prompt version and model name in the sidebar caption (app.py:131-133).
- **C-139** `extract_text` returns a string reply content unchanged; for a list content it keeps string blocks and dict blocks with `type == "text"`, joins them, and strips the result, dropping all other blocks; any other content type is converted with `str()` (agent.py:82-95; agent.py:185).
- **C-140** Web page: each turn's step box is labelled "No tool needed" when there was no tool call, otherwise "🔧 N tool call" / "🔧 N tool calls", followed by " · ❌ M failed" when M tool results were marked failed (app.py:61-67; app.py:76; app.py:102).
- **C-141** Terminal: input that is empty after trimming spaces is skipped without sending a request (cli.py:144-146).
- **C-142** `get_price_table` writes a `✅ [ASSET] price (as of last update)` log line for each asset that succeeds (tools.py:361).
- **C-143** When at least one asset succeeded, `get_price_table` writes a `✅ [TOOL OK] table with N assets (M failed)` log line before returning the table (tools.py:384-390).
- **C-144** `start_session` writes a first line `=== NEW SESSION STARTED: YYYY-MM-DD HH:MM:SS ===` with the local time (trace_log.py:14-17).
- **C-145** `log_request`, called at the start of each turn, appends a blank line and a line of 60 `=`, then `REQUEST: <local timestamp>`, `Question: <question>`, and a line of 60 `-` (trace_log.py:20-24; agent.py:158).

## 9. Out of scope / non-goals

- **C-135** No tool queries a crypto data source; the data calls in `tools.py` go only to Frankfurter, yfinance (`^JKSE`), and DuckDuckGo (tools.py:100; tools.py:179; tools.py:204; tools.py:412).
- **C-136** The conversation history is kept in memory only (a module variable for the terminal, `st.session_state` for the web page); the only file writes in the in-scope code are to the log file (agent.py:98; app.py:35; trace_log.py:15; trace_log.py:28).
- **C-137** The web page code has no login or user authentication (app.py:10-143).
- **C-138** None of the four tools places orders or changes external data; they only read rates, prices, and search results (tools.py:100; tools.py:180; tools.py:186; tools.py:204-205; tools.py:412).

## 10. Open questions

- **Q-01** Who the target users are and why the product was built (R6): not stated in code.
- **Q-02** Whether Frankfurter data is daily rather than real-time, and which date it returns when no date is given: only the module docstring says "DAILY" (tools.py:7-9); runtime API behavior is not visible in code.
- **Q-03** Whether Frankfurter actually returns XAU and XAG rates: the code treats them as currencies, but only comments and docstrings say they are supported (tools.py:5-6; tools.py:218-219).
- **Q-04** The time zone of the IHSG "Last Update" timestamp: the code formats the yfinance index without a time-zone label (tools.py:189; tools.py:193); the zone depends on yfinance runtime data.
- **Q-05** What `max_retries=2` retries and when (agent.py:37): library behavior, not visible in code.
- **Q-06** Whether the tool docstrings reach the model as tool descriptions, and whether the model follows them and the system prompt: library and model behavior.
- **Q-07** How `create_agent` handles an exception that escapes a tool (e.g. code outside a tool's `try`), and when a tool message gets status `"error"` (agent.py:124): library behavior.
- **Q-08** Whether Streamlit `st.session_state` is per browser tab, as a comment says (app.py:32): framework runtime behavior.
- **Q-09** How `msvcrt.getwch` reports Ctrl+C, arrow keys, and other special keys on Windows (cli.py:107-120): runtime behavior.
- **Q-10** The supported Python version: only a comment says "Python 3.10" (requirements.txt:1).
- **Q-11** Whether the exact Gemini error texts contain the keywords `_classify_error` looks for (agent.py:67-76): depends on live API errors.
- **Q-12** What the test suite in `tests/` (named in pytest.ini:2) covers: out of scope for this spec.
- **Q-13** Intent of `127.0.0.1` and the stdout reconfigure: only comments explain them (.streamlit/config.toml:2-4; currency_agent.py:20).
- **Q-14** (from C-34) Whether `web_search` returns at most 4 entries, and one line per entry: the code passes `max_results=4` to `DDGS().text` but never caps or checks the result count, and result bodies are not stripped of newlines (tools.py:412; tools.py:416-418); depends on the `ddgs` library and live search results.

## 11. Change log

- C-26 — fixed — the web page shares `format_step` and the per-kind base messages but wraps steps in backticks and words the "not saved" note, the `other` reason, and unexpected errors differently (reviewer row C-26, incorrect)
- C-34 — fixed — output described as one entry per returned result with no cap in code; the "up to 4" part moved to Q-14 (reviewer row C-34, unresolved)
- C-48 — fixed — `to_currency` still appears in the logged call string on the IHSG path, so "not used" was too broad (reviewer row C-48, incorrect)
- C-73 — fixed — the code uses `netloc` (may include port or user info), not the bare host; missing `href` also gives `unknown site` (reviewer row C-73, incorrect)
- C-125 — fixed — the log path is relative, so sharing depends on working directory; the terminal shares the file only when started from the same folder; the shared-process part depends on Streamlit (reviewer row C-125, incorrect). My wording is slightly narrower than the reviewer's: it states the shared-process condition instead of asserting it, per R5.
- Q-14 — added — holds the "up to 4 results" part of C-34 (reviewer row C-34)
- C-139 — added — `extract_text` keeps only text blocks of a list reply (omission, agent.py:82-95)
- C-140 — added — web step-box labels (omission, app.py:61-67)
- C-141 — added — terminal skips blank input (omission, cli.py:145-146)
- C-142 — added — per-asset log line in `get_price_table` (omission, tools.py:361)
- C-143 — added — summary log line in `get_price_table` (omission, tools.py:389)
- C-144 — added — log session header line (omission, trace_log.py:16-17)
- C-145 — added — per-request log block (omission, trace_log.py:21-24)
