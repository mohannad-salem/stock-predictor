# Data model

The most important thing to get right early; changes here ripple everywhere. Any change to this file is a design decision and goes through Mohannad.

All times are stored in UTC and shown to users in New York time, labelled "ET" (D-034). All tables live in PostgreSQL (D-015) and change only through Alembic migrations.

## Entities

| Entity | Key fields | Notes |
| --- | --- | --- |
| Prediction | `id` (random UUID), `ticker`, `timeframe` (one of 15m, 1h, 4h, 1d, 1w, 1mo), `created_at`, `start_price`, `start_price_time`, `end_time`, `result_due_at`, `direction` (up or down), `probability`, `model_version_id`, `status` (pending or scored), `outcome` (yes, no or no_change; empty until scored), `end_price`, `end_price_time`, `scored_at`, `requester_ip_hash` | `id` is used in the share link `/p/<id>` (D-012, D-037). `start_price` is the close of the last full 1-minute bar before Predict was pressed, or the last close if the market was closed (D-021). It and `start_price_time` are empty when the prediction is saved; the worker fills them once that bar is at least 15 minutes old (D-039). `end_time` follows D-008 and D-020. `result_due_at` is `end_time` + 15 minutes (D-010). `requester_ip_hash` is a salted hash, never the raw IP (D-033). Only the worker writes the start-price fields (`start_price`, `start_price_time`) and the scoring fields (`status`, `outcome`, `end_price`, `end_price_time`, `scored_at`), under the same database lock (D-017, D-039). |
| ModelVersion | `id` (for example `stand-in-1`), `kind` (stand-in or logistic), `trained_at`, `accuracy_source` (generated_sample or real_data), `model_data` (empty for the stand-in) | Exists from milestone 1 (D-031). The app uses the newest version (D-023). |
| ModelAccuracy | `model_version_id`, `timeframe`, `accuracy`, `sample_size` | One row per model version and timeframe. Results of "no change" are left out of accuracy (D-022). Shown in the notice next to every prediction and result (D-019). |
| PriceBar | `ticker`, `resolution` (1-minute or 1-day), `bar_start`, `open`, `high`, `low`, `close`, `volume`, `fetched_at` | Shared price cache for all users, training and scoring (D-010, D-023). Unique on (`ticker`, `resolution`, `bar_start`). Written only by the price provider module. Full-market bars only. |
| MarketDay | `date`, `open_time`, `close_time`, `fetched_at` | Calendar cache from Alpaca, refreshed once a day, always covering at least the next 2 months (D-026). `close_time` is 13:00 ET on half-days (D-025). Days not listed are closed. |

The stock list is not a table. It is the file `data/sp500.csv` (ticker in Alpaca's symbol format, company name), updated by hand (D-028).

## Relationships

- Each Prediction belongs to one ModelVersion (`model_version_id`). A ModelVersion has many Predictions.
- Each ModelVersion has one ModelAccuracy row per timeframe.
- A Prediction's `ticker` must be in `data/sp500.csv`. PriceBar rows are keyed by the same ticker.
- The notice for a prediction looks up ModelAccuracy by the prediction's `model_version_id` and `timeframe`. If the version's `accuracy_source` is generated_sample, the notice also says "measured on test data, not real prices" (D-031).

## Who can see or change what

There are no user accounts in version one (D-012).

- Prediction: anyone with the link can view it. Users create predictions only through Predict (subject to D-027 limits). Nobody can edit or delete a prediction through the app. Only the worker fills in the start-price and scoring fields (D-039). `requester_ip_hash` is never shown to users.
- ModelVersion and ModelAccuracy: read by the app; written only by the stand-in registration (milestone 1) and the `train` command (milestone 2).
- PriceBar and MarketDay: written only by the price provider module; never shown raw beyond the start and end prices of a prediction.
- "My predictions": a list of prediction IDs kept in the user's browser storage, not on the server (D-037).
