---
layout: post
title: Coverage is not a property of a dataset
date: 2026-11-08 14:00:00 +0900
description: On what it means for a chemical dataset to have a hole in it, and why the answer depends on who is asking
tags: cheminformatics admet machine-learning evaluation
categories: research
---

Most of the time, a model's relationship to its training data is treated as a filter. A prediction comes out, you ask whether the query compound falls inside the applicability domain, and if it does not you discount the number. That is a defensive question, and a useful one.

Lately I have been thinking about the same relationship from the other side. Suppose you are not only consuming a dataset but able to add to it — you can ask a laboratory to measure a handful of new compounds, and you have to say which ones. The defensive question becomes a design question: not _is this prediction trustworthy_, but _what is this dataset missing, and what should we measure next_.

It sounds like a straightforward thing to work out. Project the dataset into some space, look for the empty regions, go and fill them. The more I look at it, the less straightforward it gets, and the reason is that the word "missing" does not survive contact with a definition.

## Datasets have the shape of the projects that produced them

Public ADMET data is not a sample of chemical space. It is an accumulation of whatever a few hundred medicinal chemistry programmes happened to make, measured by whichever laboratories happened to publish. Compounds arrive in series: a scaffold, then thirty variations on it, because that is how optimisation campaigns work. Whole regions are dense because a target was fashionable in 2009, and whole regions are empty because nobody had a reason to go there.

A model trained on that inherits the shape. It will be confident in the neighbourhoods where the data piled up and uninformative everywhere else, and the boundary between the two has nothing to do with where chemistry is interesting. So the instinct to go and fill the empty parts is sound. The difficulty is saying which parts those are.

## Empty relative to what?

The first problem is that a hole needs a reference. A dataset is not empty in the abstract; it is empty relative to some universe you think it ought to cover.

Pick ChEMBL as the universe and your gaps are wherever bioactive chemistry has been explored but your assay has not followed. Pick a purchasable catalogue and the gaps move to wherever compounds are cheaply available and untested — a much more actionable set, and a much more commercially biased one, because catalogues reflect what is easy to make. Pick the chemotypes around one target family and you get a third answer again, narrower and more useful for one programme and useless for the next.

These are not refinements of a single question. They are three different questions that produce three different shopping lists, and the choice between them is made before any analysis happens. I have read a reasonable amount of work in this area and the reference set is very often left implicit.

## Empty in which space?

The second problem is sharper, and it is the one I keep coming back to, because I ran into the evidence for it in my own published work.

Emptiness is a statement about density, and density requires a metric. Write $\phi$ for whatever featurisation turns a molecule into a vector. It induces a distance

$$d_\phi(x, y) = \lVert \phi(x) - \phi(y) \rVert$$

and a hole is a region where the density of the dataset under $d_\phi$ is low while its surroundings are not. Nothing in that definition is a property of the molecules alone. Change $\phi$ and you change which compounds count as neighbours, which changes where the density is low, which changes what is missing.

That is not a theoretical worry. In [CaliciBoost]({{ '/blog/2026/caliciboost-molecular-representations/' | relative_url }}) we compared eight molecular representations on one permeability task through a single fixed pipeline, and they did not agree — not on performance, and by implication not on which compounds resemble which. If eight featurisations disagree about similarity, they disagree about holes.

Here is the effect on a library I enumerated to look at it directly. Same molecules, two standard featurisations, each reduced to two dimensions and binned on the same grid; a hatched cell is an empty region fully enclosed by occupied ones. Change either space, or the grid resolution, and watch what counts as missing.

<iframe
  id="chemspace-frame"
  src="{{ '/assets/html/chemical-space-coverage.html' | relative_url }}"
  title="Where a chemical library looks empty — interactive"
  loading="lazy"
  style="width: 100%; height: 1180px; border: 1px solid var(--global-divider-color); border-radius: 6px; display: block"
></iframe>

<script>
  (function () {
    var f = document.getElementById("chemspace-frame");
    if (!f) return;
    function sendTheme() {
      var t = document.documentElement.getAttribute("data-theme");
      try {
        f.contentWindow.postMessage({ theme: t === "dark" ? "dark" : "light" }, "*");
      } catch (e) {}
    }
    f.addEventListener("load", sendTheme);
    new MutationObserver(sendTheme).observe(document.documentElement, { attributes: true, attributeFilter: ["data-theme"] });
    addEventListener("message", function (e) {
      if (e.source === f.contentWindow && e.data && e.data.chemspaceHeight) {
        var h = e.data.chemspaceHeight;
        if (Math.abs(h - parseFloat(f.style.height || 0)) > 2) f.style.height = h + "px";
      }
    });
  })();
