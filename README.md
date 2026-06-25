# River Lifelines

**Forecasting Amazon river navigability under climate regime shift — with calibrated, distribution-free uncertainty.**

![Status](https://img.shields.io/badge/status-work%20in%20progress-orange)
![Phase](https://img.shields.io/badge/phase-v1%20applied%20ML-blue)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)

> When a river is the only road, predicting *when it closes* is the difference between supplies arriving and a community being cut off. In that setting, a forecast without honest uncertainty is worse than no forecast at all.

---

## Table of Contents

- [Overview](#overview)
- [Motivation](#motivation)
- [Research Question (v1)](#research-question-v1)
- [Approach](#approach)
- [Data](#data)
- [Installation](#installation)
- [Usage](#usage)
- [Repository Structure](#repository-structure)
- [Results](#results)
- [Roadmap](#roadmap)
- [Mathematical & Technical Foundations](#mathematical--technical-foundations)
- [Scope & Responsible Use](#scope--responsible-use)
- [Citation](#citation)
- [References](#references)

---

## Overview

River Lifelines is an applied machine-learning study that forecasts when a river reach will cross a critical navigability threshold, and expresses that forecast as a **calibrated probability with finite-sample coverage guarantees** rather than a single point estimate.

The scope is deliberately lean. The project builds and demonstrates rigorous ML fundamentals — time-series modeling, regularization, time-aware validation, and uncertainty quantification — on a real, high-stakes problem. The full physics-informed (SciML) treatment of the broader *water → isolation → health → logistics* nexus, using PINNs, Neural ODEs, and graph methods, is pursued separately as a later research horizon (see the [SciML lab]()). **This repository is the foundation on which that work builds.**

## Motivation

Across the *Amazônia Legal*, hundreds of communities are reachable only by river. Medicine, supplies, and medical personnel travel by boat; there is no road. When the water drops below a navigable level, those communities are cut off — and recent climate-driven droughts have pushed Amazon rivers to historic lows exactly when health crises tend to spike.

Conventional forecasting struggles here for three specific reasons:

1. **Regime shift.** The climate's data-generating process is no longer stationary; the historical record has partly lost its predictive validity.
2. **Scarce, noisy data.** Gauge coverage is sparse and observations are irregular.
3. **Asymmetric cost of error.** A wrong forecast can mean medicine that never arrives, so a single point estimate is irresponsible.

This project isolates the **trigger** — river navigability — and asks whether it can be forecast honestly.

## Research Question (v1)

> Can we forecast, weeks ahead, when a river reach will cross a critical navigability threshold — and express it as a calibrated probability with distribution-free coverage guarantees, rather than a single number?

Where error is costly and asymmetric, the uncertainty *is* the product. "The river closes in week 6" is a guess. "85% probability of dropping below the critical level between weeks 5 and 7" is something a planner can act on.

## Approach

Each step is a baseline the next must beat.

1. **Naive baselines** — persistence and seasonal-naive forecasts. Any learned model must outperform these.
2. **Regularized regression** — engineered features (lagged stage, cumulative upstream rainfall, ENSO index) with ridge and lasso; regression framed as projection, regularization to control the bias–variance trade-off under data scarcity.
3. **Calibrated uncertainty (conformal prediction)** — distribution-free prediction intervals with finite-sample coverage guarantees, robust precisely because they do not assume the future is distributed like the past.
4. **Honest validation** — time-aware, strictly out-of-sample splits. Reported metrics include error against baselines, empirical interval coverage, and sharpness.

**Scientific spine:** train on the pre-shift record, test on a recent extreme drought, and *measure* where and how the model fails — making the regime shift visible rather than assumed — then evaluate mitigations.

## Data

All sources are open and Brazilian or public-domain.

| Source | Provider | Role |
|---|---|---|
| Historical stage, discharge, rainfall | ANA HidroWeb / SNIRH | Primary signal (national gauge network; the Manaus stage record spans over a century) |
| ENSO indices | NOAA CPC | Exogenous climate driver |
| Health & logistics (DATASUS / SIVEP) | Ministério da Saúde | Reserved for v3 |

Stage, discharge, and rainfall series are downloadable as CSV and via the SNIRH public web service.

## Installation

```bash
git clone https://github.com/<your-username>/river-lifelines.git
cd river-lifelines
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Requires Python 3.10+.

## Usage

```bash
# 1. Ingest and assemble the dataset for a single gauging station
python -m src.data.build_dataset --station <station_id>

# 2. Fit baselines and the regularized model with conformal intervals
python -m src.models.train --config configs/v1.yaml

# 3. Backtest on the held-out drought period; report coverage and sharpness
python -m src.evaluation.backtest --config configs/v1.yaml
```

The entry points above are representative; adjust them to match the modules in `src/`.

## Repository Structure

```
river-lifelines/
├── data/
│   ├── raw/              # downloaded ANA / NOAA series (gitignored)
│   └── processed/        # cleaned, feature-ready tables
├── notebooks/            # exploration, diagnostics, figures
├── src/
│   ├── data/             # ingestion from ANA HidroWeb, NOAA CPC
│   ├── features/         # lagged stage, cumulative rainfall, ENSO
│   ├── models/           # baselines + regularized regression
│   ├── conformal/        # MAPIE wrappers for prediction intervals
│   └── evaluation/       # coverage, sharpness, time-aware backtesting
├── configs/              # experiment configuration
├── tests/
├── requirements.txt
└── README.md
```

## Results

> _To be reported._ Metrics will include error relative to the naive baselines, empirical coverage of the conformal intervals against the nominal level, and interval sharpness — all evaluated out-of-sample on a recent drought period.

## Roadmap

| Stage | Scope | Phase |
|---|---|---|
| **v1 — Single reach** | One gauging station; navigability forecast with calibrated intervals | Applied ML — *this repo, active* |
| **v2 — River network** | Multiple stations; the network as a directed graph | Transition toward SciML |
| **v3 — The nexus** | Couple navigability → isolation → health → logistics | Future SciML research horizon |

v2 and v3 are the long view, pursued once the ML foundations are solid.

## Mathematical & Technical Foundations

These foundations are self-taught — built from open coursework and primary references rather than a formal degree.

**Mathematics.** Numerical linear algebra (least squares as projection, conditioning), probability and statistics (MLE, the bias–variance decomposition, the coverage arguments behind conformal prediction), time series, and regularization.

**Stack.** `Python` · `pandas` · `numpy` · `scikit-learn` · `statsmodels` · `MAPIE` (conformal prediction) · `matplotlib`

## Scope & Responsible Use

This is a **feasibility study and research prototype**, not a deployed decision system. It investigates whether methods robust to scarcity and regime shift *could* support logistic-sanitary planning with honest uncertainty. It is **not** validated for operational routing of medical supplies or for any clinical or emergency decision. Real-world use would require partnership with domain authorities and field validation.

## Citation

If you reference this work, please cite it as:

```bibtex
@misc{riverlifelines,
  author       = {<Your Name>},
  title        = {River Lifelines: Forecasting Amazon River Navigability under Climate Regime Shift},
  year         = {2026},
  howpublished = {\url{https://github.com/<your-username>/river-lifelines}}
}
```

## References

- ANA — Agência Nacional de Águas e Saneamento Básico. *HidroWeb / SNIRH.* <https://www.snirh.gov.br/hidroweb>
- NOAA Climate Prediction Center. *ENSO Monitoring & Indices.* <https://www.cpc.ncep.noaa.gov>
- Angelopoulos, A. N., & Bates, S. *A Gentle Introduction to Conformal Prediction and Distribution-Free Uncertainty Quantification.*
- Taquet, V., et al. *MAPIE: Model-Agnostic Prediction Interval Estimator.* <https://github.com/scikit-learn-contrib/MAPIE>

---

*Status: work in progress — **v1** (applied ML foundation).*
