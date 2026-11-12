# Spec — Agentic Financial Monitoring Assistant
- Input repo: input/demo-a/ (Demo A)
- Commit: 6caaf96 (tag demo-v1)
- Skill version: v1
- Run ID: run-01
- Date: 2026-11-12
- Stage: revised

---

## 1. Purpose and target user

- **C-01** The product is a conversational assistant ("Asset Price Assistant") that answers questions about currency exchange rates and asset prices: fiat currencies, gold, silver, and the IHSG stock index.
- ~~**C-02** The target user is someone in Indonesia who tracks prices in rupiah (IDR).~~ → moved to Q-06
- **C-03** The product has two interfaces: a terminal chat and a Streamlit web page.
- **C-04** Both interfaces use the same agent, the same tools, and the same system prompt.

## 2. User stories / features

- **C-05** For a "market summary" or "price update" request, the system prompt tells the agent to use `get_price_table`, which returns one table with several assets, their price, % change, and last update time.
- **C-06** The default assets in the market summary table are USD, SGD, TWD, XAU (gold), and IHSG.
- **C-07** As a user, I can ask for the latest exchange rate of one currency pair.
- **C-08** As a user, I can ask for the average exchange rate of a pair over a date range, and I also get the number of days, the minimum, and the maximum.
- **C-09** As a user, I can ask for the average IHSG closing value over a date range.
- **C-10** In prompt `v2` (the default), the system prompt tells the agent to use `web_search` for questions about why a price moved, to give only reasons found in the search results, and to name the source of each reason. Prompt `v1` has no such rule.
- **C-81** Prompt `v2` tells the agent to get the rate on one past date by calling `get_average_exchange_rate` with `start_date` and `end_date` both set to that date.
- **C-11** As a user, I can ask follow-up questions ("why did it go up?") and the agent uses the context of earlier turns in the same conversation.
- **C-12** In the terminal, I can type `reset` or `clear` to start a new conversation.
- **C-13** On the web page, I can click a "Reset conversation" button in the sidebar to start a new conversation.
- **C-14** On the web page, the sidebar shows four example questions as buttons that ask the question when clicked.
- **C-15** While the agent works, each tool call and tool result is shown live (printed lines in the terminal, a status box on the web page).
- **C-84** On the web page, each turn's tool steps are labeled "No tool needed" or "🔧 N tool call(s)", with " · ❌ N failed" added when any tool step failed.
- **C-85** The web page sidebar shows the active prompt version and the model name.
- **C-16** In the terminal, I can quit by typing `exit`, `quit`, `bye`, or `esc` (not case-sensitive).
- **C-17** The system prompt tells the agent to answer questions unrelated to exchange rates or asset prices without calling a tool; no code enforces this.

## 3. Components (files and their responsibility)

- **C-18** `currency_agent.py` is the entry point for the terminal chat; it calls `cli.chat()`.
- **C-19** `currency_agent.py` reconfigures stdout to UTF-8 with `errors="replace"` before starting.
- **C-20** `cli.py` runs the terminal input loop and maps agent errors to user-facing messages.
- **C-21** `agent.py` creates the Gemini model and a LangChain `create_agent` agent with the 4 tools.
- **C-22** `agent.py` keeps the conversation history for the terminal session.
- **C-23** `tools.py` defines the 4 tools and the data-fetching helpers behind them.
- **C-24** `prompts.py` holds two system prompt versions, `v1` and `v2`.
- **C-25** `trace_log.py` writes a text log of each session to `agent_log.txt`.
- **C-26** `app.py` is the Streamlit web page.
- **C-27** The web page keeps a separate conversation for each browser tab (in Streamlit session state).

## 4. Tools

| Claim | Name | Inputs | Output | Data source | Failure behavior |
|---|---|---|---|---|---|
| **C-28** | `get_exchange_rate` | `from_currency`, `to_currency` | Text `1 FROM = rate TO (as of date)` for the latest available date | Frankfurter API v2, cross rate via EUR | Returns text starting with `ERROR:` if no rate is found or the request fails |
| **C-29** | `get_average_exchange_rate` | `from_currency`, `to_currency`, `start_date`, `end_date` (YYYY-MM-DD) | Text with the average rate, number of days of data, min, and max; for `from_currency='IHSG'`, the average daily close in index points (`to_currency` ignored) | Frankfurter API v2; yfinance for IHSG | Returns text starting with `ERROR:` if there is no data in the range or the request fails |
| **C-30** | `get_price_table` | `assets`: comma-separated codes, default `"USD,SGD,TWD,XAU,IHSG"` | Markdown table `Asset \| Price \| Change (vs previous day) \| Last Update`; currencies in Rp, gold in USD/oz plus Rp/g and Rp/0.01 g, IHSG in points | Frankfurter API v2; yfinance for IHSG | A failed asset becomes a row with `-` for price and change and `FAILED: <reason>` in Last Update; the tool returns `ERROR:` if every asset fails or if no asset codes are given |
| **C-31** | `web_search` | `query` | Up to 4 results, each as `- [site] title: body` | DuckDuckGo (via `ddgs`) | Returns text starting with `ERROR:` if there are no results or the search fails |

