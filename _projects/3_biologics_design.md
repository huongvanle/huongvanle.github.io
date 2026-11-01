---
layout: page
title: Computational design of peptide and antibody therapeutics
description: Generating binders against a target, and the harder problem of deciding which generated design is worth making
img: assets/img/projects/biologics-design.png
importance: 3
category: research
---

**2024 – present**

## Generating is the easy part

Generative models have changed what is possible in binder design. Given a target structure and a site on it, a model will produce plausible peptide or antibody candidates in hours. The difficulty has moved: with hundreds of designs in hand and the budget to synthesise a handful, the question is which ones to make.

That question is harder than it looks, because the obvious way to rank designs is circular. A generative pipeline optimises against a structure predictor's own confidence, so a design that scores well has, in part, been selected for being the kind of thing that predictor is confident about. Its confidence is not independent evidence that the molecule binds.

So the work splits in two. One half is generation. The other half, which takes most of the time, is assembling checks that the generator did not get to optimise against: physics-based simulation of the designed complex, a binding-affinity model trained separately on measured data, and property prediction to catch liabilities before anything is synthesised.

{% include figure.liquid loading="eager" path="assets/img/projects/biologics-design.png" class="img-fluid rounded z-depth-1" caption="Two generation routes feeding one verification chain. Designs that survive it are the ones worth making." %}

## Peptide binders

Peptides sit awkwardly between small molecules and proteins. They are flexible, so a single modelled conformation says less than it would for a rigid ligand, and they often have to compete at a protein–protein interface that is broad and shallow rather than a well-formed pocket.

The generative route starts from a random sequence folded against the target, and optimises that sequence by following the gradient of a loss built from the predictor's confidence in the complex together with packing and topology terms. The resulting backbone is then re-sequenced by an inverse-folding model, with the interface residues held fixed, and the redesigned sequences are re-predicted and relaxed. I selected and tested the baseline architecture, collected the training data, and fine-tuned it for the targets we work on.

Before any of that, though, I spend time on the biology: the pathway the target sits in, what the natural binding partner does, and what a useful peptide would actually have to achieve. A generator will hit a site you point it at, so pointing it at the right site is the part that decides whether the campaign is worth running.

Each design then goes through the verification chain — peptide docking and MD to rebuild and relax the complex, binding affinity from the DeepCalici Plus 4D-CNN model that my team built for [Pharmaco-Net](https://pharmaco-net.org/), and ADMET prediction — before a shortlist goes to the collaborator for synthesis and assay.

Two campaigns have run this way. One designed a peptide to mimic an endogenous secreted agonist, so that it would switch a signalling pathway on rather than block it — the less common problem, and the harder one, because occupying the site is not enough: the peptide has to reproduce what the natural partner does there. The other competed at a well-studied immune-checkpoint interface, where the contact surface is broad and flat and the peptide has to displace an existing protein–protein interaction rather than fill a pocket. The loop itself grew out of earlier work on the TEAD4 interface, described under [structure-based modelling]({{ '/projects/1_structure_based_modelling/' | relative_url }}).

## Antibodies

Antibody design adds a second objective, and it is the one that makes the problem interesting. It is not enough to bind the antigen well. The antibody must bind _this_ antigen and not the others it will meet, and a model rewarded only for affinity to the intended target has no reason to avoid anything else. Specificity has to be designed in, not hoped for.

Mechanically the route differs from peptides. The framework is held fixed and only the CDR loops are designed; hotspot residues define the epitope to aim at; a diffusion model conditioned on the antigen generates the antibody–antigen pose and the loop conformations together; and inverse folding assigns sequences to those loops. Each design is then re-folded by a structure predictor and kept only where the prediction agrees with the intended structure — which is the weakest link in the whole pipeline, since no filter has yet been established that reliably separates designs that will work from designs that merely look right.

We are building a specificity-aware design model in collaboration with an academic AI laboratory. On that strand I manage the programme, prepare the data, and verify the model and its test results. Alongside it I lead applied campaigns for collaborators, currently against a secreted extracellular target where the antibody has to engage the intended protein and leave its structural relatives alone. On those I work from target mechanism and pathway through planning, generation, and the docking and MD verification that follows, to the point where designs are handed over for experimental testing.

The same design question appears in a five-year national programme on antibody–drug conjugates, now in its first year. An ADC has more to design than the binder: the antibody has to reach the right cells, and the linker and payload decide what happens once it arrives. Predicted linker toxicity and pharmacokinetic behaviour are not downstream checks there but inputs to the design choice itself, which is why that work belongs here rather than with property prediction.

## Status

None of this is published yet. The specificity-aware model is still being built, the ADC programme is in its first of five years, and the applied campaigns are at the stage where designs have been delivered and the experimental verdict is what matters next. That verdict is the only thing that will tell us whether the verification chain is filtering the right way, which is the question I most want answered.
