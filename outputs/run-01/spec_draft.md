# Spec — Agentic Financial Monitoring Assistant
- Input repo: input/demo-a/ (Demo A)
- Commit: 6caaf96 (tag demo-v1)
- Skill version: v1
- Run ID: run-01
- Date: 2026-11-12
- Stage: draft

---

## 1. Purpose and target user

- **C-01** The product is a conversational assistant ("Asset Price Assistant") that answers questions about currency exchange rates and asset prices: fiat currencies, gold, silver, and the IHSG stock index.
- **C-02** The target user is someone in Indonesia who tracks prices in rupiah (IDR).
- **C-03** The product has two interfaces: a terminal chat and a Streamlit web page.
- **C-04** Both interfaces use the same agent, the same tools, and the same system prompt.

## 2. User stories / features

- **C-05** As a user, I can ask for a "market summary" or "price update" and get one table with several assets, their price, % change, and last update time.
- **C-06** The default assets in the market summary table are USD, SGD, TWD, XAU (gold), and IHSG.
- **C-07** As a user, I can ask for the latest exchange rate of one currency pair.
- **C-08** As a user, I can ask for the average exchange rate of a pair over a date range, and I also get the number of days, the minimum, and the maximum.
- **C-09** As a user, I can ask for the average IHSG closing value over a date range.
- **C-10** As a user, I can ask why a price moved; the agent searches the web and names the source of each reason.
- **C-11** As a user, I can ask follow-up questions ("why did it go up?") and the agent uses the context of earlier turns in the same conversation.
- **C-12** In the terminal, I can type `reset` or `clear` to start a new conversation.
- **C-13** On the web page, I can click a "Reset conversation" button in the sidebar to start a new conversation.
- **C-14** On the web page, the sidebar shows four example questions as buttons that ask the question when clicked.
- **C-15** While the agent works, each tool call and tool result is shown live (printed lines in the terminal, a status box on the web page).
- **C-16** In the terminal, I can quit by pressing ESC or by typing `exit`, `quit`, `bye`, or `esc`.
- **C-17** If a question is not about exchange rates or asset prices, the agent answers normally without calling a tool.

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
| **C-30** | `get_price_table` | `assets`: comma-separated codes, default `"USD,SGD,TWD,XAU,IHSG"` | Markdown table `Asset \| Price \| Change (vs previous day) \| Last Update`; currencies in Rp, gold in USD/oz plus Rp/g and Rp/0.01 g, IHSG in points | Frankfurter API v2; yfinance for IHSG | One failed asset becomes a row marked `FAILED`; the whole tool returns `ERROR:` only if every asset fails |
| **C-31** | `web_search` | `query` | Up to 4 results, each as `- [site] title: body` | DuckDuckGo (via `ddgs`) | Returns text starting with `ERROR:` if there are no results or the search fails |

- **C-32** All tool failures are returned to the agent as text starting with `ERROR:`; tools do not raise exceptions to the agent.
- **C-33** Cross rates are computed from Frankfurter EUR-based rates as `rate_to / rate_from`.
- **C-34** The % change in the price table is the change from the previous available day to the latest day.
- **C-35** For gold, the % change is computed on the USD-per-ounce price.
- **C-36** Rupiah per gram of gold is computed as IDR per troy ounce divided by 31.1034768.
- **C-37** In the price table, a positive or zero change is shown with 🔺 and a negative change with 🔻.

## 5. External data sources and dependencies

- **C-38** Currency and precious-metal rates come from the Frankfurter API v2 at `https://api.frankfurter.dev/v2`, which needs no API key.
- **C-39** Frankfurter data is daily, not real-time.
- **C-40** IHSG data comes from yfinance with ticker `^JKSE`, using 1-minute intraday data.
- **C-41** Web search uses DuckDuckGo through the `ddgs` library.
- **C-42** The language model is Google Gemini `gemini-3.5-flash-lite`, used through `langchain-google-genai`.
- **C-43** Dependencies are pinned to exact versions in `requirements.txt`, tested on Python 3.10.
- **C-44** Streamlit is only needed for the web page; the terminal chat works without it.

## 6. Configuration

- **C-45** `GOOGLE_API_KEY` is required and is read from a `.env` file via `python-dotenv`.
- **C-46** If `GOOGLE_API_KEY` is missing or still equals `your-key-here`, the program stops with a message telling the user to copy `.env.example` to `.env`.
- **C-47** The `PROMPT_VERSION` environment variable selects the system prompt; the default is `v2`.
- **C-48** An unknown `PROMPT_VERSION` value stops the program with a message listing the valid versions.
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
- **C-59** When a turn fails, the conversation history is left unchanged, and the user is told the question was not saved.
- **C-60** In the terminal, an `other` error also shows a one-line summary of the reason.
- **C-61** Pressing Ctrl+C during a turn in the terminal cancels that question without closing the chat.
- **C-62** Unexpected exceptions are caught and logged, so neither the terminal chat nor the web page ever crashes.
- **C-63** If the model's final reply contains no text, the user sees a fallback message asking them to try again or rephrase.
- **C-64** If the agent stops without a final AI message, the turn fails with an `other` error.
- **C-65** Every tool failure is written to the log with `[TOOL FAILED]`.
- **C-66** The system prompt tells the agent to try `web_search` when a structured data tool fails.

## 8. Edge cases and known limits

- **C-67** Outside market hours, when 1-minute IHSG data is empty, the price table uses the latest daily close instead.
- **C-68** An IHSG date range with no trading days (weekend or holiday) returns an `ERROR:` result that explains the market may have been closed.
- **C-69** `get_exchange_rate` does not support crypto or stock indexes.
- **C-70** An unknown asset code in the price table produces a `FAILED` row instead of breaking the table.
- **C-71** The log file is overwritten when a new session starts, so only the latest session is kept.
- **C-72** On the web page, all browser tabs write to the same log file, and starting a conversation in any tab clears it.
- **C-73** Quitting with ESC works only on Windows; on other systems the user quits with Ctrl+C or end-of-input.
- **C-74** Today's date is put into the system prompt once, when the program starts, so a process running past midnight keeps the old date.
- **C-75** The web page escapes `$` and removes `<br>` in text before showing it as Markdown.
- **C-76** If gold has no IDR rate on the latest USD date, the IDR price from the latest IDR date is used.

## 9. Out of scope / non-goals

- **C-77** There are no user accounts or authentication.
- **C-78** The product does not give financial advice.
- **C-79** Conversations are not saved across program restarts.
- **C-80** There is no structured data source for crypto; crypto prices are only available through web search.

## 10. Open questions

- **Q-01** Is `gemini-3.5-flash-lite` an available model name at runtime? The code can't confirm it.
- **Q-02** Does Frankfurter actually return XAU and XAG rates for every date? This depends on the live API.
- **Q-03** Is the IDR focus a product decision (target market) or just the developer's own use case?
- **Q-04** What happens to the log file when two browser tabs write to it at the same time?
- **Q-05** Do the keyword lists in the error classifier match the real error messages Gemini returns?

## 11. Change log

— (draft)
