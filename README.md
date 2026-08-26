# Crisis-Driven Social Media Moderation

Research on the sensitivity of temporal decay in predictor and classifier outputs when flagging social media posts, with a particular focus on how prediction behavior differs across distinct crisis events.

## Overview

Automated moderation systems for crisis-related social media content typically rely on classifiers trained on data drawn from specific events (e.g., natural disasters, public health emergencies, civil unrest). This project investigates the extent to which such classifiers' predictions decay in reliability over time following an event, and whether this temporal decay pattern generalizes across event types or is instead event-specific. The central research question is whether a single crisis-classification model can be trusted to perform consistently as an event unfolds and recedes from public attention, or whether event-specific recalibration is necessary.

## Motivation

Crisis moderation systems are frequently deployed under time pressure, often using models trained on prior events with the assumption that classification performance remains stable. This assumption is rarely tested explicitly. Divergence in temporal decay behavior across event types has direct implications for how moderation systems should be retrained, recalibrated, or ensembled during an active crisis.

## Repository Structure

```
.
├── figures/            Generated plots and visualizations of classifier behavior over time
├── notebooks/           Analysis notebooks covering data preparation, model training, and evaluation
├── results/             Output artifacts from experiments (metrics, predictions, summary tables)
├── requirements.txt      Python dependencies
└── LICENSE               MIT license
```

## Getting Started

### Prerequisites

- Python 3.9+
- Dependencies listed in `requirements.txt`

### Installation

```bash
git clone https://github.com/primeyloa/Crisis-Driven-Social-Media-Moderation.git
cd Crisis-Driven-Social-Media-Moderation
pip install -r requirements.txt
```

### Usage

Analyses are organized as notebooks under `notebooks/`. Each notebook corresponds to a stage of the pipeline (data preparation, model training, temporal decay analysis, and cross-event comparison). Run them in sequence to reproduce the reported results; generated figures and result artifacts are written to `figures/` and `results/` respectively.

## Methodology

At a high level, the approach involves:

1. Assembling labeled social media post data spanning multiple distinct crisis events.
2. Training or applying a predictor/classifier for crisis-post flagging on each event.
3. Measuring how classifier performance changes as a function of time elapsed since the triggering event.
4. Comparing the resulting decay curves across events to assess whether decay behavior is event-invariant or event-specific.

## Results

Summary tables and figures are available in `results/` and `figures/`. (Add a short narrative summary of key findings here once results are finalized.)

## Limitations

This is an active research repository, and the findings should be read with the following caveats in mind:

- Results depend on the specific events and datasets included in the analysis and may not generalize to crisis types outside that sample.
- Temporal decay estimates are sensitive to how "time since event" is defined and to the density of posts available at each time window.
- No claims are made regarding real-time deployment readiness; this work is exploratory and diagnostic in nature.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## Citation

If you use this work, please cite this repository. (Add a formal citation once a paper or preprint is available.)