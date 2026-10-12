# Time Series

> START HERE
```
TIME SERIES DATA ─ plot it · look at the ACF
│
ADF UNIT-ROOT TEST  (the fork)                  ▼ PANEL 1
│
├─ reject H0 ──────► STATIONARY
│                    ├─► the MEAN               ▼ PANEL 2
│                    │     ARIMA (1 series) · VAR (several)
│                    └─► the VARIANCE           ▼ PANEL 4
│                          ARCH / GARCH  (on returns)
│
└─ fail to reject ─► NON-STATIONARY             ▼ PANEL 3
                     ├─ ONE series ─► ROUTE 1  difference
                     │                ─► STATIONARY
                     └─ SEVERAL ─► cointegrated?  (test it)
                                   no  ─► ROUTE 1
                                   yes ─► ROUTE 2  ECM / VECM

EVERY model: AIC/BIC · check residuals          ▼ PANEL 5

legend:  ├─► go deeper   ──► outcome   ▼ jump to panel
```

---

> PANEL 1 — The decision spine

```
TIME SERIES DATA
What: observations ordered in time — today depends on yesterday
Why:  that dependence is the whole subject; it breaks the usual
      "independent errors" assumption of ordinary statistics
How:  plot it → spot trend / season / breaks → look at the ACF
│
├─► UNIT ROOT TEST                     ◄── THIS IS THE FORK
│   What: does a shock die out, or last forever?
│   Why:  it decides your entire modelling path
│   How:  ADF test → H0: unit root present (non-stationary)
│         reject H0 (p < .05)  ──► STATIONARY      ▼ PANEL 2
│         fail to reject       ──► NON-STATIONARY  ▼ PANEL 3
│   │
│   ├─► ADF: AUGMENTED DICKEY–FULLER
│   │   What: the standard unit root test
│   │   Why:  plain DF assumes no autocorrelation — unrealistic
│   │   How:  DF + lagged differences of y; pick the lags by
│   │         AIC/BIC; then pick the form from the plot:
│   │
│   │           series looks like       ADF form
│   │           wanders around zero     none
│   │           wanders around a mean   constant
│   │           trends up or down       constant + trend
│   │
│   └─► KPSS: THE CROSS-CHECK   (H0 reversed: H0 = STATIONARY)
│       Why:  "fail to reject" in ADF is weak evidence —
│             the test has low power
│       How:  read the two tests together
│             ADF           KPSS          verdict
│             reject        don't reject  STATIONARY
│             don't reject  reject        UNIT ROOT
│             otherwise                   inconclusive
│             (then recheck the plot, breaks, ADF form)
│
└─► MODEL SELECTION  ▼ PANEL 5  (every model)
```

---

> PANEL 2 — Stationary branch: models for the MEAN

```
STATIONARY TIME SERIES
What: constant mean and variance; covariance depends only
      on the lag
Why:  shocks are temporary; parameters can be estimated stably
How:  walk the path
      Q1  How many series?
          ONE                  ──► ARIMA
          SEVERAL              ──► VAR
      Q2  Does the spread change?  (calm and wild periods)
          NO                   ──► the mean model is enough
          YES                  ──► variance model ▼ PANEL 4
│
├─► ARIMA(p,d,q) / BOX–JENKINS      (ONE series)
│   What: p = AR lags · d = times differenced · q = MA lags
│   Why:  a disciplined loop instead of guessing orders
│   How:  1. Identify — read the ACF / PACF (below)
│         2. Estimate
│         3. Diagnose — are the residuals white noise?
│            no → go back to 1
│         4. Forecast
│   Note: d = 0 is plain ARMA; d ≥ 1 comes from ▼ PANEL 3
│         seasonal pattern → add seasonal terms (SARIMA)
│   │
│   └─► ACF / PACF   (autocorrelation / partial autocorrelation)
│       What: ACF  = corr(y_t, y_{t-k}) at each lag k
│             PACF = the same, after removing lags 1 … k−1
│       How:  read the pattern
│               ACF         PACF        →  model
│               cuts off q  tails off   →  MA(q)
│               tails off   cuts off p  →  AR(p)
│               tails off   tails off   →  ARMA(p,q)
│               no spikes   no spikes   →  white noise (done)
│
└─► VECTOR AUTOREGRESSION (VAR)      (SEVERAL series)
    What: every variable regressed on lags of itself AND
          all the others
    Why:  univariate ARIMA ignores feedback between series
    How:  choose lag order by AIC/BIC → estimate equation by
          equation → check stability → test Granger-causality
    Note: needs STATIONARY series — otherwise difference them
          (Route 1) or use a VECM (Route 2)  ▼ PANEL 3
```

---

> PANEL 3 — Non-stationary branch: three questions, two routes

