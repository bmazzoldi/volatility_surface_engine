# volatility_surface_engine
Implied volatility surface construction for SPY options: put-call parity forwards, Black-76 inversion, SVI calibration, and static arbitrage screening. Python.
Builds an implied volatility surface from live SPY option chains. The forward curve is recovered from put-call parity rather than assuming a dividend yield, implied volatilities are inverted from Black-76 using Brent's method, and a 5-parameter SVI smile is calibrated to each expiration slice via liquidity-weighted multi-start optimization. The fitted surface is then screened for both static arbitrage conditions: butterfly (Gatheral's g(k) >= 0) and calendar (total implied variance non-decreasing in maturity).

Black-76, the implied volatility solver, the SVI fitter, and both arbitrage tests are implemented from scratch. No options pricing library is used.
