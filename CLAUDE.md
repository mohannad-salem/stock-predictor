# Project guide for Claude

Read this first in every session. It is the single source of truth for how work happens here.

## The project

- Product brief: docs/brief.md
- Architecture: docs/architecture.md
- Data model: docs/data-model.md
- Decisions log: docs/decisions.md (read it before making any design choice; if your question is answered there, follow it). Its "Rules in force" list is what applies; open docs/decisions-archive/ only when you need an old decision's reasoning.
- How the workflow runs: workflow/README.md

## Stack

Details and reasons are in docs/architecture.md.

- Language: Python, managed with uv (D-016)
- Web: FastAPI, Jinja2 templates, htmx; no JavaScript framework (D-014)
- Database: PostgreSQL via SQLAlchemy, Alembic migrations, in every environment (D-015)
- Price data and market calendar: Alpaca free market-data plan, only through `src/stock_predictor/prices/` (D-010, D-026)
- Model: logistic regression (scikit-learn) behind the model module; stand-in model in milestone 1 (D-011, D-030, D-036)
- Processes: one web process and one worker process from the same codebase (D-013, D-017)
- Tools: pytest, ruff, mypy (D-016)
- Running locally: Docker Compose (PostgreSQL, web, worker). No public hosting yet (D-018)

## Commands

Each command is on its own line because `&&` doesn't work in Windows PowerShell 5.1 (D-035).

- Install: `uv sync`
- Run locally (everything): `docker compose up`
- Run locally (without Docker for the app; database still in Docker): `docker compose up -d db`, then `uv run alembic upgrade head`, then `uv run uvicorn stock_predictor.web.app:app --reload` and, in a second terminal, `uv run python -m stock_predictor.worker`
- Database migrations: `uv run alembic upgrade head`
- Test: `uv run pytest` (needs PostgreSQL, for example `docker compose up -d db`; never needs an Alpaca key)
- Type check: `uv run mypy src`
- Lint: `uv run ruff check .`
- Format check: `uv run ruff format --check .`

## Conventions

- Folders (D-035):
  - `src/stock_predictor/prices/`: price provider, market calendar, asset list. The only code that talks to Alpaca.
  - `src/stock_predictor/model/`: the model module (fixed interface, see docs/architecture.md).
  - `src/stock_predictor/scoring/`: the plain scoring function.
  - `src/stock_predictor/worker.py`: the worker process (catch-up, then scoring loop).
  - `src/stock_predictor/web/`: FastAPI app and routes. `src/stock_predictor/templates/`: Jinja2 templates.
  - `src/stock_predictor/db/`: SQLAlchemy tables and session. `migrations/`: Alembic.
  - `data/sp500.csv`: the stock list (ticker in Alpaca's symbol format, company name).
  - `tests/`: tests. `tests/data/`: the generated bar sample. `scripts/`: the sample generator (fixed seed).
- Settings come from environment variables only, listed in a committed `.env.example`; `.env` is git-ignored. Names: `ALPACA_API_KEY_ID`, `ALPACA_API_SECRET_KEY`, `DATABASE_URL`, `RATE_LIMIT_PER_IP_PER_HOUR` (default 30), `RATE_LIMIT_GLOBAL_PER_HOUR` (default 1000), `TRUST_FORWARDED_IP` (default false), `IP_HASH_SALT`.
- Tests never call Alpaca and never need an API key. Use the generated sample in `tests/data/`; no real provider data goes in the repo (D-023, D-024).
- Times are stored in UTC and shown in New York time, labelled "ET" (D-034).
- Database changes go through Alembic migrations and must match docs/data-model.md.

- Secrets (keys, tokens, passwords) never go in the repo, a pull request or a chat. Keep them in environment variables or a git-ignored file.
- Every change ships with tests for its acceptance criteria.
- If a command in this file can't run in your environment (for example the database tests need Docker), say so in the pull request instead of skipping it silently.
- Keep pull requests small: one GitHub issue per pull request.

## Branches

- `main` is Mohannad's. Never push to it, never merge into it, never open a pull request that targets it. Only Mohannad merges a milestone into main.
- Each milestone has a branch named `milestone/<slug>`. Feature branches come off it and pull requests target it.
- Never force push. Never rewrite history on a shared branch.
- Never add or remove the `reviewer-approved` or `changes-requested` labels; only the reviewer sets them. Open pull requests as normal pull requests, not drafts.

## When something is not covered

If a design choice is not answered by the docs or the decisions log, do not guess. Stop, describe the question and 2 or 3 options in the pull request or issue, and add the `needs-decision` label. Mohannad answers, and the design partner records the answer in docs/decisions.md.

## Commits

Write clear commit messages describing what changed and why. Do not add co-author or "generated with" lines.
