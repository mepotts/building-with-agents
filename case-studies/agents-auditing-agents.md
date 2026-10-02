# Agents auditing agents

**Status:** Local repository. Claude Code agents wrote the code. I led the audit and decide what counts as a result. The remediation is in my working tree and not yet committed, so treat it as in progress until it is. This write-up contains no personal financial information.

**Scope:** Five agents audited the agent-written codebase: four adversarial code auditors and one primary-source researcher. Every number was reproduced by running code or extracted from a primary source, and nothing from docstrings or vendor material is quoted without a flag.

**Result:** The evaluation harness, the repo's mandatory gate, was audited and repaired too. Verdicts moved toward "no evidence", and the audit itself had to be corrected the same day.

## Background

OpenStock is a Python research platform for equities and options. Over three generations of work, agents wrote the strategies, the backtest engines, a volatility-forecasting module, and, in August, an evaluation harness. The harness tells a real result from a lucky one, and the repo's agent guide says nothing counts as a result until it does. I then pointed agents at the whole repository, including the harness itself after a false start.

## Design

The four auditors took adversarial lanes: the paper-trading loop, the legacy backtest engines and experiments, the volatility module, and the evaluation gate. Each finding carries a file-and-line anchor. The report also lists what checked out. Black-Scholes agrees with Hull to four decimals, put-call parity holds to 1e-15, and equity-engine positions equal the one-bar-shifted signal exactly. Purge and embargo are clean on all 10 cross-validation splits.

## What it caught

| Finding | Effect |
|---|---|
| Look-ahead: the script that seeded production weights multiplied today's signal by today's return, unshifted. | Sharpe 3.29 vs 1.00 shifted (3.16 vs 0.94 on a second component). The reported "OOS Sharpe 3.07" was inflated about 3 times. |
| Regime leakage: a train-only regime-model (HMM) fit was discarded and refit on the full sample. | Every "with regime" result leaked. "The overlay prevented the COVID drawdown" was the leak's signature. |
| Settlement and accounting: in-the-money expiries settled at $0, Sharpe and drawdown came from realized P&L only, entry credit never reached cash, and annualization was wrong in both directions. | "0.00% max drawdown, 97 to 100% win rate" were artifacts. A +4.09%/yr wheel result corrects to about +1.9%. |
| CVaR sign bug: a negated quantile selected about 95% of the sample instead of the worst 5%. (CVaR is the average loss in the worst 5% of days.) | Tail risk understated about 17-fold. |
| Volatility module and its study: a day-count mismatch, expected shortfall collapsing to zero, and a tail model fed the full future sample. | Spread credits understated by 33%, which biased the study against the strategy. Its recorded magnitudes needed a re-run. |
| A no-lookahead test with a 21-day blind spot. | Two deliberately leaky forecasters passed it. |
| The paper loop traded stale prices and silently skipped sessions. | 57 runs against about 107 trading days. |

The four options engines that carried every critical and high finding had zero tests.

## Where the audit was wrong

One of the audit's own "verified correct" items failed. The vendor options dataset had been called "real and clean". Forensics later that day showed a modeled implied-volatility surface with fabricated volume and open-interest zeros and formulaic bid-ask widths. A spread-width "measurement" made on it was retracted, and the follow-up study was redesigned to use the data only as a volatility surface.

The evaluation-gate auditor was cancelled mid-run, so the gate had only incidental checks from the other auditors. A follow-up later that day found three statistical bugs those checks had passed.

**Diebold-Mariano test:** It used the wrong kernel for overlapping forecasts. Empirical size at a nominal 5% went from 10.2% to 6.4% once fixed.

**Overfitting probability:** It zero-filled failed sweep configurations. Adding four all-NaN columns moved 8 noise configurations from 0.508 to 0.223.

**Random null:** A "matched" null matched only the sign-switch rate, so a pure-beta position with no timing skill beat it at p = 0.0000. Afterward that case scores p = 0.766, and 4.5% of 150 null draws fall below .05.

## Harness

The guide names two ways to make the harness lie: declaring one trial after sweeping 200 configurations, and passing zero turnover. The harness now warns on both, and its docs say a verdict is conditional on honest inputs it cannot check. After the audit, causality tests catch the planted leaky variants that used to pass.

Every fix got a regression test. The suite grew from 327 to 485 tests, and roughly 70 of them guard the audit's fixes directly. Most of the rest cover a new execution-cost logger. The legacy backtests were simply marked superseded instead of re-scored, because pushing corrupted pipelines through a clean gate would launder them.

## Verdicts

| Recorded claim | After the audit |
|---|---|
| ETF-rotation production weights, OOS Sharpe 3.07 | Look-ahead artifact, superseded. |
| Options-income backtests | Settlement, accounting, and annualization artifacts, superseded. |
| SPY momentum beats a random null (100th percentile, p < 0.001) | 82.5th percentile, p = 0.176: INDISTINGUISHABLE_FROM_NOISE. Overfitting probability 0.604 across 8 lookbacks. |
| Spread-economics study, net Sharpe -0.09 | +0.30 after fixes: INDISTINGUISHABLE_FROM_NOISE, not yet decidable (about 8,100 observations needed, 5,433 available). |
| Volatility forecast: HAR-with-VIX beats VIX itself | Survived: +27.3% QLIKE skill against a random walk, VIX alone -16.1%, signal verified causal end to end. |

No verdict moved in the passing direction. However, one blend test barely survived the corrected kernel (p = 0.048), and that is recorded too.

## Lessons

I think none of the bugs were exotic, because each looked plausible in the code and in the results. An audit's own claims need the same skepticism as the code. A guard is only as good as the fault it has been shown to catch. An earlier 13-agent literature sweep had 21 fabricated claims, which its own fact-check pass caught.

## Limits

The remediation is uncommitted. The 485 figure is the count recorded when it finished, and it is not a fresh run. An audit is not a proof of correctness, and its report lists what stayed unaudited.