</script>

The two panels are the same compounds. One space says this library is full of holes; the other says it is essentially solid. And the compounds that sit on the rim of a gap in the first space — the ones you would reach for if you were trying to fill it — are scattered across the second, bordering nothing.

Running the same analysis through five standard featurisations makes the spread clearer:

| Featurisation               | Holes | Molecules on a hole rim |
| --------------------------- | ----: | ----------------------: |
| Morgan fingerprints (r=2)   |    20 |             446 (26.0%) |
| Morgan fingerprints (r=3)   |    15 |             767 (44.6%) |
| MACCS keys                  |     6 |               90 (5.2%) |
| Atom-pair fingerprints      |     2 |              136 (7.9%) |
| Physicochemical descriptors |     1 |                9 (0.5%) |

The rim is the shortlist: if you were using this analysis to choose compounds for assay, those are the ones you would send. Measured as a Jaccard overlap, the five shortlists agree with each other by **0.38 at best and 0.00 at worst** — and 0.38 is between Morgan at radius 2 and Morgan at radius 3, two settings of the same fingerprint. Across families the overlap is near zero; Morgan r=2 and physicochemical descriptors share 0.4% of theirs.

The code and the full overlap matrix are in [a small repository](https://github.com/huongvanle/chemical-space-coverage), if you want to change the grid resolution and watch the numbers move.

I want to be careful not to overstate what this shows. It is an enumerated library, not a real assay set, and two dimensions of PCA is a crude view of either space. But the direction of the effect is not an artefact of those choices, and it is enough to make the point that interests me: **a gap is a property of a dataset and a featurisation together, and the featurisation is a modelling decision somebody made.** Two groups can analyse the same dataset, both honestly conclude that it has holes, and commission entirely different experiments.

## You cannot order a point in chemical space

There is a third difficulty, and it is the one I suspect is least appreciated in the machine learning literature, because it does not arise in the settings where that literature was developed.

Active learning assumes a pool. You identify the most informative point and you query its label. In a laboratory you cannot do that. What you identify is a _location_ — a position $z$ in feature space where data would help — and what you can actually buy or make is whatever the catalogue happens to contain:

$$x^\star = \operatorname*{arg\,min}_{x \,\in\, \mathcal{C}} \; d_\phi\big(\phi(x),\, z\big)$$

That residual distance is the gap between what you wanted and what you can get, and it is not small. It is also, inevitably, measured in the same $\phi$ whose arbitrariness is the problem in the first place. Add that synthesis cost is wildly uneven across regions — the empty parts of chemical space are sometimes empty precisely because they are hard to make — and the clean picture of filling a void starts to look like a constrained optimisation with an objective nobody has written down carefully.

## And then, how would you know it worked

Suppose you do it anyway: define a space, define a universe, pick compounds, run the assays, retrain. How do you tell whether it helped?

The tempting answer is to report the change in global test error, and I think that is the wrong statistic — for the same reason I [argued about multitask models]({{ '/blog/2026/multitask-admet-what-the-average-hides/' | relative_url }}) recently. If you added fifty compounds in one sparse region of a thousand-compound set, the global mean is mostly noise from everywhere else. What you want to know is whether the model improved _in the region you filled_, and whether it did so more than the same budget spent at random or on the points the model was least certain about. Those baselines are not optional. Coverage-driven selection is one of several established strategies, and the literature on uncertainty sampling, query-by-committee and core-set selection is old enough that a new method has to beat them rather than cite them.

## Where this leaves me

I am working on this, and I have nothing to report yet — no method I am ready to defend and no numbers. What I have is a reframing that I think is correct and that I have not seen stated plainly:

> Coverage is not a property of a dataset. It is a property of a dataset, a featurisation, and a reference universe — and most of the time only the first is reported.

The selection half of that is now measured: five reasonable featurisations nominate almost disjoint sets of compounds. What is not measured is whether the choice changes the _outcome_ — whether a model trained on one shortlist ends up better than a model trained on another. That needs acquisition simulation, retraining, and evaluation inside the filled region against proper baselines, and it is the part I am working on.

If that is right, then "we identified gaps in chemical space and designed experiments to fill them" is an incomplete sentence, and two papers making that claim may not be comparable at all. Whether the choice of featurisation changes the _outcome_ of a gap-filling campaign, and not merely its shopping list, is an empirical question. It is the one I would most like to answer, and it is answerable retrospectively, before anyone spends money at the bench.

That seems like the right order to do it in.
