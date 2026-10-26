---
layout: post
title: What CaliciBoost taught me about molecular representations
date: 2026-10-26 14:00:00 +0900
description: Notes on our Journal of Cheminformatics paper comparing eight molecular representations for Caco-2 permeability prediction
tags: admet cheminformatics machine-learning publications
categories: research
---

An oral drug has to be absorbed through the intestine before it can work. Caco-2 cell permeability is a widely used in vitro proxy for this intestinal absorption, and predicting it well from structure helps prioritize compounds and saves both time and cost in ADMET screening.

Many models have been built for this task, but most studies focus on a limited set of features or a single modeling approach. In [CaliciBoost](https://doi.org/10.1186/s13321-025-01137-7) (_Journal of Cheminformatics_, 2025), we instead compared a wide range of molecular representations systematically, to see which way of describing a molecule works best for Caco-2 permeability prediction.

## The setup

We compared eight molecular representations, from fingerprints to descriptor sets to a learned embedding:

- **Fingerprints:** Morgan, Avalon, ErG, MACCS
- **Descriptor sets:** RDKit descriptors, PaDEL, Mordred
- **Learned embedding:** CDDD

Each representation went through the same pipeline: AutoML training (AutoGluon), feature importance with permutation importance and SHAP, retraining on the top-ranked features, and Bayesian hyperparameter optimization. Using one pipeline for every representation kept the comparison fair.

{% include figure.liquid loading="eager" path="assets/img/blog/caliciboost-workflow.jpg" class="img-fluid rounded z-depth-1" caption="The CaliciBoost workflow: one pipeline, applied to each of the eight representations. Figure from Le et al., <em>J Cheminform</em> 17:184 (2025), <a href='https://creativecommons.org/licenses/by-nc-nd/4.0/'>CC BY-NC-ND 4.0</a>." %}

We used two datasets that differ in scale and chemical diversity. The TDC Caco-2 benchmark has 906 compounds, with the scaffold-based split provided by TDC, so that test molecules have scaffolds not seen in training. We also curated a set of 5,481 compounds from OCHEM, about five times larger. To reduce experimental variability, we excluded measurements labeled basolateral-to-apical, measured at a pH other than 7.4, or measured at a temperature other than 37 °C, and removed duplicates.

## What we found

{% include figure.liquid loading="lazy" path="assets/img/blog/caliciboost-mae.png" class="img-fluid rounded z-depth-1" caption="Test MAE of the top-feature model for each representation on the two datasets. Values from Figs. 6 and 7 of the paper." %}

Three things stood out to me.

**Descriptor sets did best on the benchmark.** On TDC, PaDEL, Mordred and RDKit descriptors gave the lowest errors. The best model, an XGBoost regressor trained on selected PaDEL descriptors, reached a mean absolute error (MAE) of 0.2525 on the test set. Evaluated under the official TDC protocol over five seeds, it scored 0.2560 ± 0.006 and ranked first on the TDC Caco-2 leaderboard. The learned CDDD embedding had the highest error on TDC. Among the most influential PaDEL features were physicochemical and autocorrelation descriptors, such as the number of acidic groups (nAcid) and charge- and mass-weighted autocorrelations.

**Feature selection mattered most for the largest descriptor sets.** PaDEL provides 1,875 descriptors and Mordred more than 1,800. Trained on all of them, PaDEL (MAE 0.3058) and Mordred (0.3033) trailed RDKit descriptors, Morgan and Avalon. After feature selection and tuning, they moved to the top: the final PaDEL model used its 73 most important features. A smaller, well-chosen subset kept or improved accuracy while making the model easier to interpret.

**The dataset changes the ranking.** On the larger and more diverse OCHEM set, errors were higher for every representation, and the order shifted: RDKit descriptors did best (MAE 0.3529), while PaDEL fell to fifth. A representation that wins on one benchmark is not guaranteed to win on another, which is a good reason to test on more than one dataset before drawing conclusions.

## What I take from it

For me, the main lesson was about where to spend effort. It is tempting to start with the newest architecture, but on a data-limited ADMET task, a careful choice of representation and features, combined with an AutoML model, was enough to reach the top of the benchmark. This is how I now approach other ADMET tasks: compare representations under a fixed pipeline first, then decide whether a more complex model is worth it.

The paper is open access, and the code is on [GitHub](https://github.com/Calici/CaliciBoost).

**Citation:** Le HV, Ren W, Kim J, Yun Y, Park YB, Kim YJ, Han BK, Choi I, Park J-I, Yun H-y, Choi J-M. CaliciBoost: Performance-driven evaluation of molecular representations for Caco-2 permeability prediction. _Journal of Cheminformatics_ 17, 184 (2025). [doi:10.1186/s13321-025-01137-7](https://doi.org/10.1186/s13321-025-01137-7)