```
NON-STATIONARY TIME SERIES
What: mean and/or variance drift over time
Why:  trends, unit roots, breaks — shocks last, so normal
      t / F tests fail
How:  walk the path
      Q1  What kind of drift?
          unit root (wanders)  ──► Q2
          trend only           ──► detrend, then ▼ PANEL 2
          variance grows       ──► take logs, re-run ADF
      Q2  One series or several?
          ONE                  ──► ROUTE 1
          SEVERAL              ──► Q3
      Q3  Cointegrated?  (test: ROUTE 2, steps 1–2)
          NO                   ──► ROUTE 1
          YES                  ──► ROUTE 2, step 3
│
├─► ROUTE 1: DIFFERENCING      (any non-stationary series)
│   What: model Δy_t = y_t − y_{t-1} instead of y_t
│   Why:  makes it stationary → normal inference works again
│   How:  difference → re-run ADF → repeat until it rejects
│         d = number of differences → the series is I(d)
│         d is the "d" in ARIMA(p,d,q) → then ▼ PANEL 2
│         (several series → fit a VAR on the differences)
│   Cost: throws away the long-run level information
│   E.g.: RANDOM WALK  y_t = y_{t-1} + ε_t  is I(1)
│         shocks never decay; best forecast = today's value
│         (plus drift if included); same as ARIMA(0,1,0)
│         finance: prices behave roughly like this ▼ PANEL 4
│
├─► THE TRAP: SPURIOUS REGRESSION      (several series)
│   What: unrelated trending series give high R² and big t
│         statistics
│   Why:  regress levels on levels and you "prove" nonsense
│   How:  warning sign: R² larger than the Durbin–Watson stat
│         the cure is Route 1 or Route 2
│
└─► ROUTE 2: COINTEGRATION     (several I(1) series, one trend)
    What: I(1) series whose linear combination is I(0)
    Why:  keeps the long-run link that differencing destroys
          (e.g. spot & futures prices, a pair of stocks)
          never assume it — test it
    │
    ├─► STEP 1 — are all the series I(1)?
    │     ADF on each one; if not, this route doesn't apply
    │
    ├─► STEP 2 — TEST   (pick by number of variables)
    │                  ENGLE–GRANGER     JOHANSEN
    │     variables    2                 k ≥ 2
    │     finds        one relationship  r relationships
    │     H0           no cointegration  r = 0, then r ≤ 1 …
    │     then fit     ECM               VECM
    │     cointegrated → STEP 3 · not → Route 1
    │
    │     Engle–Granger, step by step
    │       1. OLS of y_t on x_t → residuals ê_t
    │       2. ADF on ê_t  (no constant, no trend)
    │          H0: ê_t has a unit root  (= no cointegration)
    │          reject H0 → cointegrated
    │       Note: use Engle–Granger critical values,
    │             NOT the standard ADF table
    │     Johansen: trace / max-eigenvalue statistics
    │
    └─► STEP 3 — FIT THE MODEL   (only if the test said yes)
        ECM    after Engle–Granger · one equation
          What: short-run changes PLUS a pull back to the
                long-run equilibrium
          How:  Δy_t = γ(y_{t-1} − βx_{t-1}) + short-run terms
                bracket = lagged residual of the Engle–Granger
                regression = last period's "error"
                γ < 0 (between −1 and 0): γ = −0.3 → 30% of
                the gap closes each period
        VECM = VAR + ECM    after Johansen · whole system
          Why:  a VAR on differences omits the long-run term
          How:  VAR in differences + error-correction term(s)
```

---

> PANEL 4 — Volatility & financial time series: models for the VARIANCE

