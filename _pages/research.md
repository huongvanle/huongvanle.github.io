---
layout: page
permalink: /research/
title: research
description: What I work on, and the question I would like to spend a doctorate on.
nav: true
nav_order: 1
---

Most of what I do comes back to one question: **when should you believe what a model says about a molecule?**

It is an awkward question to work on, because the honest answer is often "less than the number suggests". But it is the question that decides whether computational work is useful to the people doing the experiments, and it is the thread running through everything below.

## How I came to it

I was not trained in biology or in machine learning. My master's was in science and technology information science, and I came to drug discovery in 2021 by joining a company that did it.

So I learned the field by working in it. Learning what a PDB file actually contains, column by column, and why the same compound in a 2D and a 3D SDF answers different questions. Learning proteins downwards — chains, residues, the twenty amino acids, atoms — until the structures stopped being opaque. Then, for each new problem, reading what had already been tried: what people use for protein–ligand interaction, for protein–protein interaction, for predicting the ADMET profile of a compound. Choosing a baseline from that, testing it honestly, finding more data, retraining, measuring, tuning, and measuring again.

That is a real methodology and I have got reasonably fast at it. It is also knowledge assembled one project at a time, under delivery pressure, which is a particular way of knowing a field: strong where I have been, with gaps I cannot see from the inside.

## What it taught me

Three results shaped the question I now care about, and they are the same lesson wearing different clothes.

**Representation mattered more than architecture.** In [CaliciBoost]({{ '/projects/2_admet_prediction/' | relative_url }}) I compared eight ways of describing a molecule for Caco-2 permeability under one fixed pipeline. On a benchmark of 906 compounds, classical descriptors beat a learned embedding, and selecting features mattered more than model capacity. The model that topped the leaderboard was an unglamorous one.

**Binding is not function.** In a [study of hepatitis E virus replication]({{ '/projects/1_structure_based_modelling/' | relative_url }}), four fatty acids docked into the same viral pocket within 0.44 kcal/mol of each other. In cells, one increased replication, one reduced it, and one did nothing. No scoring function could have separated them, because affinity and effect are different quantities.

**A generator cannot validate itself.** In [binder design]({{ '/projects/3_biologics_design/' | relative_url }}), candidates are optimised against a structure predictor's confidence — so a high-scoring design has partly been selected for being the kind of thing that predictor likes. Its confidence is not independent evidence that the molecule works.

Put together: a model's own score is rarely the evidence it appears to be, and the interesting work is building the checks that sit outside it.

## What I would like to pursue

Making machine learning dependable for molecular prediction when the data is small, inconsistent, and cannot be pooled. Concretely, the directions I find myself drawn to are:

- **Evaluation that survives contact with deployment.** Most reported gains come from one benchmark with one split. How much transfers across datasets gathered under different assay conditions, and what splitting and reporting practice would make that visible rather than hidden?
- **Uncertainty that means something.** On endpoints with a few hundred noisy measurements, what is a calibrated and honestly communicated confidence, and how should a chemist act on it?
- **Multitask and federated learning where the bias differs per source.** Endpoints and institutions each carry their own systematic error. What genuinely transfers between them, what conflicts, and how should training be weighted when the data cannot be inspected centrally?
- **Physics as an independent check.** Simulation can corroborate a learned prediction — but only if the two are not already correlated. When does it add evidence, and when does it merely agree?

## Why a doctorate, and why now

Because the way I learned this field has limits I would like to fix deliberately. I want to systematise what I picked up by practice, go properly deep into the machine learning rather than the parts a project happened to need, and work on a question for longer than a delivery cycle allows. I would also like more of what I do to end up in the public record, where it can be checked, rather than only in internal tools.

I am looking for a group working on machine learning for molecular property prediction, structure-based modelling, or the design of biologics — and particularly one where evaluation and reliability are treated as research problems in their own right, not as the last section of a paper.

If that sounds like your group, I would be glad to hear from you.
