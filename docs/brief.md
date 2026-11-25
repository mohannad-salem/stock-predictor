# Product brief: Stock Predictor

Owner: Mohannad. Drafted with the design partner; every line here is a decision Mohannad made.

## Who it's for

Public users. The code is open source on GitHub. (D-001)

## The problem

Stock prediction apps make big claims and never show their track record. Stock Predictor makes a prediction, waits, and shows publicly whether it was right, with its accuracy against a coin flip always visible. (D-032)

Line for the GitHub readme: "An open-source example of building an honest prediction loop: data, model, scoring and no look-ahead." (D-032)

## Version one features

1. Choose a stock from a fixed list of big US stocks (the S&P 500). (D-007)
2. Choose a timeframe: 15 minutes, 1 hour, 4 hours, 1 day, 1 week or 1 month. Intraday options only appear when they would end at or before today's actual US market close (13:00 on half-days), and no timeframe is offered if its start and end would use the same closing price. (D-005, D-006, D-008, D-022, D-041)
3. Press Predict to get a prediction for that stock over that timeframe: up or down, with a confidence percentage. (D-009)
4. When the timeframe ends, see a yes/no visual of whether the prediction came true, or "no change" if the price ended exactly flat. (D-002, D-022)
5. Every prediction and result carries a "not financial advice" notice with the model's tested accuracy for that timeframe compared with a 50% coin flip, and every page has a footer notice. (D-019)

## Out of scope for version one

- Placing trades, connecting to brokers or trading accounts, importing or journaling trades. (D-003)
- Anything to do with prop firms or trading challenges. (D-004)
- Stocks outside the S&P 500, and non-US stocks. Planned for after version one. (D-007)
- Price-range predictions. Planned as an extra feature after version one. (D-009)
- User accounts and sign-in. Optional accounts are planned for after version one. (D-012)
- A separate stats page. (D-019)
- Gradient-boosted tree models (for example LightGBM). Planned for after version one. (D-011)

## How we'll know it works

Pass bar (D-029):
- 99% of predictions are scored within 10 minutes of their expected result time, counting only time the scoring worker was running.
- No prediction is ever scored twice, and none is scored using prices from after its start time (look-ahead).

Reported, not a pass bar:
- The model's tested accuracy per timeframe against the 50% coin flip. This guides what comes after version one.
