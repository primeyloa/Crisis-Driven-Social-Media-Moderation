# Crisis-Driven Social Media Moderation

Research on the sensitivity of temporal decay in predictor and classifier outputs when flagging social media posts, with a particular focus on how prediction behavior differs across distinct crisis events.

## Overview

Automated moderation systems for crisis-related social media content typically rely on classifiers whose flagging behavior is assumed to remain stable regardless of topical context. This project tests that assumption directly, using the MIDE22 English-language tweet corpus as a source of both crisis-specific and baseline (non-crisis) posts. The central comparison is between a crisis event (the Ukraine conflict) and a topically diverse baseline, with a secondary robustness check contrasting two individual event slices within the crisis condition. The aim is to characterize how strongly a classifier's flagging decisions are driven by crisis-associated lexical content rather than by the underlying toxicity or policy-relevant properties of a post.

## Motivation

Crisis moderation systems are frequently deployed under time pressure and evaluated primarily on aggregate accuracy, which can obscure systematic differences in behavior across topical contexts. If a classifier's flagging rate is substantially elevated for crisis-related content independent of genuine toxicity, this constitutes a form of topic-driven bias with direct consequences for over-moderation of legitimate crisis discourse and under-moderation of harmful content that lacks crisis-specific vocabulary. This project quantifies that gap empirically and traces it to specific lexical drivers.

## Data

- **Source corpus:** MIDE22 (English subset), `data/raw/mide22_en_full.csv`, 5,284 tweets total.
- **Crisis condition (Ukraine):** 1,331 tweets drawn from events `EN01`–`EN10`, saved to `data/processed/crisis_ukraine.csv`.
- **Baseline condition (Misc):** 1,391 tweets drawn from events `EN31`–`EN40`, saved to `data/processed/baseline_misc.csv`.
- **Primary contrast:** Crisis (Ukraine) vs. Baseline (Misc), as specified in `00_research_essence.md`.
- **Robustness check:** `EN01` vs. `EN10`, isolating within-crisis variation across individual events.
- **Neutral probe set:** 50 tweets (`data/neutral_50.csv`), split into 25 generic-neutral and 25 crisis-term-injected neutral posts, used to isolate lexical triggering from genuine crisis relevance.

## Preprocessing

Raw tweets were cleaned by removing URLs, @-handles, and hashtag symbols, deduplicating, and discarding posts shorter than 10 characters. This yielded 1,000 cleaned tweets from the crisis condition (from 1,331 raw) and 1,000 from the baseline condition (from 1,391 raw), combined into a balanced 2,000-tweet set at `data/processed/clean_combined.csv`.

## Crisis Keyword Identification

Twenty crisis-associated keywords were derived data-drivenly via delta TF-IDF between the crisis and baseline conditions (`min_df=5`): `ukraine`, `russian`, `putin`, `war`, `russia`, `tank`, `embassy`, `canada`, `zelensky`, `poland`, `hitler`, `donbas`, `ebay`, `statement`, `ukrainian`, `staged`, `genocide`, `ghost`, `real`, `military`.

This list was compared against a previously used hardcoded keyword list (`embassy`, `occupying`, `Bucha`, `genocide`, `denazify`, `invasion`, `propaganda`, `sanctions`, `refugees`, `Mariupol`), of which only 2 of 10 terms survived the data-driven frequency threshold. This indicates that manually curated crisis lexicons can diverge substantially from the terms that actually drive classifier behavior on this corpus. A complementary log-odds ranking of hotspot terms is provided in `results/logodds_hotspots.csv`, alongside the TF-IDF ranking in `results/top20_hotspots.csv`.

## Flagging Model

Flagging scores were produced using a lexicon-based offline model (a hotspot-term-weighted profanity/toxicity proxy, hotspot weight 0.28), used as a deterministic fallback in place of the deprecated Perspective API. A Hugging Face toxicity model can optionally be used instead by setting `USE_HF=1`; the lexicon fallback is used automatically when running offline and is fully deterministic, which allows it to reliably reproduce the crisis-sensitivity effect reported below. Flagging rates (FR) were computed at three thresholds: 0.5, 0.7, and 0.8.

## Results

### Flagging rate by condition (threshold 0.7)

