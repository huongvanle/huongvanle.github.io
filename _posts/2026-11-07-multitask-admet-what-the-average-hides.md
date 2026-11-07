---
layout: post
title: Multitask ADMET models, and what the average improvement hides
date: 2026-11-07 18:00:00 +0900
description: Why training many ADMET endpoints in one model is not free, and why the mean across tasks is the wrong thing to report
tags: machine-learning admet multitask-learning evaluation
categories: research
---

Training one model to predict many ADMET endpoints is an appealing idea, and the argument for it is sound. Absorption, distribution, metabolism, excretion and toxicity all depend on overlapping structural features — size, lipophilicity, polarity, the presence of particular functional groups. A single encoder that learns those features once, feeding a separate head per endpoint, should let an endpoint with four hundred measurements borrow representation from one with forty thousand.

That is the theory. In practice the sharing is not free, and the price is paid unevenly across endpoints. Worse, the way these models are usually reported hides exactly who paid.

## The label matrix is block-sparse, not randomly sparse

The first thing that surprised me is structural rather than algorithmic.

Multitask molecular datasets are sparse — very few compounds carry labels for every endpoint. That much is expected, and masked losses handle it. But the sparsity is not random. Measurements come from assay campaigns, and a campaign measures one property across a chemical series chosen for that programme. The compounds with Caco-2 permeability data are largely not the compounds with hepatotoxicity data.

{% include figure.liquid loading="eager" path="assets/img/blog/mtl-conflict.png" class="img-fluid rounded z-depth-1" caption="Left: measured labels cluster by assay campaign rather than spreading across the matrix. Right: when two endpoints disagree about which direction the shared parameters should move, their sum serves neither." %}

This matters because co-labelled compounds are precisely where a shared encoder learns that two endpoints are related. A compound measured for both permeability and clearance constrains the representation to explain both at once. A compound measured for one of them constrains nothing about the relationship. When the overlap is concentrated in a few small blocks, most of what the model learns about each endpoint is learned in isolation, and the "shared" representation is shared in name more than in evidence.

So before reaching for a weighting scheme, it is worth measuring the overlap: for each pair of endpoints, how many compounds carry both labels. In my experience that number is often small enough to explain a disappointing result on its own.

## Gradients that pull in opposite directions

The algorithmic problem is better known. With shared parameters $$\theta$$ and per-task losses $$\mathcal{L}_i$$, the update follows the sum $$\sum_i w_i g_i$$ where $$g_i = \nabla_\theta \mathcal{L}_i$$. Two tasks conflict when their gradients point apart — the condition in [Gradient Surgery for Multi-Task Learning](https://arxiv.org/abs/2001.06782) is simply $$\cos\varphi_{ij} < 0$$.

That paper's useful contribution, to me, is not the method but the diagnosis. It argues that conflict alone is not what damages training; harm requires three things together, which they call the tragic triad: gradients that conflict, gradients of very different magnitude so one dominates the average, and high curvature along those directions, which makes the dominant task's gain look larger than it is and the other task's loss look smaller. All three conditions are easy to satisfy in an ADMET panel, where endpoints differ in data volume by two orders of magnitude.

Their remedy is to project away the conflicting component before summing: when $$g_i \cdot g_j < 0$$, replace $$g_i$$ with

$$g_i - \frac{g_i \cdot g_j}{\lVert g_j \rVert^2}\, g_j$$

and leave non-conflicting gradients alone. It is cheap, and it is a reasonable default. But it treats a symptom. If two endpoints genuinely demand different representations, removing the conflict from the update does not make them compatible — it just stops one from overwriting the other.

## Losses that are not on the same scale

A separate problem gets confused with the first. A squared-error loss on log _P_<sub>app</sub> and a cross-entropy loss on a binary toxicity label produce gradients of quite different magnitude, and the ratio has nothing to do with which endpoint matters. Hand-tuning per-task weights for dozens of endpoints is not a serious option.

The standard answer is to learn the weights as observation noise. [Kendall, Gal and Cipolla](https://arxiv.org/abs/1705.07115) treat each task's homoscedastic uncertainty $$\sigma_i$$ as a parameter, giving, for a regression and a classification head,

$$\mathcal{L} \approx \frac{1}{2\sigma_1^{2}}\mathcal{L}_1 + \frac{1}{\sigma_2^{2}}\mathcal{L}_2 + \log\sigma_1 + \log\sigma_2$$

A noisy endpoint learns a large $$\sigma$$ and is down-weighted; the $$\log\sigma$$ terms stop $$\sigma$$ from growing without bound, which would otherwise let the model ignore the data entirely. In practice you predict $$s = \log\sigma^2$$ for numerical stability.

I like this because the weighting is estimated rather than asserted, and because $$\sigma$$ is interpretable after training — an endpoint whose learned noise is high is telling you something about its data.

A third imbalance is easy to miss: sampling. If batches are drawn in proportion to dataset size, a forty-thousand-label endpoint dominates a four-hundred-label one regardless of loss weighting. Temperature sampling, $$p_i \propto n_i^{\alpha}$$ with $$\alpha < 1$$, borrowed from multilingual translation, flattens this.

## The measurement problem

Here is the part I think matters most, and it is not a modelling question.

**Negative transfer is defined relative to a single-task baseline.** An endpoint has suffered from joint training if the multitask model predicts it worse than a model trained on that endpoint alone. Which means: if you have not trained the single-task baselines, you cannot know whether it happened.

Most multitask results are reported as a mean across endpoints, or as a win against a previously published multitask model. Both can improve while a third of the individual endpoints regress. And the regressions are not randomly distributed — the endpoints most likely to be overwritten are the sparse ones with few co-labelled compounds, which are exactly the endpoints the multitask approach was supposed to rescue. An average driven by the data-rich endpoints is not just an incomplete summary; for this purpose it is close to the wrong statistic.

So the honest protocol costs more: train every endpoint alone, train them jointly, and report the per-endpoint difference. The mean can be a summary line, but the table of deltas is the result.

## Which endpoints should share a model at all

The obvious next question is whether to group endpoints rather than pool all of them, and the honest answer is that nobody can predict the groups well.

[Standley and colleagues](https://alphaxiv.org/abs/1905.07553) studied this directly and found something I keep returning to: the affinity between tasks measured by _transfer_ — how well a representation learned for one task serves another — showed no correlation with how well those tasks actually trained together, Pearson's $$r = -0.12$$, $$p = 0.74$$ in their main setting. Relatedness in the intuitive sense is not a reliable predictor of whether joint training helps.

That is an uncomfortable result, because it means the grouping has to be measured rather than reasoned about, and measuring pairwise interactions across dozens of endpoints is quadratic in the number of endpoints. For an ADMET panel that is expensive but not impossible, and it is probably a better use of compute than another architecture search.

## Where this leaves me

Multitask learning on ADMET panels is worth doing — I would not have built one otherwise. But I have stopped treating the average as the result. The questions I now ask of any multitask model, including my own:

- How many compounds carry labels for each _pair_ of endpoints?
- What is the per-endpoint change against a single-task baseline, not the mean?
- Which endpoints got worse, and do they share a property — low data volume, a distinct assay, little overlap with the rest?
- Are the learned $$\sigma$$ values telling me something about data quality I should have known beforehand?

None of this is exotic. It is mostly the discipline of comparing against the simpler thing, which is the same lesson I took from [comparing molecular representations]({{ '/blog/2026/caliciboost-molecular-representations/' | relative_url }}): the unglamorous baseline is the one that tells you whether the complicated idea earned its place.
