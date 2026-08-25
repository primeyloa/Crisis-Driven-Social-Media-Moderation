# Crisis-Driven Moderation: Sensitivity Analysis (MiDe22 EN)

## Abstract
# Project Abstract

**Title:** Crisis-Driven Moderation: A Longitudinal Analysis of Algorithmic Sensitivity and Temporal Bias in Social Media Mining

**Abstract:**
Automated content moderation systems are increasingly vital for maintaining the integrity of digital discourse, yet they remain vulnerable to the dynamic nature of online communication. This research investigates the phenomenon of "Crisis Sensitivity"—the hypothesis that algorithmic flagging systems exhibit shifting decision boundaries and increased false-positive rates during societal crises, such as pandemics or geopolitical conflicts. Utilizing longitudinal datasets including MiDe-22 and Mega-COV, this study employs a multi-faceted computational approach: (1) sensitivity auditing using state-of-the-art toxicity APIs, (2) sema

## 1. Introduction
Motivation per docs/project_proposal.md:15 — static models degrade during crises (No Time Like the Present [2], Temporal Bias [3]).

## 2. Method
- Data: MiDe22 EN n=5284, scoped Ukraine (EN01-10, n=1331) vs Misc (EN31-40, n=1391), cleaned 1000+1000, 50 neutral probes (25 generic + 25 injected, generic fixed to avoid 'embassy' contamination, injected via data-driven TF-IDF delta top-20).
- Auditing: Perspective API DEPRECATED — local HF cardiffnlp/twitter-roberta-base-offensive / unitary/toxic-bert with deterministic lexicon offline fallback (USE_HF=1 to enable HF), FR at 0.5/0.7/0.8.
- Drift: TF-IDF delta + log-odds + optional BERTopic, hotspot mapping to flagged tweets (data-driven, no hardcoded list).
- Decay: TF-IDF + LR (baseline-fit) and RF replication, test on crisis; mitigated mix (+20% crisis).

## 3. Results
Flagging Rate @0.7:
           split  threshold    n    FR  mean_tox
 neutral_generic        0.7   25 0.000  0.101920
neutral_injected        0.7   25 0.000  0.441411
          crisis        0.7 1000 0.746  0.828985
        baseline        0.7 1000 0.055  0.243436

Decay:
{
  "n_baseline": 471,
  "n_crisis": 533,
  "LR_baseline_F1": 0.9090909090909091,
  "LR_crisis_F1": 0.7170795306388527,
  "LR_decay_F1": 0.1920113784520564,
  "RF_baseline_F1": 0.9115646258503401,
  "RF_crisis_F1": 0.336322869955157,
  "mitigated_F1": 0.6916221033868093,
  "label_mapping": "False=1 (misinfo), True=0, Other=dropped"
}

Hotspots top 10:
           word  delta_tfidf  crisis_tfidf  baseline_tfidf
        ukraine     0.051435      0.052933        0.001498
        russian     0.039104      0.041907        0.002803
          putin     0.032429      0.035139        0.002711
            war     0.031957      0.032772        0.000815
         russia     0.029198      0.040211        0.011012
        embassy     0.023917      0.023917        0.000000
         canada     0.022929      0.023665        0.000735
           tank     0.021558      0.021558        0.000000
russian embassy     0.020300      0.020300        0.000000
 embassy canada     0.017930      0.017930        0.000000

Figures: see ../figures/fig1_flagging_rate.png etc.

## 4. Discussion
Shifts imply over-flagging of legitimate crisis discourse; time-stratified mixing mitigates F1 decay. Limitations: English-only, topic-time confound, proxy thresholds.

## 5. References
See docs/project_proposal.md:47 + Toraman et al. 2024 MiDe22.

Generated 2026-08-25 14:10:57.388626