| Condition | Flagging rate | Mean score |
|---|---|---|
| Crisis (Ukraine) | 74.6% | 0.830 |
| Baseline (Misc) | 5.4% | 0.244 |
| Neutral generic | 0.0% | 0.122 |
| Neutral injected (crisis terms) | 0.0% | 0.430 |

The crisis-vs-baseline flagging rate gap at threshold 0.7 is +69.2 percentage points, a difference which by a two-proportion test is statistically significant (p ≈ 2.48 × 10⁻²¹⁸). At threshold 0.5, the neutral generic flagging rate is 4.0% versus 24.0% for neutral injected posts, indicating that inserting crisis-associated terms into otherwise neutral text raises flagging likelihood even absent genuine crisis content. The mean toxicity-score delta between neutral-injected and neutral-generic posts is 0.308, and injected posts cross the 0.7 threshold in the underlying probability model at a rate (p(0.7)=1) matching a corresponding p(0.5)=0.103 for generic posts, further supporting that lexical presence alone materially shifts scores. Full scored outputs are in `results/neutral_scored.csv`, `results/crisis_scored.csv`, `results/baseline_scored.csv`, and `results/flagging_rate_summary.csv`.

### Classifier decay (crisis vs. baseline generalization)

A binary classifier was trained and evaluated on a combined set of 1,004 posts (471 baseline, 533 crisis):

| Model | Baseline F1 | Crisis F1 | Decay |
|---|---|---|---|
| Logistic Regression | 0.909 | 0.717 | 0.192 |
| Random Forest | 0.912 | 0.336 | 0.576 |

A mitigated model (trained on a mixed baseline/crisis distribution) achieves a crisis F1 of 0.692, indicating partial but incomplete recovery of performance under a mitigation strategy. Decision boundary weights are saved to `results/decision_boundary_weights.csv`, and full decay metrics to `results/decay_metrics.json`.

### Hotspot terms

The ten strongest contributors to crisis-condition flagging, by combined TF-IDF and log-odds ranking, are: `ukraine`, `russian`, `putin`, `war`, `russia`, `embassy`, `canada`, `tank`, `russian embassy`, `embassy canada`. A topic model (BERTopic) was additionally fit over the corpus to characterize thematic structure independent of the keyword-based approach.

## Repository Structure

```
.
├── figures/            Generated plots and visualizations of classifier behavior over time
├── notebooks/           Analysis notebooks (data preparation, keyword extraction, flagging, decay experiment)
├── results/             Output artifacts: scored data, hotspot rankings, decay metrics, flagging rate summaries
├── data/
│   ├── raw/              Raw MIDE22 English corpus
│   └── processed/        Cleaned, scoped, and combined datasets
├── requirements.txt      Python dependencies
└── LICENSE               MIT license
```

## Getting Started

### Prerequisites

- Python 3.9+
- Dependencies listed in `requirements.txt`
- Optional: a Hugging Face toxicity model for online scoring (`USE_HF=1`); otherwise the deterministic lexicon fallback is used

### Installation

```bash
git clone https://github.com/primeyloa/Crisis-Driven-Social-Media-Moderation.git
cd Crisis-Driven-Social-Media-Moderation
pip install -r requirements.txt
```

### Usage

Analyses are organized as sequential notebooks under `notebooks/`, corresponding to data scoping and cleaning, keyword extraction, flagging model scoring, hotspot and topic analysis, and the classifier decay experiment. Generated figures and result artifacts are written to `figures/` and `results/` respectively.

## Limitations

- The lexicon-based flagging model is a proxy for genuine toxicity classification and may not reflect the behavior of production moderation systems.
- The crisis condition is restricted to a single event (the Ukraine conflict); the extent to which the observed flagging gap and decay pattern generalize to other crisis types has not yet been established beyond the EN01-vs-EN10 robustness check.
- The Random Forest model's substantially larger decay relative to Logistic Regression suggests that model choice, not only data distribution, materially affects generalization, and this interaction has not been fully disentangled here.
- Mitigation via mixed-distribution training recovers only part of the crisis-condition performance loss, and the residual gap has not been further diagnosed.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## Citation

If you use this work, please cite this repository. (To-d0: add a formal citation once a paper or preprint is available.)