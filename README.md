# Implied Volatility Surface Engine

Builds an implied volatility surface from live SPY option chains. The forward curve is recovered from put-call parity rather than assuming a dividend yield, implied volatilities are inverted from Black-76 using Brent's method, and a 5-parameter SVI smile is calibrated to each expiration slice via liquidity-weighted multi-start optimization. The fitted surface is then screened for both static arbitrage conditions: butterfly and calendar.

Black-76, the implied volatility solver, the SVI fitter, and both arbitrage tests are implemented from scratch. No options pricing library is used.

## Calibrated Surface

![SVI Volatility Surface](surface_3d.png)

Red points are market quotes. The surface is the SVI fit.

## Snapshot

| | |
|---|---|
| Underlying | SPY |
| Surface points inverted | [N] |
| Slices calibrated | [N] |
| Median fit error | [X] vol points |
| Butterfly violations | [N] |
| Calendar violations | [N] |

## Fit Quality

Market quotes against the calibrated SVI curve, one panel per expiration. Fit error is reported in volatility points per slice.

![Volatility Smiles](smile_fits.png)

The downward slope from left to right is the equity index skew: out-of-the-money puts trade at higher implied volatility than out-of-the-money calls, because index options are priced for crash risk rather than symmetric moves.

## Arbitrage Screening

A fitted surface can look clean and still be untradeable. Two conditions must hold.

**Butterfly.** Within a single expiration, the implied risk-neutral density must be non-negative. Gatheral's condition requires `g(k) >= 0` at every log-moneyness. A violation means a butterfly spread could be purchased for negative cost.

![Butterfly Condition](butterfly_gk.png)

**Calendar.** Across expirations, total implied variance must be non-decreasing in maturity at fixed log-moneyness. If the curves cross, a calendar spread is free money.

## Pipeline

1. Pull SPY option chains across expirations, filtered on bid, relative spread, and open interest
2. Recover the forward per expiry from put-call parity across matched near-the-money strikes
3. Invert Black-76 for implied volatility, out-of-the-money quotes only
4. Calibrate raw SVI per slice: `w(k) = a + b[rho(k-m) + sqrt((k-m)^2 + sigma^2)]`, multi-start SLSQP weighted by open interest
5. Screen for butterfly and calendar arbitrage on a 121-point log-moneyness grid

## Design Notes

**Forwards from parity, not assumption.** Rather than plugging in a dividend yield, the forward is backed out of `C - P = e^{-rT}(F - K)` across matched strikes and taken as the median over near-the-money pairs. The market's implied forward already contains the dividend and financing assumptions.

**Out-of-the-money quotes only.** OTM contracts carry the tightest spreads. It also avoids the American early-exercise bias that would distort deep-in-the-money inversions, since SPY options are American while the model is European.

**Validation.** The numerical core was tested against a synthetic arbitrage-free chain generated from known SVI parameters: forwards recovered to four decimal places, implied volatilities to 1e-9, sub-basis-point calibration error, zero false-positive arbitrage flags, and a passing negative control on a deliberately inadmissible slice.

## Known Limitations

- **Snapshot only.** Free data provides a single point-in-time chain, so surface dynamics cannot be tracked. Historical options data would enable skew and term-structure time series.
- **European pricing on American options.** The early-exercise premium on a low-dividend ETF is small, but it is not zero.
- **Flat discount curve.** A single short-rate proxy stands in for the full curve.

## Stack
Python, NumPy, pandas, SciPy, matplotlib, yfinance