- **C-32** Failures caught inside a tool's `try` block are returned to the agent as text starting with `ERROR:` instead of being raised, except that a partial `get_price_table` failure is shown as `FAILED` rows inside a normal table.
- **C-83** A tool step counts as failed when its result starts with `ERROR:` or the tool message has status `"error"`.
- **C-33** Cross rates are computed from Frankfurter EUR-based rates as `rate_to / rate_from`.
- **C-34** The % change in the price table is the change from the previous available day to the latest day.
- **C-35** For gold, the % change is computed on the USD-per-ounce price.
- **C-36** Rupiah per gram of gold is computed as IDR per troy ounce divided by 31.1034768.
- **C-37** In the price table, a positive or zero change is shown with 🔺 and a negative change with 🔻.
- **C-82** In the price table, Last Update for Frankfurter-based assets (currencies and gold) is the rate date plus the local time of the check, formatted `<date> • checked HH:MM`.

## 5. External data sources and dependencies

- **C-38** Frankfurter requests go to `https://api.frankfurter.dev/v2` and send no API key or other credentials.
- ~~**C-39** Frankfurter data is daily, not real-time.~~ → moved to Q-07
- **C-40** IHSG data comes from yfinance with ticker `^JKSE`.
- **C-88** In the price table, IHSG's current price is the latest close from 1-minute intraday data (fallback: C-67).
- **C-89** In the price table, IHSG's % change is measured against the previous daily close from 5 days of daily history.
- **C-90** IHSG date-range averages use daily closing values.
- **C-41** Web search uses DuckDuckGo through the `ddgs` library.
- **C-42** The language model is Google Gemini `gemini-3.5-flash-lite`, used through `langchain-google-genai`.
- **C-43** Dependencies are pinned to exact versions (`==`) in `requirements.txt`.
- **C-44** Streamlit is only needed for the web page; the terminal chat works without it.
- **C-86** The `yfinance` logger is set to `CRITICAL`, which hides its lower-level log messages.

## 6. Configuration

- **C-45** `GOOGLE_API_KEY` is required and is read from the process environment; `python-dotenv` first loads a `.env` file into the environment if one is present.
- **C-46** If `GOOGLE_API_KEY` is missing, empty after trimming, or equals `your-key-here`, the terminal program exits with a message telling the user to copy `.env.example` to `.env`.
- **C-47** The `PROMPT_VERSION` environment variable selects the system prompt; the default is `v2`.
- **C-48** An unknown `PROMPT_VERSION` value makes the terminal program exit with a message listing the valid versions.
- **C-87** On the web page, the startup errors from C-46 and C-48 are caught when `agent` is imported; the page shows the message as an error and stops the page script.
- **C-49** Each Gemini request has a 30-second timeout.
- **C-50** Gemini requests are retried at most 2 times.
- **C-51** No temperature is set for the model.
- **C-52** Frankfurter HTTP requests use a 15-second timeout.
- **C-53** yfinance requests use a 10-second timeout.
- **C-54** The Streamlit server listens only on `127.0.0.1`.
- **C-55** Streamlit usage statistics are disabled.
- **C-56** The log file is `agent_log.txt` in the current working directory.

## 7. Error handling

- **C-57** Agent errors are classified into four kinds, `quota`, `timeout`, `auth`, and `other`, by matching keywords in the error type and message.
- **C-58** Each error kind has a fixed user-facing message, and the terminal and web page show the same messages.
- **C-59** When a turn fails, the conversation history is left unchanged.
- **C-91** The notice "This question was not saved to the conversation" is shown for agent errors (`AgentRunError`) in both interfaces and for Ctrl+C in the terminal, but not for other unexpected exceptions.
- **C-60** In the terminal, an `other` error also shows a one-line summary of the reason.
- **C-61** Pressing Ctrl+C during a turn in the terminal cancels that question without closing the chat.
- **C-62** Unexpected exceptions raised while answering a question are caught, logged, and shown as "Unexpected error: …" in both interfaces.
- **C-63** If the model's final reply contains no text, the user sees a fallback message asking them to try again or rephrase.
- **C-64** If the agent stops without a final AI message, the turn fails with an `other` error.
- **C-65** Tool failures handled inside the tools are written to the log with `[TOOL FAILED]`.
- **C-66** The system prompt tells the agent to try `web_search` when a structured data tool fails.

## 8. Edge cases and known limits

- **C-67** Outside market hours, when 1-minute IHSG data is empty, the price table uses the latest daily close instead.
- **C-68** An IHSG date range with no trading days (weekend or holiday) returns an `ERROR:` result that explains the market may have been closed.
- **C-69** `get_exchange_rate` has no special handling for crypto or stock-index codes; any code is sent to Frankfurter.
- **C-70** An unknown asset code in the price table produces a `FAILED` row instead of breaking the table.
- **C-71** The log file is overwritten when a new session starts, so only the latest session is kept.
- **C-72** On the web page, all browser tabs write to the same log file, and starting a conversation in any tab clears it.
- **C-73** Quitting with ESC works only on Windows; on other systems the user quits with Ctrl+C or end-of-input.
- **C-74** Today's date is put into the system prompt once, when the program starts, so a process running past midnight keeps the old date.
- **C-75** The web page escapes `$` and removes `<br>` in text before showing it as Markdown.
- **C-76** If gold has no IDR rate on the latest USD date, the IDR price from the latest IDR date is used.

