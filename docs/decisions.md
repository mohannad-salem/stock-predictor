# Decisions log

Every agent reads this before making a design choice. Only the design partner edits this file, and only to record a decision Mohannad made.

It stays short on purpose, because every thread and every review reads it:
- **Rules in force:** one line per decision that still applies, newest first. This is what to follow.
- **Full entries for the current milestone:** the question, options and reasoning for decisions made since the last milestone was merged.

When a milestone is finished, the design partner's "milestone close" session moves its full entries to `docs/decisions-archive/<slug>.md` and keeps their one-line rules here. D-numbers never change. Open an archive file only when you need an old decision's reasoning or the options that were turned down.

## Rules in force

- D-041: Intraday timeframes may end exactly at the close: they are offered if they end at or before that day's actual close (15:00 + 1h = 16:00 is allowed). Updates D-008. (project-design)
- D-040: The model's input is only full-market bars ending at least 15 minutes before the prediction's start time; milestone 2's training uses the same 15-minute gap, and the no-look-ahead test checks this rule. Updates D-011 and D-030. (project-design)
- D-039: A prediction is saved with `start_price` empty; the worker fills `start_price` and `start_price_time` (per D-021's definition) once that bar is at least 15 minutes old, under the same lock as scoring; the prediction page shows "start price pending" until then. Updates D-021. (project-design)
- D-038: `/health` returns JSON including `overdue_count`, the number of predictions more than 10 minutes past their expected result time. (project-design)
- D-037: Each prediction has a page at `/p/<id>` showing "pending until <time> ET" or its yes/no/no-change result; the home page lists the user's predictions from browser storage. (project-design)
- D-036: The milestone 1 stand-in model always predicts "up" with probability 0.50 and is registered as model version `stand-in-1`, with accuracy measured on the generated sample. (project-design)
- D-035: Folder layout, commands and environment settings are as written in CLAUDE.md; only `src/stock_predictor/prices/` talks to Alpaca; settings come from environment variables listed in a committed `.env.example`, and `.env` is git-ignored. (project-design)
- D-034: All times are stored in UTC and shown to users in New York time, labelled "ET". (project-design)
- D-033: Raw IP addresses are never stored; abuse limits count predictions by a salted hash of the IP (`requester_ip_hash`), with the salt from an environment variable. (project-design)
- D-032: The product's problem statement is "an honest experiment": predict, wait, and publicly show whether it was right, with accuracy against a coin flip always visible; the GitHub readme adds the "open-source example of an honest prediction loop" line. (project-design)
- D-031: The model-versions table exists from milestone 1 (ID, training date, tested accuracy per timeframe, and where accuracy was measured: generated sample or real data); the stand-in model is registered as a model version; the notice looks up accuracy by model version and timeframe and says "measured on test data, not real prices" when it was measured on the sample. (project-design)
- D-030: Version one is built in two milestones: `prediction-loop` (the whole loop end to end with a stand-in model) then `logistic-model` (real model, train command, model versions). The model interface is fixed now: input is ticker, timeframe, start time and only full-market bars ending at least 15 minutes before the start time (see D-040); output is direction, probability and model version. The no-look-ahead test is built in milestone 1 against this interface. (project-design)
- D-029: Version one passes if 99% of predictions are scored within 10 minutes of their expected result time (counting only time the worker was running), with none scored twice or with look-ahead; model accuracy against the coin flip is reported, not a pass bar. On start, the worker first fills any missing start prices and then scores every overdue prediction, using the correct start and end prices from the past, and a test covers this catch-up. (project-design)
- D-028: The S&P 500 list is a hand-updated file in the repo (ticker in Alpaca's symbol format, company name); at startup one call to Alpaca's asset list checks every ticker is tradable, only warning in the logs, and the check is skipped with a log line when no Alpaca key is set. (project-design)
- D-027: Abuse limits are 30 predictions per IP address per hour plus a global cap of 1,000 predictions per hour, both set by environment variables; the user's IP address is worked out in one place, with a setting for whether to trust the host's forwarded-IP header. (project-design)
- D-026: The market calendar (trading days, holidays, half-days) comes from Alpaca's calendar endpoint through the price provider module; it is cached, refreshed once a day, and always covers at least the next 2 months. (project-design)
- D-025: "Today's close" for intraday timeframes means the actual close that day: 16:00 New York time normally, 13:00 on early-close half-days. Updates D-008. (project-design)
- D-024: The test sample of bars is generated by a committed script with a fixed seed (no real provider data in the repo), and it includes a weekend gap, a US market holiday, an early-close half-day (13:00 close) and a few missing minute bars. (project-design)
- D-023: Models are trained by a manual train command that saves each model with its tested accuracy and training date as a model version in the database; the app uses the newest version and each prediction records its model version; training shares the app's Alpaca rate limit and cache and can never starve scoring; tests never call Alpaca or need a key and use a small saved sample of bars in the repo. (project-design)
- D-022: The picker blocks any timeframe whose start and end would use the same close; any flat result that still happens is shown as "no change" and left out of accuracy. (project-design)
- D-021: A prediction's start price is the close of the last full 1-minute bar before Predict was pressed, or the last close if the market is closed; it is filled in later by the worker (see D-039). (project-design)
- D-020: 1d, 1w and 1mo predictions end in calendar time (24 hours, 7 days, 1 calendar month after the start) and are judged on the last full-market price at or before that moment; this rule is to be revisited later. (project-design)
- D-019: Every page has a "not financial advice" footer line, and every prediction and result shows a notice with the model's tested accuracy for that timeframe compared with the 50% coin flip; no separate stats page in version one. (project-design)
- D-018: Version one runs locally only, using Docker Compose to start PostgreSQL, the web app and the worker; the public host is chosen later, after the Alpaca open item (D-010) is resolved. (project-design)
- D-017: Scoring runs in a separate worker process from the same codebase; the scoring logic is a plain function, a database lock ensures a prediction is never scored twice, and the health URL reports how many predictions are overdue by more than 10 minutes past their expected result time. (project-design)
- D-016: Python tooling is uv (packages and lock file), pytest (tests), ruff (lint and format) and mypy (type checking). (project-design)
- D-015: Data is stored in PostgreSQL, accessed through SQLAlchemy with Alembic migrations, in every environment (local, tests, hosted). (project-design)
- D-014: The web app uses FastAPI, with server-rendered HTML pages (Jinja2 templates) and htmx for partial page updates; no JavaScript framework. (project-design)
- D-013: The app is one web app: a single server serves the pages and the API, and the scoring background job lives in the same codebase. (project-design)
- D-012: No user accounts in version one: each prediction has its own ID and shareable link, the browser remembers the user's own predictions, and abuse limits work per IP address. Optional accounts come after version one. (project-design)
- D-011: Predictions come from a logistic regression model (one per timeframe) behind its own model module; training and testing split by time, a test proves no look-ahead (stricter 15-minute rule in D-040), and tested accuracy per timeframe is recorded against the 50% coin-flip baseline. Gradient-boosted trees come after version one. (project-design)
- D-010: Price data comes from Alpaca's free market-data plan, only through one price provider module; start and end prices come from full-market bars at least 15 minutes old; price data is cached and shared across users; the API key comes from environment variables. Open item: confirm public display is allowed before public hosting. (project-design)
- D-009: A prediction is a direction (up or down) plus a confidence percentage; it scores "yes" if the end price is on the predicted side of the start price. Price-range predictions come after version one. (project-design)
- D-008: Intraday timeframes (15m, 1h, 4h) can only be picked if they end at or before today's actual US market close (see D-025, D-041); outside market hours only 1d, 1w and 1mo are available. (project-design)
- D-007: Version one supports only a fixed list of big US stocks (the S&P 500); any US stock and global stocks come after version one. (project-design)
- D-006: The timeframe picker offers exactly: 15 minutes, 1 hour, 4 hours, 1 day, 1 week, 1 month. (project-design)
- D-005: Users can pick both intraday timeframes (under a day) and daily-or-longer timeframes. (project-design)
- D-004: The product is called "Stock Predictor" and has nothing to do with prop firms. (project-design)
- D-003: The app only makes and scores predictions; it never places, imports or tracks trades and never connects to a broker or trading account. (project-design)
- D-002: Version one is the smallest useful loop: pick a stock, pick a timeframe, press Predict, and when the timeframe ends see a yes/no visual of whether the prediction came true (plus "no change" for flat results, per D-022). (project-design)
- D-001: The app is for public users and the code is open source on GitHub. (project-design)
- D-000: Agents build in threads that open pull requests into milestone branches; only Mohannad merges into main. See workflow/README.md. (setup)

## Full entries for the current milestone

Format for each entry, newest first:

```
## D-001: Short title (YYYY-MM-DD)
Question: what was asked, and where it came from (issue or PR number)
Options considered: A, B, C
Decision: what Mohannad chose
Why: his reason, in a sentence or two
Applies to: which parts of the app this rule covers
```

## D-041: Intraday timeframes may end exactly at the close (2026-11-25)
Question: may an intraday prediction end exactly at the close? (Mohannad's review of the design docs)
Options considered: end strictly before the close (D-008 wording); end at or before the close
Decision: at or before. 15m, 1h and 4h are offered if they end at or before that day's actual close (D-025). For example, 15:00 + 1h = 16:00 is allowed.
Why: a prediction ending at the close is well defined and scorable.
Applies to: timeframe picker and server-side validation. Updates the D-008 rule.

## D-040: Model input uses only bars at least 15 minutes old (2026-11-25)
Question: which bars can the model see at press time? (Mohannad's review; D-030's input conflicted with what the free plan gives at press time)
Options considered: none offered; Mohannad stated it.
Decision: the model's input is only the full-market bars that are at least 15 minutes old at press time, meaning bars ending at least 15 minutes before the start time. Milestone 2's training must use the same 15-minute gap. The no-look-ahead test checks this stricter rule.
Why: that's all Alpaca's free plan gives at press time (D-010). Training with the same gap keeps training and live predictions consistent.
Applies to: model module interface, no-look-ahead test, milestone 2 training. Updates the D-011 and D-030 rules.

## D-039: Start price filled in later by the worker (2026-11-25)
Question: how is the start price obtained, given that the free plan can't return the full-market bar from 1 minute ago at press time? (Mohannad's review; D-021 conflicted with D-010)
Options considered: none offered; Mohannad stated it.
Decision: D-021's definition of the start price stays. The prediction is saved with `start_price` empty. The worker fills `start_price` and `start_price_time` once that bar is at least 15 minutes old, under the same lock as scoring. The prediction page shows "start price pending" until then.
Why: keeps D-010's rule that start and end prices come from full-market bars at least 15 minutes old.
Applies to: Predict, worker, data model (the worker also writes start_price), prediction page. Updates the D-021 rule.

## D-038: Health URL format (2026-11-25)
Question: what does the health URL return? (project design session, proposal P7)
Options considered: proposed by the design partner; Mohannad approved it.
Decision: `/health` returns JSON including `overdue_count`, the number of predictions more than 10 minutes past their expected result time (D-017).
Why: a machine-readable way to spot a stopped worker.
Applies to: web app, worker monitoring

## D-037: Prediction pages and "my predictions" (2026-11-25)
Question: how does a user see a prediction and find it again? (project design session, proposal P6)
Options considered: proposed by the design partner; Mohannad approved it.
Decision: each prediction has a page at `/p/<id>` showing "pending until <time> ET" or its yes/no/no-change result, with the notice (D-019). The home page lists "my predictions" from the browser's storage (D-012).
Why: shareable links without accounts.
Applies to: web pages, browser storage

## D-036: Milestone 1 stand-in model (2026-11-25)
Question: what does the stand-in model in `prediction-loop` do? (project design session, proposal P5)
Options considered: proposed by the design partner; Mohannad approved it.
Decision: it always predicts "up" with probability 0.50, through the D-030 interface, and is registered as model version `stand-in-1` with accuracy per timeframe measured on the generated sample (D-024, D-031).
Why: the simplest model that exercises the full interface and the accuracy notice.
Applies to: model module, model-version registration, milestone 1

## D-035: Folder layout, commands and settings (2026-11-25)
Question: how is the code organised, which commands run it, and how are settings provided? (project design session, proposals P3 and P4)
Options considered: proposed by the design partner; Mohannad approved it, with lint written as two separate commands because `&&` doesn't work in Windows PowerShell 5.1.
Decision: as written in CLAUDE.md under Commands and Conventions. Only `src/stock_predictor/prices/` talks to Alpaca. Settings come from environment variables listed in a committed `.env.example`; `.env` is git-ignored.
Why: workers need one shared layout and command set; Mohannad works on Windows.
Applies to: whole codebase, CLAUDE.md

## D-034: Time zones (2026-11-25)
Question: how are times stored and shown? (project design session, proposal P2)
Options considered: proposed by the design partner; Mohannad approved it.
Decision: all times are stored in UTC and shown to users in New York time, labelled "ET".
Why: one unambiguous storage format; the market runs on New York time.
Applies to: data model, pages, scoring, calendar

## D-033: Storing IP addresses (2026-11-25)
Question: how are abuse limits counted without storing personal data? (project design session, proposal P1)
Options considered: proposed by the design partner; Mohannad approved it.
Decision: raw IP addresses are never stored. Each prediction stores `requester_ip_hash`, a salted hash of the IP (from the one place that works out the IP, D-027), and limits are counted from it. The salt comes from the `IP_HASH_SALT` environment variable.
Why: keeps version one free of personal data (D-012) while still allowing per-IP limits.
Applies to: data model, rate limiting, settings

## D-032: The problem the product solves (2026-11-25)
Question: what problem does Stock Predictor solve? (project design session)
Options considered: A an honest experiment with a visible track record; B a fun guessing tool; C a learning project for developers
Decision: A for the brief, with C's line added for the GitHub readme.
Why: matches the honesty rules already chosen (D-019, D-029); C's line suits an open-source repo.
Applies to: brief, readme, tone of the app's wording

## D-031: Tested accuracy in milestone 1 (2026-11-25)
Question: where does the notice's tested accuracy (D-019) come from in milestone 1, before the train command exists? (project design session; gap in D-030's interface)
Options considered: A create the model-versions table in milestone 1 and register the stand-in; B the model module reports its own accuracy, table added in milestone 2; C hard-code 50% in milestone 1
Decision: A. The model-versions table exists from milestone 1, holding each version's ID, training date, tested accuracy per timeframe, and where the accuracy was measured (generated sample or real data). The stand-in model is registered as a model version, with accuracy measured on the generated sample (D-024). The notice looks up accuracy by model version and timeframe. When the accuracy was measured on the generated sample, the notice says "measured on test data, not real prices".
Why: the data model is final from day one, so milestone 2 changes no tables. Users are never misled about where an accuracy number came from.
Applies to: data model (model versions), notice (D-019), stand-in registration, milestone 2's train command (D-023)

## D-030: First milestone and the model interface (2026-11-25)
Question: what goes in the first milestone? (project design session)
Options considered: A all of version one in one milestone; B "data-and-model" then "predict-and-score"; C "prediction-loop" with a stand-in model, then "logistic-model"
Decision: C. Milestone 1 is `prediction-loop`: every version-one rule end to end, with a stand-in model behind the model module. Milestone 2 is `logistic-model`: the real model (D-011), the train command and model versions (D-023). The model module's interface is fixed now, from D-009, D-011 and D-023:
- Input: the ticker, the timeframe, the start time, and only the bars from before the start time.
- Output: a direction, a probability, and the model version.
The stand-in model uses exactly this interface. The no-look-ahead test (D-011) is built in milestone 1 against it, so milestone 2 only swaps what's inside the module.
Why: proves the pass bar (D-029) first, and proves the model can be replaced without touching the rest of the app.
Applies to: milestone plan, model module, feature docs
Updated by D-040: the input bars must end at least 15 minutes before the start time.

## D-029: How we'll know version one works; worker catch-up (2026-11-25)
Question: what are the signs that version one works? (project design session)
Options considered: A the loop works reliably; B the model beats the coin flip; C both, with A as the pass bar and B only reported
Decision: C.
- Pass bar: 99% of predictions are scored within 10 minutes of their expected result time, and no prediction is ever scored twice or with look-ahead. The 10 minutes only count time the worker was running.
- Reported, not a pass bar: tested accuracy per timeframe against the 50% coin flip (D-011).
- Catch-up: when the worker starts after being stopped (for example Mohannad's PC was off), it first scores every overdue prediction, using the correct end price from the past, not the price at catch-up time. A test covers this catch-up.
Why: a fair bar for the build that's fully in Mohannad's control, while tracking the model's real result honestly. Running locally (D-018) means the worker will often be off, so catch-up must be correct.
Applies to: brief (success measures), worker start-up, scoring, tests
Amended (2026-11-25, after D-039): catch-up first fills any missing start prices (D-021's definition, the correct bar from the past), then scores overdue predictions. The catch-up test covers both.

## D-028: Source of the S&P 500 list (2026-11-25)
Question: where does the list of S&P 500 stocks for the picker come from? (project design session)
Options considered: A a hand-updated list file in the repo; B download from a public web page on a schedule; C a paid index-membership feed
Decision: A. The file holds each ticker (in Alpaca's symbol format) and company name. At startup, the app checks the list against Alpaca with one call to Alpaca's asset list (not one call per ticker). Any ticker that isn't a tradable asset only produces a warning in the logs, never a crash. When no Alpaca key is set, the check is skipped with a log line, so CI and tests start the app normally.
Why: tickers and company names are public facts with no licensing worry, and the list is the same everywhere. The check catches drift cheaply.
Applies to: stock list file, stock chooser, app startup, price provider module (the asset-list call goes through it)

## D-027: Abuse limits (2026-11-25)
Question: how strict is the per-IP abuse limit from D-012? (project design session)
Options considered: A 10 per IP per hour; B 30 per IP per hour plus a global cap; C no limit until public hosting
Decision: B. 30 predictions per IP address per hour, and a global cap of 1,000 predictions per hour across all users. Above either limit, Predict says "busy, try again later". Both numbers are environment variable settings with those defaults. The user's IP address is worked out in one place, with a setting for whether to trust the host's forwarded-IP header (the header a hosting platform adds with the real visitor's address), so it can be set correctly once the public host is chosen.
Why: generous for real users while protecting the database and the Alpaca budget. Settings allow tuning without code changes.
Applies to: Predict endpoint, rate limiting, settings, hosting setup (D-018 open item)

## D-026: Market calendar source (2026-11-25)
Question: where does the app get trading days, holidays and half-days? (project design session)
Options considered: A Alpaca's calendar endpoint through the price provider; B a Python calendar library (for example exchange_calendars); C a hand-kept holiday file in the repo
Decision: A. The calendar is fetched through the price provider module, cached, and refreshed once a day. The cache always covers at least the next 2 months, so 1mo predictions can be checked against it.
Why: one source means the calendar and the prices never disagree, and swapping providers swaps both together.
Applies to: price provider module, timeframe picker and validation (D-008, D-022, D-025), scoring (D-020)
Known risk: if the calendar can't be fetched and the cache doesn't cover what's needed, the picker should show "try again" rather than guess.

## D-025: Half-day close for intraday timeframes (2026-11-25)
Question: on an early-close half-day, does D-008's "today's close" mean 16:00 or the actual close? (project design session; raised by D-024)
Options considered: the regular 16:00 close; the actual close that day
Decision: the actual close that day: 16:00 New York time normally, 13:00 on early-close half-days.
Why: otherwise an intraday prediction could end after the market has shut.
Applies to: timeframe picker and validation. Updates the D-008 rule.

## D-024: Test sample of bars (2026-11-25)
Question: where does the saved sample of bars for tests (D-023) come from? (project design session)
Options considered: A made-up bars from a script with a fixed seed; B a small slice of real Alpaca bars; C real bars from a source that allows redistribution
Decision: A. The generated sample must include the quirks the rules depend on: a weekend gap, a US market holiday, an early-close half-day (13:00 close), and a few missing minute bars.
Why: no licensing risk in a public repo, and the built-in quirks let the timeframe, start-price, flat-result and scoring tests (D-008, D-020, D-021, D-022) run on realistic edge cases.
Applies to: test data, the generator script, all tests of timeframe, start-price, flat-result and scoring rules

## D-023: Training, storing and updating models (2026-11-25)
Question: how are the 6 models trained, stored and updated? (project design session)
Options considered: A manual train command saving model versions in the database; B train once and commit model files to the repo; C worker retrains automatically on a schedule
Decision: A. A manual `train` command fetches history through the price provider, trains all 6 models with the time split (D-011), and saves each model with its tested accuracy and training date as a model version in the database. The app always uses the newest version, and each prediction records which model version made it. Conditions:
1. Training shares the app's Alpaca rate limit and cache (D-010), so a training run can never use up the 200 calls a minute the scoring job needs. It's fine if training is slow.
2. Tests never call Alpaca or need an API key. Training and model tests use a small saved sample of bars in the repo, so CI and the agents' threads can run them.
Why: simple and repeatable, and the accuracy notice (D-019) always matches the model that made the prediction.
Applies to: train command, model module, model versions in the database, price provider rate limiting, tests and test data
Known cost: models go stale unless Mohannad reruns training.

## D-022: Flat results (2026-11-25)
Question: what happens when the end price equals the start price? (project design session; left open in D-009)
Options considered: A flat counts as "no"; B flat is a third result, "no change", left out of accuracy; C block predictions that can't move, and count any remaining flat as "no"
Decision: a mix of C and B. The picker blocks any timeframe whose start and end would use the same close, meaning no new full-market price would exist between start and end (for example, 1d pressed on Friday after the close, or on a weekend). Any flat result that still happens is shown as "no change" and left out of accuracy.
Why: removes the predictions that can't move at all, and keeps accuracy honest for the rare real flat result.
Applies to: timeframe picker and server-side validation (alongside D-008), scoring (three results: yes, no, no change), result visual, accuracy calculation

## D-021: Start price (2026-11-25)
Question: which price is a prediction's start price? (project design session)
Options considered: proposed as one rule alongside D-020; Mohannad accepted it.
Decision: the close of the last full 1-minute bar before Predict was pressed. If the market is closed, the last close.
Why: uses only prices from before the prediction, which fits the no-look-ahead rule (D-011).
Applies to: prediction creation, scoring
Updated by D-039: the worker fills the start price once its bar is at least 15 minutes old.

## D-020: When 1d, 1w and 1mo predictions end (2026-11-25)
Question: exactly when do 1d, 1w and 1mo predictions end, and which price is used? (project design session; left open in D-008)
Options considered: A at the close N trading days later; B same time of day N trading days later; C calendar time (24 hours, 7 days, 1 calendar month)
Decision: C. The end moment is start + 24 hours, start + 7 days, or start + 1 calendar month. The end price is the last full-market price at or before that moment: the minute bar if the market is open then, otherwise the last close.
Why: matches how people say "a day", "a week" and "a month".
Applies to: scoring, the result time shown to users
Known cost: a 1d prediction made on a Friday ends on Saturday and is judged on Friday's close, so very little time passes. The same happens for predictions ending on holidays.
Open item: Mohannad wants to revisit this logic later.

## D-019: "Not financial advice" notice (2026-11-25)
Question: how does the app show a "not financial advice" notice? (project design session; Mohannad asked for this question during D-011)
Options considered: A footer line on every page; B footer line plus a notice next to every prediction and result; C one-time pop-up to accept, plus footer
Decision: B. The notice next to each prediction and result shows the model's tested accuracy for that timeframe, for example: "Experimental model, not financial advice. Tested accuracy for 15 minutes: 51% (coin flip is 50%)." There is no separate stats page in version one.
Why: shared prediction links may be opened by strangers, so the warning and the honest accuracy number travel with the prediction itself.
Applies to: every page footer, prediction and result views, model module (must expose tested accuracy per timeframe, from D-011)

## D-018: Hosting (2026-11-25)
Question: where does the app run? (project design session)
Options considered: A local only for version one, with Docker Compose; B managed platform (for example Render or Railway); C own server (VPS) with Docker Compose
Decision: A. Docker Compose starts PostgreSQL, the web app and the worker on Mohannad's machine.
Why: public hosting has to wait for the Alpaca open item (D-010), so there's no point paying for it yet. Docker Compose keeps a later move small.
Applies to: environments, local setup, CLAUDE.md commands
Open item: choose the public host (for example a managed platform or a VPS) after Alpaca's terms are confirmed.

## D-017: How the scoring job runs (2026-11-25)
Question: how does the job that scores finished predictions run? (project design session)
Options considered: A scheduler inside the web app process; B separate worker process from the same codebase; C task queue (Celery with Redis)
Decision: B, with these conditions:
1. The scoring logic is a plain function that can be called and tested on its own.
2. A database lock ensures a prediction is never scored twice, however the job is run.
3. The app's health URL reports whether scoring is behind, for example the number of predictions overdue by more than 10 minutes past their expected result time, so a stopped worker shows up in the health check.
Why: scoring is independent of web traffic and easy to test. The lock and health check cover the risk of double scoring and of a worker silently stopping.
Applies to: worker process, scoring function, database locking, health URL

## D-016: Python tooling (2026-11-25)
Question: which tools for installing, testing, linting and type checking? (project design session)
Options considered: A uv, pytest, ruff, mypy; B Poetry, pytest, ruff, mypy; C pip and venv with requirements.txt, pytest, ruff, mypy
Decision: A.
Why: fast installs and an exact lock file, so every machine and worker gets the same versions.
Applies to: CLAUDE.md commands, CI, every worker's setup

## D-015: Database (2026-11-25)
Question: where are predictions and cached prices stored? (project design session)
Options considered: A SQLite; B PostgreSQL via SQLAlchemy and Alembic; C SQLite locally, PostgreSQL when hosted
Decision: B. PostgreSQL everywhere, through SQLAlchemy, with Alembic for migrations.
Why: standard for a public hosted app, safe with the web server and scoring job writing at the same time, and tests run against the same database as production.
Applies to: all stored data, migrations, tests
Known cost: local development and database tests need PostgreSQL, usually through Docker.

## D-014: Web framework and pages (2026-11-25)
Question: which Python web framework, and how are pages built? (project design session)
Options considered: A FastAPI with server-rendered pages and htmx; B Django; C Flask
Decision: A. FastAPI, with Jinja2 templates for the pages and htmx for updating parts of a page without a reload.
Why: fits a small app with an API at its centre. Type-checked inputs make server-side checks (such as D-008's timeframe rule) easy.
Applies to: web server, pages, API, input validation
Known cost: no built-in admin or login; accounts after version one need a login library added.

## D-013: App shape (2026-11-25)
Question: one web app, separate frontend and backend, or mobile app? (project design session)
Options considered: A one web app (pages, API and scoring job in one codebase); B separate JavaScript frontend plus Python API; C mobile app plus backend
Decision: A.
Why: the interface is small, and one app means one thing to build, test and host. The model is Python, so the whole app can be too.
Applies to: whole app structure, hosting

## D-012: User accounts (2026-11-25)
Question: do users need an account to make and find predictions? (project design session)
Options considered: A no accounts, a link per prediction plus a browser-remembered list; B optional accounts; C accounts required
Decision: A for version one. B (optional accounts) is planned for after version one.
Why: smallest way to deliver D-002 publicly, with no personal data to protect. Giving every prediction its own ID keeps the door open for B.
Applies to: data model (no user table), prediction pages and links, "my predictions" list (stored in the browser), abuse limits (per IP address)

## D-011: How predictions are made (2026-11-25)
Question: what method produces the direction and confidence percentage? (project design session)
Options considered: A logistic regression on recent-price signals; B gradient-boosted trees (for example LightGBM); C deep learning or an AI language model
Decision: A for version one, one model per timeframe. B is planned for after version one. Conditions:
1. The model sits behind its own module, so B can replace it without touching the rest of the app.
2. No look-ahead: training and testing split by time (train on older bars, test on newer ones), and a test proves the inputs for a prediction only use bars from before the prediction's start time.
3. The tested accuracy per timeframe is recorded against the 50% coin-flip baseline, so we know whether the model beats guessing.
Why: the hard part of version one is an honest predict-wait-score loop. A gives real probabilities for the confidence percentage and is hard to fool yourself with.
Applies to: model module, training, feature inputs, tests
Updated by D-040: input bars must end at least 15 minutes before the start time, in live predictions and in training.

## D-010: Price data source (2026-11-25)
Question: where does the price data come from? (project design session)
Options considered: A Alpaca free market-data plan; B Massive (formerly Polygon.io) Starter, about $29 a month; C Yahoo Finance via yfinance
Decision: A, with these conditions:
1. All price fetching goes through one "price provider" module. No other code calls Alpaca directly.
2. Both the start price and the end price of a prediction come from full-market bars, fetched once they are at least 15 minutes old. The app tells users the result appears about 15 minutes after the timeframe ends.
3. Price data is cached and shared across users, so the app stays under Alpaca's limit of about 200 calls a minute.
4. The Alpaca API key comes from environment variables, never from the repo (the repo is public).
Why: free, official, minute-level data with generous limits. The provider module keeps a later switch small. Full-market bars avoid scoring on the single-exchange (IEX) live price.
Applies to: price provider module, scoring, caching, the result-timing message shown to users
Open item: before the app is hosted for the public, Mohannad must confirm in Alpaca's market data agreement (or with Alpaca support) that showing the data or results to public users is allowed. If it isn't, switch providers by changing the price provider module.

## D-009: What a prediction says (2026-11-25)
Question: what does a prediction contain, and what counts as "yes"? (project design session)
Options considered: A up or down only; B up or down plus a confidence percentage; C a price range
Decision: B for version one. C (price range) is planned as an extra feature after version one.
Why: costs little more than A, and showing confidence tells public users it's a guess, not a promise. It also allows checking later whether the confidence levels are accurate.
Applies to: prediction engine (must output a probability), prediction display, scoring
Flat end prices: resolved by D-022.

## D-008: Market hours and intraday timeframes (2026-11-25)
Question: how does the timeframe clock work when the US market is closed? (project design session)
Options considered: A market-time clock that pauses overnight; B intraday only within today's session; C wall-clock time judged on last known price
Decision: B. 15m, 1h and 4h are only offered when they would end before today's regular close (16:00 New York time). When the market is closed (nights, weekends, holidays), only 1d, 1w and 1mo can be picked.
Why: simplest to build and explain; "1 hour" really means one hour from now.
Applies to: timeframe picker, prediction validation on the server, scoring
Known cost: intraday options are unavailable for much of the day and all weekend, especially for users outside US time zones. Exact rule for when 1d, 1w and 1mo end is still to be decided (decided in D-020).
Updated by D-025 (actual close on half-days) and D-041 (may end at, not only before, the close).

## D-007: Which stocks users can choose (2026-11-25)
Question: which stocks can users pick from? (project design session)
Options considered: A fixed list of big US stocks (S&P 500); B any US-listed stock; C global stocks
Decision: A for version one. B and C are planned for after version one.
Why: big US stocks trade every minute and have plenty of free data, which 15-minute predictions need. Supporting one market keeps hours and holidays simple.
Applies to: stock chooser, data source, market hours rules. Design shouldn't make B or C impossible later (for example, don't hard-code one exchange where it's easy to avoid).

## D-006: Exact timeframe list (2026-11-25)
Question: which exact timeframes go in the picker? (project design session)
Options considered: A 1h, 1d, 1w, 1mo; B 15m, 1h, 4h, 1d, 1w, 1mo; C any end time the user chooses
Decision: B. 15 minutes, 1 hour, 4 hours, 1 day, 1 week, 1 month.
Why: Mohannad wants more choice and fast feedback from the 15-minute option.
Applies to: timeframe picker, data source (needs minute-level intraday prices), scoring rules
Known cost: 15-minute predictions need minute-level data, which is the most limited on free plans; 4-hour predictions often run past the market close.

## D-005: Which kinds of timeframe (2026-11-25)
Question: can users pick days-and-longer timeframes, intraday timeframes, or both? (project design session)
Options considered: A daily and longer only; B intraday only; C both
Decision: C. Both.
Why: Mohannad wants users to have the full range of choices in version one.
Applies to: timeframe picker, data source (needs intraday and daily prices), scoring rules
Known cost: two data paths and two scoring rules to build and test; intraday data is more limited on free providers.

## D-004: Product name and focus (2026-11-25)
Question: what is the product called, and does it relate to prop firms? (project design session; Mohannad corrected an earlier prompt that mentioned prop firms)
Options considered: none offered; Mohannad stated it.
Decision: the app is called "Stock Predictor". It has nothing to do with prop firms.
Why: the prop firm wording came from a different context and was a mistake.
Applies to: whole app, brief, naming

## D-003: Does the app touch trading? (2026-11-25)
Question: does the app place trades, track and analyse trades, or only make predictions? (project design session)
Options considered: A predictions only; B track and analyse trades (journal, challenge rules, stats); C place trades through a broker
Decision: A. Predictions only.
Why: it matches the first version in D-002, and a public, open-source app shouldn't hold users' broker logins or money.
Applies to: whole app; no broker connections, no order sending, no trade import or journal

## D-002: Smallest useful first version (2026-11-25)
Question: what must version one do at minimum? (project design session)
Options considered: Mohannad gave the scope directly, without a list of options.
Decision: a stock chooser, a timeframe picker and a Predict button. When the prediction's timeframe ends, the app shows a yes/no visual of whether the prediction came true.
Why: this is the smallest loop that shows whether a prediction was right.
Applies to: brief (version one features), first feature goal

## D-001: Who uses the app (2026-11-25)
Question: who is the app for: just Mohannad, a few friends, or the public? (project design session)
Options considered: A personal tool; B shared view for a few people, no logins; C public users
Decision: C. Public users, and the code is open source on GitHub.
Why: Mohannad wants it to be a public, open project.
Applies to: whole app; hosting, accounts and data-licensing decisions must assume public use
