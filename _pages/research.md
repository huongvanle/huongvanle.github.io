---
layout: page
permalink: /research/
title: research
description: Machine learning for molecular property prediction, structure-based modelling and biologics design — and when its predictions can be trusted.
nav: true
nav_order: 1
---

**In brief.** I lead the AI team at Calici Co., Ltd. in Daejeon, South Korea. I am first author of [CaliciBoost](https://doi.org/10.1186/s13321-025-01137-7) (_Journal of Cheminformatics_, 2025), which ranks **first on the Therapeutics Data Commons Caco-2 permeability benchmark**, a co-author on a 2026 [_PNAS_](https://doi.org/10.1073/pnas.2614150123) study of hepatitis E virus replication, and co-inventor on **three US patents** in GPU-based drug discovery. I came to this field from information science, with no formal training in biology or machine learning, and learned it by building systems that had to work. I am now looking for a PhD position.

Most of what I do comes back to one question: **when should you believe what a model says about a molecule?** The honest answer is often "less than the number suggests" — and that gap is what I would like to spend a doctorate on.

## Three results that shaped the question

**Representation mattered more than architecture.** In [CaliciBoost]({{ '/projects/2_admet_prediction/' | relative_url }}) I compared eight ways of describing a molecule for Caco-2 permeability through one fixed pipeline. On 906 compounds with a scaffold split, classical descriptors beat a learned embedding, and feature selection mattered more than model capacity. MAE 0.2560 ± 0.006 over five seeds, first on the leaderboard — with an unglamorous model.

**Binding is not function.** In the [hepatitis E study]({{ '/projects/1_structure_based_modelling/' | relative_url }}), four fatty acids docked into the same viral pocket within 0.44 kcal/mol of each other. In cells, one increased replication, one reduced it, and one did nothing. No scoring function could have separated them, because affinity and effect are different quantities.

**A generator cannot validate itself.** In [binder design]({{ '/projects/3_biologics_design/' | relative_url }}), candidates are optimised against a structure predictor's own confidence — so a high-scoring design has partly been selected for being the kind of thing that predictor likes. Its confidence is not independent evidence.

Together: a model's own score is rarely the evidence it appears to be, and the interesting work is building the checks that sit outside it.

## What I would like to pursue

Making machine learning dependable for molecular prediction when data is small, inconsistent, and cannot be pooled:

- **Evaluation that survives deployment.** Most reported gains come from one benchmark and one split. What transfers across datasets gathered under different assay conditions, and what reporting practice would make that visible?
- **Uncertainty that means something.** On endpoints with a few hundred noisy measurements, what is a calibrated confidence, and how should a chemist act on it?
- **Multitask and federated learning where bias differs by source.** Endpoints and institutions each carry their own systematic error. What transfers, what conflicts, and how should training be weighted when the data cannot be inspected centrally?
- **Physics as an independent check.** Simulation can corroborate a learned prediction, but only if the two are not already correlated.

## Why a doctorate

I learned this field one project at a time, under delivery pressure — reading PDB files column by column, learning proteins downwards from chains to residues to atoms, then for each new problem surveying prior work, choosing a baseline, testing it, extending the data and measuring again. That is a real method and I have become fast at it. It is also knowledge with gaps I cannot see from the inside.

I want to systematise it, go properly deep into the machine learning rather than the parts a project happened to need, work on questions that take longer than a delivery cycle, and put more of the result into the public record.

I am looking for a group working on machine learning for molecular property prediction, structure-based modelling or biologics design — particularly one where evaluation and reliability are research problems in their own right. If that sounds like your group, I would be glad to hear from you.
