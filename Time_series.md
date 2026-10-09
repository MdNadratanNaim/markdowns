> PANEL 1 — The decision spine

```text
TIME SERIES DATA
What: observations ordered in time, so today depends on yesterday
Why:  that dependence is the whole subject — it breaks normal stats
How:  plot it, decompose trend/season, look at the ACF
│
├─► UNIT ROOT TEST
│   What: test whether a shock is permanent or dies out
│   Why:  THIS IS THE FORK. It decides your entire modelling path
│   How:  ADF test → H0: unit root present (non-stationary)
│         reject H0 (p < .05)  ──► STATIONARY      ▼ PANEL 2
│         fail to reject       ──► NON-STATIONARY  ▼ PANEL 3
│
│   AUGMENTED DICKEY–FULLER (ADF)
│   What: the standard unit root test
│   Why:  plain DF assumes no autocorrelation — unrealistic
│   How:  DF + lagged differences of y on the RHS to soak up
│         autocorrelation; pick lags by AIC/BIC
│
└─► MODEL SELECTION CRITERIA  ▼ PANEL 4 (applies to every model)
```

---

> PANEL 2 — Stationary branch

```text
STATIONARY TIME SERIES
What: mean, variance, autocovariance all constant over time
Why:  shocks are temporary; parameters can be estimated stably
How:  ADF rejects unit root, or difference until it does
│
├─► AUTOCOVARIANCE FUNCTION  (ACF / PACF)
│   What: cov(y_t, y_{t-k}) plotted against lag k
│   Why:  it IS the memory of the series — tells you which lags matter
│   How:  ACF cuts off → MA(q); ACF decays & PACF cuts off → AR(p)
│   │
│   └─► BOX–JENKINS / ARIMA
│       What: univariate workflow, model = ARIMA(p,d,q)
│       Why:  gives a disciplined loop instead of guessing orders
│       How:  Identify (ACF/PACF) → Estimate → Diagnose residuals
│             → Forecast; iterate if residuals aren't white noise
│
├─► VECTOR AUTOREGRESSION (VAR)
│   What: every variable regressed on lags of itself AND all others
│   Why:  univariate ARIMA ignores feedback between series
│   How:  choose lag order by AIC/BIC, estimate equation by equation,
│         then check stability and Granger-causality
│
└─► ARCH ──► GARCH ──► VOLATILITY EQUATION
    What: models of the VARIANCE, not the level
    Why:  volatility clusters — calm periods and wild periods group together
    How:
      ARCH   σ²_t = α₀ + α₁·ε²_{t-1}
             uses past squared shocks; needs many lags in practice
      GARCH  σ²_t = ω + α·ε²_{t-1} + β·σ²_{t-1}
             adds past variances → far fewer parameters, same fit
      Volatility equation = the σ²_t line above. You model the mean
             equation (e.g. ARIMA) and the variance equation together.
```

---

> PANEL 3 — Non-stationary branch

```text
NON-STATIONARY TIME SERIES
What: mean and/or variance drift over time
Why:  trends, unit roots, structural breaks — normal t/F tests fail
How:  detect from plots + ADF; then pick ONE of the two routes below
│
├─► RANDOM WALK MODEL
│   What: y_t = y_{t-1} + ε_t
│   Why:  simplest unit-root process; shocks never decay
│   How:  best forecast is today's value (plus drift if included)
│   └─► it is the special case of an integrated series with d = 1
│
├─► INTEGRATED TIME SERIES  I(d)
│   What: needs differencing d times to become stationary
│   Why:  d is exactly the "d" in ARIMA — tells you how much to difference
│   How:  difference, re-run ADF, repeat until it rejects
│   │
│   ├─► ROUTE 1: DIFFERENCING
│   │   What: model Δy_t instead of y_t
│   │   Why:  makes it stationary, so normal inference works again
│   │   How:  difference d times, then go to ▼ PANEL 2 (ARIMA/VAR)
│   │   Cost:  throws away the long-run level information
│   │
│   └─► ROUTE 2: COINTEGRATION
│       What: two or more I(1) series whose linear combination is I(0)
│       Why:  keeps the long-run relationship that differencing destroys
│       How:  TEST IT FIRST — Engle–Granger or Johansen (see below)
│       │
│       ├─► ENGLE–GRANGER TEST
│       │   What: two-step cointegration test (usually for 2 variables)
│       │   Why:  simplest way to check if Route 2 is even available;
│       │         you must not assume cointegration — you must verify it
│       │   How:  Step 1: regress y_t on x_t by OLS → get residuals ê_t
│       │         Step 2: run ADF on ê_t (no constant, no trend)
│       │         Reject H0 of "no cointegration" → series ARE cointegrated
│       │         Fail to reject → NOT cointegrated → fall back to Route 1
│       │   Notes: H0 here is NO cointegration (opposite of usual ADF use)
│       │          Critical values are Engle–Granger's, NOT standard ADF
│       │          Only one cointegrating vector assumed
│       │   Alternative: JOHANSEN test — handles k > 2 variables and
│       │                can find multiple cointegrating vectors
│       │
│       └─► ERROR CORRECTION MECHANISM (ECM)
│           What: models short-run changes PLUS a pull back to long-run eq.
│           Why:  reconciles short-run dynamics with long-run balance
│           How:  Δy_t = γ(y_{t-1} − βx_{t-1}) + short-run terms
│                 γ is negative → the "error" gets corrected each period
│                 The (y_{t-1} − βx_{t-1}) term is the lagged residual
│                 from the Engle–Granger regression
│           │
│           └─► VECM = VAR + ECM
│               What: multivariate ECM for a cointegrated system
│               Why:  plain VAR on differenced data would omit the
│                     long-run term and be misspecified
│               How:  VAR in differences + the error-correction term
│
└─► SPURIOUS REGRESSION
    What: unrelated trending series give high R² and big t-stats
    Why:  it's the trap you fall into if you ignore non-stationarity —
          you'd "prove" nonsense relationships
    How:  avoid it by Route 1 (difference to stationarity) or
          Route 2 (Engle–Granger/Johansen → if cointegrated, use ECM/VECM)
```

---

> PANEL 4 — Cross-cutting

```text
MODEL SELECTION CRITERIA  (AIC, BIC, HQ)
What: goodness-of-fit scores with a penalty for extra parameters
Why:  more lags ALWAYS fit better in-sample; you need a tie-breaker
How:  estimate several candidate models, pick the lowest score
      BIC penalises complexity harder than AIC
Used in: Box–Jenkins (p,q) · VAR (lag order) · VECM · ECM · GARCH
```

---

> The whole thing in six lines

```text
1. Test for a unit root (ADF).
2. Stationary?      → ACF/PACF → ARIMA, or VAR, or ARCH/GARCH for variance.
3. Non-stationary?  → difference → back to step 2 (ARIMA), OR
4. Check cointegration with ENGLE–GRANGER (2 vars) or JOHANSEN (k vars).
5. Cointegrated?    → ECM → VECM.  Not cointegrated? → Route 1 (difference).
6. Never regress non-stationary levels blindly — that's spurious regression.
7. Use AIC/BIC at every step to choose lags and orders.
```