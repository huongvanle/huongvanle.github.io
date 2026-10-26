---
layout: post
title: What CaliciBoost taught me about molecular representations
date: 2026-10-26 14:00:00 +0900
description: Notes on our Journal of Cheminformatics paper comparing eight molecular representations for Caco-2 permeability prediction
tags: admet cheminformatics machine-learning publications
categories: research
---

Before a drug candidate can work as a pill, it has to cross the gut wall. One of the standard lab tests for this is the Caco-2 permeability assay, which measures how fast a compound passes through a layer of human intestinal cells. It is useful but slow and costly, so predicting it from structure alone is a common early filter in drug discovery.

In [CaliciBoost](https://doi.org/10.1186/s13321-025-01137-7) (_Journal of Cheminformatics_, 2025), we asked a simple question that is often skipped: before choosing a model, which way of describing a molecule to the model works best for this task?

## The setup

We compared eight molecular representations, from fingerprints to descriptor sets to a learned embedding:

- **Fingerprints:** Morgan, Avalon, ErG, MACCS
- **Descriptor sets:** RDKit descriptors, PaDEL, Mordred
- **Learned embedding:** CDDD

Each representation went through the same pipeline: AutoML training (AutoGluon), feature importance with permutation importance and SHAP, retraining on the top-ranked features, and Bayesian hyperparameter optimization. Keeping the pipeline fixed meant that differences in error came from the representation, not from tuning effort.

{% include figure.liquid loading="eager" path="assets/img/blog/caliciboost-workflow.jpg" class="img-fluid rounded z-depth-1" caption="The CaliciBoost workflow: one pipeline, applied to each of the eight representations." %}

We used two datasets of different scale. The TDC Caco-2 benchmark has 906 compounds, split by scaffold so that test molecules are structurally different from training ones. We also curated a larger set of 5,481 compounds from OCHEM, keeping only measurements taken under comparable conditions (apical-to-basolateral, pH 7.4, 37 °C).

## What we found

{% include figure.liquid loading="lazy" path="assets/img/blog/caliciboost-mae.png" class="img-fluid rounded z-depth-1" caption="Test error on the TDC Caco-2 benchmark for each representation, before and after feature selection and tuning." %}

Three things stood out to me.

**Classical descriptors won.** PaDEL, Mordred and RDKit descriptors gave the lowest errors. The best model, built on selected PaDEL descriptors, reached a mean absolute error of 0.256 on the TDC test set and placed first on the TDC Caco-2 leaderboard. The learned CDDD embedding came last. For a property driven by physicochemistry, such as size, polarity and hydrogen bonding, descriptors that encode those quantities directly still carry the most signal.

**Feature selection mattered most where there were the most features.** PaDEL and Mordred each provide well over a thousand descriptors. Trained on all of them, they were middling. Trained on a small, selected subset, they moved to the top. More features did not mean more information; past a point, they mostly added noise.

**The dataset changes the ranking.** On the larger and more varied OCHEM set, errors were higher across the board and the order shifted: RDKit descriptors with feature selection did best (MAE 0.353). A representation that wins on one benchmark is not guaranteed to win on another, which is a good reason to test on more than one dataset before drawing conclusions.

## What I take from it

The main lesson for me was about where to spend effort. It is tempting to start with the newest architecture. For a small, noisy ADMET endpoint, careful choice of representation and features gave a large share of the gain, with an off-the-shelf AutoML model doing the rest. This shapes how I approach other ADMET tasks now: compare representations under a fixed pipeline first, then decide whether a more complex model is worth it.

The paper is open access, and the code is on [GitHub](https://github.com/Calici/CaliciBoost).

**Citation:** Le HV, Ren W, Kim J, Yun Y, Park YB, Kim YJ, Han BK, Choi I, Park JI, Yun H-Y, Choi J-M. CaliciBoost: Performance-driven evaluation of molecular representations for Caco-2 permeability prediction. _Journal of Cheminformatics_ 17, 184 (2025). [doi:10.1186/s13321-025-01137-7](https://doi.org/10.1186/s13321-025-01137-7)
