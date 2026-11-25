# Architecture

Keep this to 1 or 2 pages. Test: if two workers read this and built separate features, would their work fit together?

## Stack

| Part | Choice | Why |
| --- | --- | --- |
| Web server and API | FastAPI (Python) | Small, fast, with type-checked inputs. (D-014) |
| Pages | Jinja2 templates plus htmx | Server-made HTML; htmx updates parts of a page without a JavaScript framework. (D-014) |
| Database | PostgreSQL via SQLAlchemy, Alembic migrations | Same database locally, in tests and hosted; safe with two writers. (D-015) |
| Tooling | uv, pytest, ruff, mypy | Locked dependencies, tests, lint and format, type checks. (D-016) |
| Price data | Alpaca free market-data plan | Free, official API with minute-level bars and about 200 calls a minute. (D-010) |
| Prediction model | Logistic regression (scikit-learn), one model per timeframe; a stand-in model in milestone 1 | Gives real probabilities for the confidence percentage; simple and hard to fool yourself with. (D-011, D-030, D-036) |
| Local running | Docker Compose (PostgreSQL, web, worker) | One command starts everything; makes a later move to a host small. (D-018) |

## Main pieces and how they connect

One codebase, two processes, one database (D-013, D-017):

```
 Browser (Jinja2 pages + htmx, "my predictions" in browser storage)
    |
    v
 Web process (FastAPI) --------------------+
    |  picker rules, Predict, /p/<id>,     |
    |  /health, abuse limits               |
    v                                      v
 model/ (model module) <-- prices/ (price provider + calendar) --> Alpaca
    ^                          ^
    |                          |
 Worker process (scoring) -----+
    |
    v
 PostgreSQL (predictions, model versions, accuracy, price cache, calendar cache)
```

Folders are in CLAUDE.md under Conventions (D-035). In short:

- `prices/`: the only code that talks to Alpaca: bars, calendar, asset list. It applies the shared rate limit and writes the PriceBar and MarketDay caches (D-010, D-026, D-028).
- `model/`: the model module with the fixed interface below. Milestone 1 holds the stand-in (D-036); milestone 2 adds logistic regression and the `train` command (D-023).
- `scoring/`: the plain scoring function: filling start prices, end-time rules, yes/no/no change, the database lock (D-017, D-020, D-022, D-039).
- `worker.py`: on start, catches up on overdue predictions, then fills start prices and scores predictions in a loop (D-029, D-039).
- `web/` and `templates/`: home page (chooser, picker, Predict, "my predictions"), `/p/<id>`, `/health` (D-037, D-038).
- `db/` and `migrations/`: SQLAlchemy tables and Alembic migrations, matching docs/data-model.md.

## Rules every feature must follow

- All price fetching goes through one price provider module. No other code calls Alpaca directly. (D-010)
- Start and end prices for a prediction come from full-market bars that are at least 15 minutes old. Results appear about 15 minutes after a timeframe ends, and the app tells users so. (D-010)
- Price data is cached and shared across users, so the app stays under Alpaca's limit of about 200 calls a minute. (D-010)
- The market calendar (trading days, holidays, half-days) comes from Alpaca through the price provider module. It's cached, refreshed once a day, and always covers at least the next 2 months. If it can't be fetched, the picker shows "try again" rather than guessing. (D-026)
- The stock list is a hand-updated file in the repo (ticker in Alpaca's symbol format, company name). At startup, one call to Alpaca's asset list checks it, warning in the logs only. The check is skipped with a log line when no Alpaca key is set. (D-028)
- "Today's close" means the actual close: 16:00 New York time normally, 13:00 on half-days. (D-025)
- API keys and secrets come from environment variables only. The repo is public. (D-010)
- The prediction model sits behind its own model module, so the model can be replaced without other changes. (D-011) Its interface is fixed (D-030):
  - Input: ticker, timeframe, start time, and only full-market bars ending at least 15 minutes before the start time. Training in milestone 2 uses the same 15-minute gap. (D-040)
  - Output: direction (up or down), probability, model version.
- Tested accuracy is looked up by model version and timeframe from the model-versions table. If a version's accuracy was measured on the generated sample, the notice says "measured on test data, not real prices". (D-031)
- No look-ahead: training and testing split by time (older bars train, newer bars test), and a test proves a prediction's inputs only use bars ending at least 15 minutes before its start time. (D-011, D-040)
- A prediction is saved with its start price empty. The worker fills `start_price` and `start_price_time` once that bar is at least 15 minutes old, under the same lock as scoring. Until then the prediction page shows "start price pending". (D-039)
- Intraday timeframes may end at or before the day's actual close. (D-008, D-041)
- No user accounts. Every prediction has its own ID and shareable link. The user's own list of predictions is kept in their browser. Abuse limits work per IP address. (D-012)
- Abuse limits: 30 predictions per IP per hour and 1,000 per hour overall, both set by environment variables. Over the limit, Predict says "busy, try again later". The user's IP is worked out in one place only, with a setting for whether to trust the host's forwarded-IP header. (D-027)
- Scoring logic is a plain function. A database lock ensures a prediction is never scored twice. (D-017)
- When the worker starts, it first fills any missing start prices, then scores every overdue prediction, using the correct start and end prices from the past, not the prices at catch-up time. A test covers this. (D-029, D-039)
- The health URL `/health` returns JSON including `overdue_count`: predictions more than 10 minutes past their expected result time. (D-017, D-038)
- Raw IP addresses are never stored. Only a salted hash (`requester_ip_hash`, salt from `IP_HASH_SALT`) is kept, and limits are counted from it. (D-033)
- Times are stored in UTC and shown in New York time, labelled "ET". (D-034)
- The tested accuracy per timeframe is recorded against the 50% coin-flip baseline. (D-011)
- Models are trained by a manual `train` command and saved as model versions (model, tested accuracy, training date) in the database. The app uses the newest version, and each prediction records its model version. (D-023)
- Training goes through the same price provider rate limit and cache as everything else and must never use up the calls scoring needs. Slow training is fine. (D-023)
- Tests never call Alpaca or need an API key. Training and model tests use a small saved sample of bars in the repo. (D-023)
- The test sample is made-up bars from a committed generator script with a fixed seed. No real provider data goes in the repo. The sample includes a weekend gap, a US market holiday, an early-close half-day (13:00 close) and a few missing minute bars. (D-024)

## Open items

- Before public hosting: Mohannad confirms in Alpaca's market data agreement (or with Alpaca support) that showing the data or results to public users is allowed. If it isn't, switch providers by changing the price provider module. (D-010)
- Revisit the calendar-time ending rule for 1d, 1w and 1mo (weekend and holiday endings barely move the price). (D-020)
- Choose the public host (for example a managed platform or a VPS) after the Alpaca item above is resolved. At that point, set the forwarded-IP trust setting correctly for that host. (D-018, D-027)

## Environments

Version one runs locally only (D-018). Docker Compose starts PostgreSQL, the web app and the worker on Mohannad's machine. There is no public deployment yet.
