# Powering the Last Mile: Targeting Off-Grid Solar Customers in Nigeria

Ranking Nigerian households by willingness to pay for solar devices — so a pay-as-you-go solar company can spend its outreach budget where conversion is most likely. Built on World Bank Multi-Tier Framework (MTF) survey microdata, including its embedded randomized-price experiment.

## Headline Result

| Targeting strategy | Conversion rate (test set) |
|---|---|
| Random outreach | 29.0% |
| Model's top-20% contact list | **57.9%** |

**Lift: 2.0×** — the model doubles the productivity of every unit of outreach budget.

## The Problem

Off-grid solar companies and electrification programs in Nigeria face expensive last-mile customer acquisition: field visits cost real money, and most contacted households don't convert. This project builds a ranking model that identifies which unelectrified and under-electrified households are most likely to purchase a solar device if reached.

## The Data

**Nigeria Multi-Tier Framework (MTF) Energy Access Survey, 2018** — World Bank/ESMAP household survey covering 3,669 households in North-West Nigeria, distributed as per-section Stata files with full codebook and questionnaires.

- Source: [World Bank Microdata Catalog](https://microdata.worldbank.org/index.php/catalog/3865)
- Raw data is **not redistributed** in this repository; the ingestion notebook documents how to obtain and extract it.

A notable design feature: the survey embedded a **randomized willingness-to-pay experiment** — each household was offered a solar device at a randomly selected price. This gives the price variable experimentally grounded predictive power and gives the model a demand-curve interpretation.

## Method

1. **Target definition.** Willingness to pay upfront for a solar device at the randomly offered price (`E_3`; 29% positive). Current solar ownership was rejected as a target: at 1.2% prevalence it is both severely imbalanced and mismatched with the business question (a vendor seeks *future* adopters, not current owners).
2. **Feature assembly across survey sections**, merged on household ID with unit-of-analysis checks (person-level rosters aggregated or filtered to household heads before merging): offered price, urban/rural residence, household head's education, grid connection, household size, formal bank account, mobile money account.
3. **Culturally grounded recoding.** Education codes include Quranic schooling — the most common category in this Northern sample, and not a rung on the formal-education ladder. It is modeled as a separate indicator alongside an ordered formal-education band (none / primary / secondary / post-secondary).
4. **Model & tuning.** Decision tree classifier tuned with 5-fold `GridSearchCV` (ROC-AUC scoring): `max_depth=5`, `min_samples_leaf=50`. Depth-sweep analysis documents the underfitting-to-overfitting transition (train/test divergence beyond depth 7).
5. **Business-aligned evaluation.** Because the deployment decision is "which 20% of households to contact," the primary metric is **precision at the top 20%** of model-ranked households, not accuracy at an arbitrary 0.5 threshold.

## Key Drivers of Willingness to Pay

| Feature | Importance |
|---|---|
| Offered price (randomized) | 0.67 |
| Household head's education | 0.17 |
| Formal bank account | 0.12 |
| Grid connection | 0.03 |

Price dominates — as the embedded experiment guarantees it should — with household head's education and formal financial inclusion adding real signal. Urban residence contributes no additional splitting power once these are accounted for.

## Repository Structure

```
notebooks/
  01_data_ingestion.ipynb     Download, extraction, and first-look validation
  02_target_and_first_tree.ipynb   Target definition, feature engineering, tuned tree, top-20% evaluation
raw_data/                     (local only — see ingestion notebook)
reports/                      Client-style targeting brief (forthcoming)
```

## How to Run

```bash
git clone https://github.com/ProfFausat/solar-targeting-Nigeria
cd solar-targeting-Nigeria
python -m venv solar
solar\Scripts\activate        # Windows
pip install -r requirements.txt
```

Then run the notebooks in order. Notebook 01 downloads and extracts the survey data; notebook 02 is fully reproducible top-to-bottom (Restart Kernel → Run All).

## Roadmap

- Ensemble comparison: random forest and gradient boosting vs. the tuned tree, judged by precision-at-top-20%
- Two-page client-style targeting brief
- Fairness diagnostics across gender and urban/rural segments

## Author

**Fausat M. Ibrahim** — Data Scientist
[LinkedIn](https://www.linkedin.com/in/ibrahimfm) | [GitHub](https://github.com/ProfFausat)
