# Capstone Report — Structured Content Archetype Clustering

- **Author:** Khushnor Rahman
- **Lane:** Structured Content Archetype Clustering
- **Repo:** FlyRank-ML-
- **Date:** 2026-09-03

## 0. Abstract
Five sentences summarizing question, data, method, headline result, and purpose.

## 1. Problem framing
Decision support for content triage using unsupervised clustering.

## 2. Data safety
Used FlyRank ML dataset; strictly excluded client domain names and future metrics.

## 3. Baseline
Evaluated standard 4-cluster K-Means baseline.

## 4. Model / analysis
StandardScaler and K-Means clustering (K=4) on historical features.

## 5. Evaluation
Compared naive random row split against honest client-grouped holdout split.

## 6. Interpretation
Identified striking distance, top performers, decay, and long-tail clusters.

## 7. Recommendation
Prioritized editorial action queue based on CTR recovery potential.

## 8. Reproducibility
Random seed 42 set across splitting and model initialization.

## 9. Acknowledgments & data credit
Built on the [FlyRank ML Internship dataset](https://flyrank.ai).
