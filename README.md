# Crypto Index Project — Code Companion

PPT: `FAN Hae crypto index.pptx`

Three Jupyter notebooks back the methodology pages in the slides. Each one is self-contained: you can open it on Colab or any local Python with `yfinance`, `pandas`, `numpy`, `matplotlib`, `arch`, `scipy` installed.

## 01_crix_style_basket_construction.ipynb — referenced by slide 10

Builds the basket index. Uses the **liquidity-coverage (LC) rule** from Trimborn & Härdle (2018) to pick how many constituents `K*` belong in the basket each month, rather than fixing K=5 by convention. The K* path is non-constant — that is the point.

- Universe: 20 tokens from yfinance (BTC, ETH, BNB, SOL, XRP, ADA, DOGE, AVAX, DOT, TRX, LINK, LTC, BCH, ATOM, XLM, ETC, NEAR, HBAR, FIL, ALGO).
- Cap proxy: daily close × a snapshot of circulating supply (transparent, defensible for a demo, less BTC-biased than raw yfinance dollar-volume).
- Monthly rebalance, equal weighting inside K*.
- Output: `basket_index.csv`, `fig_k_star_history.png`, `fig_basket_vs_btc.png`.

## 02_index_garch_and_residual_hedge.ipynb — referenced by slides 11 and 13

Two things on top of the basket from notebook 1:

1. **GARCH(1,1)** (Bollerslev 1986) fit to basket returns. Compares conditional vol against a 30-day rolling std — GARCH reacts faster to shocks, which matters for option pricing.
2. **Rolling 90-day OLS hedge** of the basket on (BTC, ETH). The hedge ratios drift through time, and the residual basket vol after the hedge is the economic motivation for the index futures discussion.

- Output: `vol_handoff.csv` (passed to notebook 3), `fig_garch_vs_rolling.png`, `fig_rolling_hedge.png`.

## 03_heston_vs_bs_index_option.ipynb — referenced by slide 14

Prices a 30-day ATM call on the basket under Black-Scholes (three different vol inputs) and under the **Heston (1993)** stochastic-volatility model.

- Heston via characteristic-function integration (Albrecher's "good" formulation, no branch-cut issues), cross-checked with a 50,000-path Monte Carlo.
- Implied-vol smile across strikes 80-120, which BS cannot reproduce.
- Parameters: `v0` from GARCH spot, `theta` from GARCH 30-day forecast, `kappa=3`, `eta=1.5·sqrt(theta)`, `rho=-0.5` (literature-style).

- Output: `fig_heston_vs_bs.png`.

## File map

```
FAN Hae crypto index.pptx
README.md
01_crix_style_basket_construction.ipynb
02_index_garch_and_residual_hedge.ipynb
03_heston_vs_bs_index_option.ipynb
basket_index.csv     (handoff: notebook 1 -> notebook 2)
vol_handoff.csv      (handoff: notebook 2 -> notebook 3)
fig_k_star_history.png
fig_basket_vs_btc.png
fig_garch_vs_rolling.png
fig_rolling_hedge.png
fig_heston_vs_bs.png
```

## What is intentionally not done

- No calibration of Heston parameters against a fitted Deribit surface — that needs a paid Deribit data pull and is a natural next step for the paper.
- No DCC-GARCH or jump-diffusion extension — rolling-window OLS hedge and pure Heston are kept on purpose, to make the figures readable in a slide.
- The "AIC" piece of CRIX is discussed but not implemented; on a 20-token BTC-dominated universe the textbook 2K penalty is degenerate (BTC + ETH already explain >99% of value-weighted return). A proper AIC needs a different likelihood model.
