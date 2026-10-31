---
layout: page
title: ADMET prediction under limited and heterogeneous data
description: Predicting absorption, distribution, metabolism, excretion, and toxicity (ADMET) from structure when the data is small, noisy, and cannot be shared
img: assets/img/projects/admet-three-ways.png
importance: 2
category: research
related_publications: true
---

**2023 – present**

## The constraint that shapes everything

How much of a compound reaches the bloodstream, where it travels once it is there, how quickly the body breaks it down and clears it, and whether it does harm along the way — these are the properties that decide whether a molecule ever becomes a medicine, and in principle every one of them can be predicted from structure alone. The difficulty is the data.

For most ADMET endpoints the public record runs to a few hundred or a few thousand measurements, gathered by different laboratories under assay conditions that do not quite match. The measurements that would help most are held inside pharmaceutical companies and hospitals, where they cannot leave. So the useful question is rarely which architecture is most powerful. It is how to build something that stays reliable when the data is this thin, this noisy, and this scattered.

That constraint runs through all the work on this page, and the three strands below are three different answers to it.

{% include figure.liquid loading="eager" path="assets/img/projects/admet-three-ways.png" class="img-fluid rounded z-depth-1" caption="Three responses to the same constraint: improve the input, share structure across endpoints, or train across institutions without moving the data." %}

## One endpoint, a better input

The first strand asks what you can gain without more data at all, by changing how the molecule is described to the model.

CaliciBoost {% cite le2025caliciboost %} is a systematic comparison of eight molecular representations for Caco-2 permeability, run through one fixed pipeline so that the differences came from the representation rather than from tuning effort. On a benchmark of 906 compounds with a scaffold split, selected PaDEL descriptors with an XGBoost regressor reached an MAE of 0.2525, and under the official five-seed TDC protocol the model scored 0.2560 ± 0.006 and ranked first on the Caco-2 leaderboard.

The result I keep returning to is the negative one: with roughly a thousand compounds, classical descriptors beat a learned embedding, and careful feature selection mattered more than model capacity. On a dataset this size, reaching for a larger architecture is using a sledgehammer to crack a nut.

I led this work end to end — choosing the problem and the benchmark, directing the data curation and feature selection, sharing the model training with a colleague, writing the main sections, and handling submission and revision as first author. I have also written about [what it taught me about molecular representations]({{ '/blog/2026/caliciboost-molecular-representations/' | relative_url }}).

## Many endpoints, one shared model

The second strand uses the endpoints themselves as the extra signal. Absorption, distribution, metabolism, excretion and toxicity are not independent of one another: they all turn on overlapping structural features, such as size, lipophilicity and polarity. A model that learns them together can therefore let a sparse endpoint borrow representation from a well-populated one.

I built a multitask model on a multi-granularity graph attention (MGA) architecture. The network reads a molecule as a graph and attends to it at more than one level at once — individual atoms and the larger substructures they form — so that the learned representation carries both local chemistry and whole-molecule context. One shared encoder then feeds a separate prediction head for each endpoint, 88 of them in all, spanning physicochemical properties, medicinal-chemistry rules, absorption, distribution, metabolism and toxicity. It now runs as the ADME-Tox module of Calici's Pharmaco-Net platform. My part ran from selecting and testing the baseline architecture through curating the training data from public collections and fine-tuning, to preparing the trained model and the code our engineering team needed to integrate it.

Training many endpoints together is not simply additive, and the literature on multitask learning names the reasons. Tasks compete for a shared representation, so gradients can conflict and joint training can leave an endpoint worse off than a dedicated model would — negative transfer. Datasets differ by orders of magnitude in size, so without reweighting the largest endpoints dominate the encoder. Labels are sparse and unevenly distributed, since few compounds are measured for every property. And because each endpoint comes from its own assay, each carries its own systematic bias, which the shared representation has to tolerate rather than absorb. Recent work on multitask ADMET models frames this as [noise interference and task conflict](https://mlanthology.org/ijcai/2025/li2025ijcai-mtgib), and addresses it with uncertainty-based task weighting and task-specific feature selection. Getting the balance right across 88 endpoints, rather than maximising an average, is the real engineering problem.

## Many institutions, no shared data

The third strand tackles the constraint directly: the most informative toxicity data exists, but it is distributed across hospitals, research institutes and pharmaceutical companies that cannot pool it.

Federated learning is the standard answer — train locally, share only model updates — and I work on this as part of a five-year Korean national consortium on AI for drug development, now in its second year, building a privacy-preserving system for drug-induced liver injury prediction across clinical and non-clinical partners.

Because I cannot see partner data, the first year's work was about being ready for it. I curated a multi-source corpus of public DILI data and harmonised it: SMILES standardisation, conflict resolution favouring human clinical labels over animal and in vitro ones, and scaffold-level splitting so that structurally similar compounds cannot leak between training and test.

The feature design then has to cope with partners who hold very different things. I set it up in tiers. A universal tier, derived from molecular structure and basic pharmacokinetic parameters, can be produced by every participating organisation, so no site is excluded for being data-poor. On top of that sits an optional tier of richer measurements — serial liver enzyme readings, pharmacogenomic variants, proteomic and metabolomic markers — which only some sites can supply, handled by a masked loss so that a missing field costs nothing where it is absent. The point of the tiers is to raise the floor without capping the ceiling: the structural features guarantee that everyone can train, while the sites holding deeper clinical data still contribute all of it. I then ran a single-task federated simulation with NVIDIA FLARE, comparing cyclic and bagging tree-based federated boosting across simulated client nodes with deliberately uneven, non-IID data — the distribution we expect in the real system. The choice of gradient boosting rather than a deep model came straight out of the CaliciBoost finding: for small, noisy tabular ADMET data, it is the right size of tool.

The other half of the work is not modelling at all. It is sitting with the clinical and non-clinical data working groups to agree what each organisation can actually supply, in what format, and under which rules, and then designing endpoint definitions and a tiered feature structure that degrades gracefully when a site cannot provide the richer fields. A federated model is only as good as the agreement about what the data means.

A second national programme, now in its first year, extends this towards a closed loop: predict, test the prediction experimentally, and feed the measurements back to improve the model. We are in the research and data-collection phase, and the design question is how to make that loop automatic rather than a sequence of handovers.

## Where it is going

The three strands are converging. The representation findings from CaliciBoost inform the model choices in the federated work; the multitask architecture is where the federated DILI endpoints are heading next; and the closed-loop programme should eventually supply the one thing public ADMET data most lacks, which is new measurements generated to answer the model's own uncertainty.