```
VOLATILITY  (financial time series)
What: how widely returns swing — formally σ_t, the std.
      deviation of returns given the past. It CHANGES over time
Why:  risk, portfolios and option prices depend on it, yet
      ordinary models assume the spread is constant
How:  walk the path
      Q1  Prices or returns?   (GARCH needs stationarity)
          prices               ──► make returns  ▼ PANEL 3
          returns              ──► Q2
      Q2  ARCH effects?   (autocorrelated squared residuals)
          test: ARCH-LM, or Ljung–Box on ε²   (H0: no ARCH)
          NO                   ──► constant variance — stop
          YES                  ──► Q3
      Q3  Fit GARCH(1,1); check the standardised residuals
          clean                ──► forecast and use σ_t
          pattern left         ──► upgrade (table below)
│
├─► FOUR WAYS TO MEASURE IT
│   historical   rolling std. deviation of past returns
│   conditional  σ_t from an ARCH / GARCH model   ◄ this panel
│   realised     built from intraday returns
│   implied      backed out of option prices (e.g. VIX)
│
├─► THE SET-UP: TWO EQUATIONS
│   return:    r_t = ln P_t − ln P_{t-1}      (prices → returns)
│   mean:      r_t = μ + ε_t           (μ constant, or an ARMA)
│   variance:  ε_t = σ_t · z_t,   z_t ~ iid(0,1)
│              σ²_t = f(past shocks ε², past variances σ²)
│   ARCH and GARCH are two choices for that last line
│
├─► ARCH ──► GARCH      (the two base models)
│   ARCH(1)     σ²_t = ω + α·ε²_{t-1}
│               driven by the last shock only; real data
│               need many lags → ARCH(q)
│   GARCH(1,1)  σ²_t = ω + α·ε²_{t-1} + β·σ²_{t-1}
│               adds last period's variance → 3 parameters
│               do the work of a long ARCH(q)
│   Read it:    ω = base level · α = reaction to news
│               β = how long old volatility lingers
│   Rules:      ω > 0, α ≥ 0, β ≥ 0, α + β < 1
│               α + β near 1 → shocks fade very slowly
│               long-run variance = ω / (1 − α − β)
│   Link:       ARCH is to ε² what AR is to y; GARCH is to
│               ε² what ARMA is to y  (▼ PANEL 2 logic)
│
├─► WHAT YOU SEE → WHAT TO USE
│   what you see                   what to use
│   volatility clusters         →  ARCH / GARCH
│   fat tails                   →  Student-t / skewed-t errors
│   leverage: falls hit harder  →  GJR-GARCH, EGARCH
│   return rises with risk      →  GARCH-M
│   shocks never fade           →  IGARCH / EWMA
│   several assets together     →  multivariate: DCC, BEKK
│   σ_t has its own randomness  →  stochastic volatility (SV)
│
│   The first three are the STYLISED FACTS of returns.
│   Returns themselves are almost uncorrelated, but SQUARED
│   returns are not — that is why the ARCH test looks at ε².
│   GARCH-M:  r_t = μ + λ·σ²_t + ε_t   (λ > 0)
│   IGARCH:   α + β = 1;  EWMA = IGARCH with ω = 0
│             (RiskMetrics: λ = 0.94 daily)
│
└─► CHECK → FORECAST → USE
    Check:     standardised residuals z_t = ε_t / σ_t (fitted)
               should be white noise with NO ARCH left
               (Ljung–Box on z_t and z²_t)
    Forecast:  variance forecasts drift back to the long-run
               level as the horizon grows
    Use:       Value-at-Risk, Expected Shortfall, position
               sizing, option pricing & hedging — all scale
               with σ_t
```

---

> PANEL 5 — Cross-cutting: choose and check every model

```
MODEL SELECTION & CHECKING     (applies to every model)
│
├─► AIC · BIC · HQ    (HQ = Hannan–Quinn)
│   What: fit scores with a penalty for extra parameters
│   Why:  more lags ALWAYS fit better in-sample — you need
│         a tie-breaker
│   How:  estimate several candidates → pick the LOWEST score
│
│           score  penalty per parameter  leans toward
│           AIC    2                      bigger models
│           HQ     2·ln(ln n)             in between
│           BIC    ln n                   smaller models
│                  (n = sample size)
│
│   Used in: ADF lags · Box–Jenkins (p,q) · VAR lag order ·
│            ECM · VECM · GARCH orders
│
└─► RESIDUAL CHECK
    What: are the leftovers pure noise?
    Why:  a leftover pattern means the model missed something
    How:  Ljung–Box test on the residuals
          H0: no autocorrelation left
          same test on SQUARED residuals → leftover ARCH
```

---

> CHEAT SHEET — tests, symbols, golden rules

```
TEST            H0                          REJECT H0 →
ADF             unit root                   stationary
KPSS            stationary                  unit root
Engle–Granger   no cointegration            cointegrated
Johansen        no cointegration (r = 0)    ≥ 1 relationship
Ljung–Box       no autocorrelation          pattern left over
ARCH-LM         no ARCH effects             fit ARCH / GARCH

Decision rule everywhere:  p < .05 → reject H0
```

```
SYMBOLS
y_t        value at time t
y_{t-1}    previous value (lag 1)
Δy_t       first difference  y_t − y_{t-1}
ε_t        shock (error)
σ²_t       variance at time t
r_t        return  ln P_t − ln P_{t-1}
I(d)       needs d differences to become stationary
H0         null hypothesis
```

```
GOLDEN RULES
1. Unit-root test first — it decides the path.
2. Never regress non-stationary levels blindly (spurious!).
3. Never assume cointegration — test it; if absent, difference.
4. GARCH models returns, not prices.
5. Pick lags / orders with AIC/BIC, then check the residuals.
```
