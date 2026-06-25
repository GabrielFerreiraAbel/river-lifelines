# River Lifelines

**Forecasting Amazon river navigability under climate regime shift — with honest, calibrated uncertainty.**

> When a river is the only road, predicting *when it closes* is not a convenience — it is the difference between supplies arriving and a community being cut off. And in that setting, a forecast without honest uncertainty is worse than no forecast at all.

---

## The Problem

Across the *Amazônia Legal*, hundreds of communities are reachable only by river. Medicine, hospital supplies, vaccines, and medical personnel travel by boat — there is no road, no runway. When the water drops below a navigable level, those communities are isolated.

In recent years, climate-driven droughts have pushed Amazon rivers to historic lows, stranding populations at the exact moment that shifting temperatures fuel epidemiological outbreaks (malaria, dengue, respiratory disease). The logistic crisis and the health crisis arrive together, and they reinforce each other.

Conventional forecasting and off-the-shelf machine learning struggle here for three specific reasons:

1. **Regime shift.** The climate's data-generating process is no longer stationary. Models trained on the historical record systematically misjudge the new extremes — the past has partially lost its predictive validity.
2. **Data scarcity and noise.** Health and logistics data for these regions are sparse, irregular, and noisy.
3. **Asymmetric cost of error.** A wrong forecast can mean medicine that never arrives. There is little room for "try and see," and a single point estimate ("the river closes in week 6") is, by itself, irresponsible.

This project does not pretend to solve that entire crisis. It isolates the **trigger** — river navigability — and asks whether it can be forecast *honestly*.

---

## The Scientific Question (v1)

> Can we forecast, weeks in advance, when a river reach will cross a critical navigability threshold — and express that forecast as a calibrated probability with distribution-free coverage guarantees, rather than a single number?

The emphasis is deliberate. In a setting where the cost of error is severe and asymmetric, **the uncertainty is the product.** "The river closes in week 6" is a guess. "There is an 85% probability the reach drops below the critical level between weeks 5 and 7" is something a planner can act on.

---

## Why This Is Hard (and Worth Doing)

The interesting difficulty is not the model — it is the **non-stationarity**. If the future looked like the past, this would be a routine time-series problem. It does not. The scientific spine of v1 is to confront that head-on:

- Train a model on the pre-shift historical record.
- Evaluate it on a recent extreme drought it has never seen.
- **Measure, quantitatively, how and where it fails** — making the regime shift visible rather than assumed.
- Then evaluate mitigations: recency-weighted training, regime-aware features, and the natural adaptivity of conformal prediction.

This turns "ML fails under climate change" from a slogan into a measured result — which is the kind of honest, falsifiable claim that distinguishes research from a demo.

---

## Approach

The method builds in steps, each one a baseline the next must beat.

**1. Naive baselines.** Persistence and seasonal-naive forecasts. Any learned model must demonstrably outperform these, or it earns nothing.

**2. Regularized regression.** Engineered hydro-climatic features — lagged river stage, cumulative upstream rainfall, and the ENSO (El Niño–Southern Oscillation) index, which is strongly correlated with Amazon water levels — fed into ridge/lasso regression. Regression here is, at its core, a projection onto a feature subspace; the regularization controls the bias–variance trade-off under scarce, noisy data.

**3. Calibrated uncertainty via conformal prediction.** Instead of trusting a model's nominal confidence, conformal prediction wraps any forecaster to produce prediction intervals with finite-sample, distribution-free coverage guarantees. This is the layer that makes the output trustworthy under the asymmetric-cost setting — and it is robust precisely because it does not assume the future is distributed like the past.

**4. Honest validation.** Strictly out-of-sample, time-aware splits (no leakage from future to past). Reported metrics include forecast error against baselines, **empirical coverage of the prediction intervals** (do the 85% intervals actually contain the truth 85% of the time?), and interval sharpness.

---

## Roadmap

This is designed to start small and grow. Each stage is a complete, self-contained artifact.

| Stage | Scope | New ingredient |
|---|---|---|
| **v1 — Single reach** | One gauging station; forecast navigability threshold crossing with calibrated intervals | Conformal prediction; the regime-shift experiment |
| **v2 — River network** | Multiple stations along a basin; a river network is naturally a directed graph (upstream → downstream) | Spatial structure; graph-based propagation of state and uncertainty |
| **v3 — The nexus (research horizon)** | Couple navigability → isolation → overlaid health signals → logistics, as a multi-year proof of concept | Data fusion across hydrology, health (DATASUS/SIVEP), and logistics |

v2 and v3 are the research trajectory, not promises of v1.

---

## Data

All v1 data is open and Brazilian.

- **River stage, discharge, and rainfall — ANA HidroWeb / SNIRH.** Historical daily series across a national network of fluviometric and pluviometric stations, downloadable as CSV and accessible through a public web service (station code, date range, data type, raw vs. consistent). The Manaus stage record extends back to the early twentieth century — a century-plus series, ideal for studying regime change.
- **ENSO indices — NOAA Climate Prediction Center.** Exogenous driver of Amazon hydrology.
- **Health and logistics data (DATASUS / SIVEP, fluvial logistics)** — reserved for v3.

> Note: ANA also mirrors station series in an open data repository, which is convenient for programmatic, reproducible ingestion.

---

## Mathematical Foundations

The project is small in scope but rests on real foundations:

- **Linear algebra** — least-squares regression as orthogonal projection; conditioning and the geometry of the feature space.
- **Probability & statistics** — maximum likelihood, the bias–variance trade-off, and the exchangeability/coverage arguments underlying conformal prediction.
- **Time-series & regularization** — feature lags, non-stationarity, and ridge/lasso as controlled-complexity estimators.

---

## Tech Stack

`Python` · `pandas` · `numpy` · `scikit-learn` · `statsmodels` · `MAPIE` (conformal prediction) · `matplotlib`

---

## Repository Structure (planned)

```
river-lifelines/
├── data/                # raw + processed (gitignored; fetch scripts versioned)
├── notebooks/           # exploratory analysis, regime-shift experiment
├── src/
│   ├── ingest/          # ANA HidroWeb fetch + cleaning pipeline
│   ├── features/        # lagged stage, cumulative rainfall, ENSO alignment
│   ├── models/          # baselines, regularized regression, conformal wrapper
│   └── eval/            # time-aware splits, coverage + sharpness metrics
├── results/             # figures, validation tables, write-up
└── README.md
```

---

## Scope & Responsible Use

This is a **feasibility study and research prototype**, not a deployed decision system. It is intended to investigate whether methods robust to data scarcity and regime shift *could* support logistic-sanitary planning — and to do so with honest uncertainty. It is not validated for, and must not be used as, an operational tool for routing medical supplies or making clinical or emergency decisions. Real-world impact would require partnership with domain authorities, field validation, and institutional oversight.

---

## References

- Agência Nacional de Águas e Saneamento Básico (ANA) — HidroWeb / SNIRH (open hydrometeorological series).
- A. Angelopoulos & S. Bates — *A Gentle Introduction to Conformal Prediction and Distribution-Free Uncertainty Quantification* (2021).
- V. Vovk, A. Gammerman & G. Shafer — *Algorithmic Learning in a Random World* (conformal prediction foundations).
- Literature on Amazon hydroclimate variability and its coupling to ENSO.

---

## Status

Work in progress — **v1**.
