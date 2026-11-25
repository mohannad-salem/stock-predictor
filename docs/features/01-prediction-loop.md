# Feature: Prediction loop

Goal: a user can pick an S&P 500 stock and an allowed timeframe, press Predict, and get a direction with a confidence percentage and an honest accuracy notice. After the timeframe ends, they see a yes, no or "no change" result on the prediction's own page. The whole loop runs end to end with a stand-in model behind the fixed model interface, so milestone 2 (`logistic-model`) only swaps what's inside the model module (D-030).

Rules this feature implements: D-002, D-006 to D-010, D-012, D-017, D-019 to D-022, D-024 to D-031, D-033 to D-041. Read docs/architecture.md and docs/data-model.md first.

## Acceptance criteria

Setup and data

1. `docker compose up` starts PostgreSQL, the web process and the worker, and `uv run alembic upgrade head` creates every table in docs/data-model.md (Prediction, ModelVersion, ModelAccuracy, PriceBar, MarketDay).
2. A committed `.env.example` lists every setting named in CLAUDE.md, `.env` is git-ignored, and no key or secret is in the repo.
3. `data/sp500.csv` holds the S&P 500 tickers (Alpaca symbol format) and company names. At startup the app checks it with one call to Alpaca's asset list and logs a warning (never crashes) for any ticker that isn't tradable. With no Alpaca key set, the check is skipped with a log line and the app still starts (D-028).
4. A committed script with a fixed seed generates the bar sample in `tests/data/`. Running it twice gives identical output, and the sample contains a weekend gap, a US market holiday, a 13:00 early-close half-day and a few missing minute bars (D-024).
5. No test calls Alpaca or needs an API key. The price provider is replaced in tests by one that serves the generated sample and a matching calendar (D-023).

Price provider and calendar

6. Only `src/stock_predictor/prices/` makes calls to Alpaca. A test or lint check fails if any other module imports the Alpaca client (D-010).
7. Fetched bars are stored in the PriceBar cache, and a second request for the same bars is served from the cache without calling Alpaca (D-010).
8. All Alpaca calls pass through one shared rate limiter that stays under 200 calls a minute (D-010).
9. The calendar is cached in MarketDay, refreshed at most once a day, and always covers at least the next 2 months. If it can't be fetched and the cache doesn't cover the needed dates, the picker shows "try again" instead of guessing (D-026).

Picker and Predict

10. The stock chooser lists only stocks from `data/sp500.csv`.
11. The timeframe picker offers only 15m, 1h, 4h, 1d, 1w and 1mo (D-006). 15m, 1h and 4h appear only when they would end at or before that day's actual close (16:00 ET, or 13:00 ET on half-days), and never when the market is closed (D-008, D-025, D-041). Tests cover a normal day, a half-day, a weekend, a holiday from the sample, and a prediction ending exactly at the close (for example 15:00 + 1h = 16:00 is offered).
12. Any timeframe whose start and end would use the same close is not offered (for example 1d pressed on a Friday after the close, or on a weekend) (D-022).
13. The server re-checks criteria 10 to 12 on every Predict request and rejects a disallowed stock or timeframe with a clear error, even if the page was bypassed.
14. Predict saves a Prediction with `start_price` and `start_price_time` empty (D-039). The worker later fills them, under the same lock as scoring, once the start bar is at least 15 minutes old: the close of the last full 1-minute bar before the press, or the last close if the market was closed (D-021). The prediction's end time follows D-008 and D-020 (15m/1h/4h after the start; 1d, 1w and 1mo as 24 hours, 7 days and 1 calendar month after the start). `result_due_at` is the end time + 15 minutes (D-010).
15. Over 30 predictions per IP per hour, or 1,000 per hour overall, Predict answers "busy, try again later". Both numbers come from environment variables. The IP is worked out in one place, honours `TRUST_FORWARDED_IP`, and only a salted hash is stored, never the raw IP (D-027, D-033).

Model

16. The model module's only entry point takes the ticker, timeframe, start time and only the full-market bars ending at least 15 minutes before the start time, and returns a direction, a probability and a model version (D-030, D-040).
17. The stand-in model always returns "up" with probability 0.50 and model version `stand-in-1` (D-036). It is registered as a ModelVersion with `accuracy_source` = generated_sample and one ModelAccuracy row per timeframe, measured on the generated sample (D-031).
18. A no-look-ahead test proves that the bars passed to the model for a prediction never include any bar ending less than 15 minutes before the prediction's start time (D-011, D-030, D-040).

Pages and notice

19. After Predict, the user sees the direction, the confidence percentage, the expected result time in ET, and the message that the result appears about 15 minutes after the timeframe ends (D-010, D-034).
20. Every prediction and result shows the notice "Experimental model, not financial advice. Tested accuracy for <timeframe>: <n>% (coin flip is 50%)", looked up by model version and timeframe. When the version's accuracy was measured on the generated sample, it also says "measured on test data, not real prices". Every page has a "not financial advice" footer line (D-019, D-031).
21. Each prediction has a page at `/p/<id>` that shows "pending until <time> ET" or the yes, no or "no change" result with the start and end prices. Until the worker has filled the start price, the page shows "start price pending" in its place. The page works for anyone with the link (D-012, D-037, D-039).
22. The home page lists the user's own predictions from browser storage, each linking to its `/p/<id>` page (D-037).

Scoring worker

23. The worker scores a prediction only once its end-time bar is at least 15 minutes old, using full-market bars. The end price is the last full-market price at or before the end time (D-010, D-020).
24. The outcome is "yes" if the end price is on the predicted side of the start price, "no" if it is on the other side, and "no change" if it is equal. "No change" results are left out of accuracy (D-009, D-022).
25. Running two scoring passes at the same time never scores a prediction twice. A test runs them concurrently against PostgreSQL and checks this (D-017).
26. When the worker starts after being stopped, it first fills any missing start prices, then scores every overdue prediction, using the correct start and end prices from the past, not the prices at catch-up time. A test covers this catch-up, including a prediction made just before the worker stopped (D-029, D-039).
27. `/health` returns JSON including `overdue_count`: the number of predictions more than 10 minutes past their `result_due_at`. A test shows it rises when the worker is stopped and falls back to 0 after catch-up (D-017, D-038).

Quality

28. `uv run pytest`, `uv run mypy src`, `uv run ruff check .` and `uv run ruff format --check .` all pass.

## Out of scope

- The real logistic regression model, the `train` command and real-data model versions (milestone 2, `logistic-model`).
- Public hosting (open items in docs/architecture.md).
- User accounts, price-range predictions, a stats page, stocks outside the S&P 500.
