# Agents auditing agents: a five-agent audit of agent-written backtesting code

*Status: local repository. Claude Code agents wrote the code. I led the audit and decide what counts as a result. The remediation described below exists in my working tree but is not yet committed, so treat it as in progress until it is. This write-up contains no personal financial information.*

**At a glance**
- Four adversarial code auditors and one primary-source researcher audited the codebase. Every number had to be reproduced by running code or extracted from a primary source.
- Caught in agent-written code: a look-ahead that inflated a Sharpe ratio from 1.00 to 3.29, full-sample regime leakage into every regime-conditioned backtest, in-the-money options expiries booked as maximum profit, and a CVaR (average loss in the worst 5% of days) sign bug that understated tail risk about 17-fold.
- The evaluation harness, the repo's mandatory gate, was itself audited and repaired. Regression tests grew from 327 to 485 (in my working tree, uncommitted).
- Verdicts moved toward "no evidence," and the audit itself had to be corrected the same day.

## Why an audit

OpenStock is a Python research platform for equities and options. Over three generations of work, agents wrote strategies, backtest engines, a volatility-forecasting module and, in August, an evaluation harness whose job is to tell a real result from a lucky one: cost curves, a paired bootstrap against a benchmark, an exposure-matched random null, deflated Sharpe, and a verdict that can say NO_EVIDENCE. The repo's agent guide carries one rule: nothing counts as a result until that harness says so. Then I pointed agents at the whole repository, including, after a false start, the harness itself.

## Design

Five agents. Four auditors took adversarial lanes: the paper-trading loop, the legacy backtest engines and experiments, the volatility module, and the evaluation gate. A fifth pulled primary sources for external claims. The report's standard was strict: each finding carries a file-and-line anchor, every number was reproduced by running code or extracted from a primary source, and nothing is quoted from docstrings or vendor material without a flag. It also lists what was verified correct, not just what broke: Black-Scholes against Hull to four decimals, put-call parity to 1e-15, equity-engine positions exactly equal to the one-bar-shifted signal, and purge and embargo clean on all 10 cross-validation splits.

## What it caught

| Finding | Effect |
|---|---|
| Look-ahead: the script that seeded production weights multiplied today's signal by today's return, unshifted. The shared engine was right; the script bypassed it. | Sharpe 3.29 vs 1.00 shifted (3.16 vs 0.94 on a second component). The reported "OOS Sharpe 3.07" was about 3x inflation. |
| Regime leakage: a train-only regime-model (HMM) fit was computed, discarded, and refit on the full sample, then decoded across the whole path. A second wrapper overwrote its own expanding window. | Every "with regime" result leaked. "The overlay prevented the COVID drawdown" was the leak's signature. |
| Settlement and accounting: in-the-money expiries settled at $0, Sharpe and drawdown came from realized P&L only, entry credit never reached cash, annualization was wrong in both directions. | "0.00% max drawdown, 97-100% win rate" were artifacts; a +4.09%/yr wheel result corrects to about +1.9%. |
| CVaR sign bug: a negated quantile selected about 95% of the sample instead of the worst 5%. | Tail risk understated about 17x. |
| Volatility module and its study: a day-count mismatch, expected shortfall collapsing to zero when tail probability was small, and a tail model fed the full future sample. | Spread credits 33% understated, biasing the study against the strategy; its recorded magnitudes needed a re-run. |
| A no-lookahead test with a 21-day blind spot. | Two deliberately leaky forecasters passed it. |
| The paper loop traded stale prices and silently skipped sessions. | 57 runs against about 107 trading days. |

Zero tests existed for the four options engines that carried every critical and high finding.

## Where the audit was wrong

One of its own "verified correct" items failed. The vendor options dataset had been called "real and clean." Forensics later that day showed a modeled implied-volatility surface with fabricated volume and open-interest zeros and formulaic bid-ask widths. A spread-width "measurement" made on it was retracted in the audit record, and the follow-up study was redesigned to use the data only as a volatility surface.

The evaluation-gate auditor was cancelled mid-run, and the other auditors verified pieces of the gate only incidentally. The follow-up ran later that day and found three statistical bugs the incidental checks had passed:

- A Diebold-Mariano forecast test used the wrong kernel for overlapping forecasts. Empirical size at a nominal 5% went from 10.2% to 6.4% once fixed.
- A probability-of-backtest-overfitting calculation zero-filled failed sweep configurations. Adding four all-NaN columns moved 8 noise configurations from 0.508 to 0.223.
- A "matched" random null matched only the sign-switch rate. A pure-beta position with no timing skill beat it at p = 0.0000. Afterward that case scores p = 0.766, and 4.5% of 150 null draws fall below .05.

## The harness as a mandatory gate

The guide names two ways to make the harness lie: declaring one trial after sweeping 200 configurations, and passing zero turnover. The harness warns on both, and after the audit its docs say plainly that a verdict is conditional on honest inputs it cannot check. Guards had to be able to fail. Causality tests perturb the future and assert the past is bit-identical, and after the audit they catch the planted leaky variants that used to pass. Every fix got a regression test, including calibration tests that draw from a known null. The suite grew from 327 to 485 tests, roughly 70 of them guarding the audit's fixes directly and most of the rest covering a new execution-cost logger. The plan marked the legacy backtests superseded instead of re-scoring them: pushing corrupted pipelines through a clean gate would launder them.

## Where the verdicts moved

| Recorded claim | After the audit |
|---|---|
| ETF-rotation production weights, OOS Sharpe 3.07 | Look-ahead artifact; superseded |
| Options-income backtests | Settlement, accounting, and annualization artifacts; superseded |
| SPY momentum beats a random null (100th percentile, p < 0.001) | 82.5th percentile, p = 0.176: INDISTINGUISHABLE_FROM_NOISE. Overfitting probability 0.604 across 8 lookbacks. |
| Spread-economics study, net Sharpe -0.09 | +0.30 after fixes; verdict INDISTINGUISHABLE_FROM_NOISE, not yet decidable (about 8,100 observations needed, 5,433 available) |
| Volatility forecast: HAR-with-VIX beats VIX itself | Survived: +27.3% QLIKE skill against a random walk, VIX alone -16.1%, signal verified causal end to end |

No verdict moved in the passing direction. One blend test barely survived the corrected kernel (p = 0.048), and that is recorded too.

## What I take from it

None of the bugs were exotic. Each looked plausible in the code and in the results, and adversarial framing plus mandatory reproduction is what found them. An audit's own claims need the same skepticism as the code. Incidental verification is not verification. A guard is only as good as the fault it has been shown to catch. An earlier 13-agent literature sweep had 21 fabricated claims caught by its own fact-check pass.

## Status and limits

The remediation is uncommitted. The 485 figure is the count recorded when the remediation finished, not a fresh run. An audit is not a proof of correctness, and its report lists what stayed unaudited.