## 9. Out of scope / non-goals

- **C-77** There are no user accounts or authentication.
- **C-78** The web page sidebar states "Not financial advice."
- **C-79** Conversations are not saved across program restarts.
- **C-80** The code has no dedicated data source for crypto.

## 10. Open questions

- **Q-01** Is `gemini-3.5-flash-lite` an available model name at runtime? The code can't confirm it.
- **Q-02** Does Frankfurter actually return XAU and XAG rates for every date? This depends on the live API.
- **Q-03** Is the IDR focus a product decision (target market) or just the developer's own use case?
- **Q-04** What happens to the log file when two browser tabs write to it at the same time?
- **Q-05** Do the keyword lists in the error classifier match the real error messages Gemini returns?
- **Q-06** (was C-02) Who is the target user? IDR prices, an IHSG index, and a rupiah caption suggest someone in Indonesia, but the code can't confirm who the users are.
- **Q-07** (was C-39) Is Frankfurter data daily rather than real-time? Only a code comment says so; it needs the provider's docs or live responses.
- **Q-08** (from C-43) Was the product tested on Python 3.10? Only a comment in `requirements.txt` says so.
- **Q-09** (from C-69, C-80) Does Frankfurter reject crypto and stock-index codes, so that crypto prices are only available through web search? Needs live API responses.
- **Q-10** (from C-05, C-10, C-17, C-78) Does the model follow the system prompt's tool-choice rules and avoid giving financial advice? Needs runtime transcripts.
- **Q-11** (from C-65) Are tool errors produced by the framework (tool message status `"error"`) logged anywhere? That depends on LangChain behavior outside the code in scope.

## 11. Change log

Round-1 review: `outputs/run-01/checks_round1.md` (60 verified / 11 incorrect / 9 unresolved by row count).

- C-02 — moved to Q-06 — claim about users; code can't confirm (unresolved row)
- C-05 — fixed — stated as a prompt instruction plus tool output, not as what the user always gets (unresolved row)
- C-10 — fixed — rule exists only in prompt v2 (incorrect row)
- C-16 — fixed — typed commands only; ESC-key quitting is Windows-only and already covered by C-73 (incorrect row)
- C-17 — fixed — stated as a prompt instruction with no code enforcement (unresolved row)
- C-30 — fixed — failure behavior: `-`/`FAILED: <reason>` row format, plus `ERROR:` when no asset codes are given (incorrect row)
- C-32 — fixed — narrowed to failures caught in `try` blocks; partial price-table failures are not `ERROR:` (incorrect row). I left out the reviewer's point that `log()` outside `try` is unprotected: it is true, but it is a separate fact about log-file errors, not a tool behavior the spec relies on.
- C-38 — fixed — kept only what the code shows (URL, no key sent); whether the service accepts keyless requests is an API property (unresolved row)
- C-39 — moved to Q-07 — property of the external API, supported only by a comment (unresolved row)
- C-40 — fixed — narrowed to the ticker; intraday vs daily details split into C-88, C-89, C-90 (incorrect row)
- C-43 — fixed — kept the exact pins; "tested on Python 3.10" moved to Q-08 (unresolved row)
- C-45 — fixed — key is read from the environment; `.env` is optional (incorrect row)
- C-46 — fixed — scoped to the terminal; added "empty after trimming"; web behavior split into C-87 (incorrect row)
- C-48 — fixed — scoped to the terminal; web behavior in C-87 (incorrect row)
- C-59 — fixed — kept the history fact; the "not saved" notice split into C-91 with its real scope (incorrect row)
- C-62 — fixed — removed "never crashes"; scoped to exceptions while answering a question (incorrect row)
- C-65 — fixed — scoped to failures handled inside the tools; framework errors noted in Q-11 (incorrect row)
- C-69 — fixed — stated as the code fact (no special handling); real API outcome moved to Q-09 (unresolved row)
- C-78 — fixed — stated as the UI disclaimer only; model behavior moved to Q-10 (unresolved row)
- C-80 — fixed — kept "no dedicated crypto source"; "only via web search" moved to Q-09 (unresolved row)
- C-81 — added — prompt v2 past-date rule (omission 1)
- C-82 — added — Last Update format for Frankfurter assets (omission 2)
- C-83 — added — how a tool step is counted as failed (omission 3)
- C-84 — added — web page turn label (omission 3)
- C-85 — added — sidebar shows prompt version and model name (omission 4)
- C-86 — added — yfinance logger set to CRITICAL (omission 5)
- C-87 — added — split from C-46/C-48: web page catches startup errors
- C-88 — added — split from C-40: IHSG price from 1-minute intraday data
- C-89 — added — split from C-40: IHSG % change vs previous daily close
- C-90 — added — split from C-40: IHSG averages use daily closes
- C-91 — added — split from C-59: scope of the "not saved" notice
