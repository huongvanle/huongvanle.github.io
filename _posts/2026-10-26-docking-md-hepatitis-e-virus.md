---
layout: post
title: Docking and MD for a viral protein pocket, and what they could not settle
date: 2026-10-26 15:00:00 +0900
description: Notes on the in silico part of our PNAS study on fatty acids and hepatitis E virus replication
tags: molecular-dynamics docking virology publications
categories: research
---

Hepatitis E virus (HEV) is a leading cause of acute viral hepatitis worldwide, and it hits pregnant women hardest, especially in the third trimester. Why pregnancy makes the infection so much worse has never been fully explained.

Our [PNAS study](https://doi.org/10.1073/pnas.2614150123) (2026), led by the Meng laboratory at Virginia Tech and the Kenney laboratory at Ohio State, looks at this through host lipids. Pregnancy raises circulating free fatty acids, and the paper traces a hormone–lipid–HEV axis: placental lactogen pushes up fatty acid levels in liver and placental cells, oleic acid enhances HEV replication, and blocking phosphatidylethanolamine synthesis reduces it.

My part of the work, with colleagues at Calici, was the structural modelling: could a fatty acid actually bind the viral protein where the biology suggested it should?

## The setup

HEV ORF1 encodes a non-structural polyprotein. An earlier study had predicted a fatty acid–binding domain (FABD) inside its papain-like cysteine protease region, but no one had tested whether fatty acids bind there, or which ones.

We started from the published crystal structure of the HEV zinc-binding non-structural protein ([PDB 6NU9](https://www.rcsb.org/structure/6NU9)) and ran the modelling on Calici's Pharmaco-Net platform. The pocket was located with the PocketFinder module, then docked with AI-Dock using a 0.2 Å grid and 500 runs per ligand, with binding affinities estimated by the DeepCalici-Plus module.

The ligand set was chosen by the biology, not by convenience. Oleic, linoleic, palmitic and heptadecanoic acids were all raised during placental lactogen treatment or HEV infection in the lipidomics. Mycolipenic acid and a lactone that went _down_ under the same conditions went in as comparisons.

The docking poses then went into molecular dynamics: Amber FF99SB for the protein and GAFF for the ligands, solvated in TIP3P water with an 8 Å buffer and neutralising ions, run in GROMACS 2021.4 through heating, NPT equilibration and a 10 ns production run. Binding free energies came from MM/PBSA on those trajectories.

{% include figure.liquid loading="eager" path="assets/img/blog/hev-fabd-docking.png" class="img-fluid rounded z-depth-1" caption="The FABD in HEV ORF1 and the predicted binding poses and contacts for oleic, linoleic and palmitic acid. Figure from Yadav et al., <em>PNAS</em> 123(27):e2614150123 (2026), SI Appendix Fig. S1, <a href='https://creativecommons.org/licenses/by-nc-nd/4.0/'>CC BY-NC-ND 4.0</a>." %}

## What we found

{% include figure.liquid loading="lazy" path="assets/img/blog/hev-binding.png" class="img-fluid rounded z-depth-1" caption="Predicted binding energies for each ligand in the FABD pocket. Values from SI Appendix, Table S1 of the paper." %}

**The fatty acids fit the pocket; the comparison compounds did not.** Oleic, linoleic, palmitic and heptadecanoic acid all docked favourably, between −5.38 and −5.89 kcal/mol. Mycolipenic acid, a branched fatty acid that was downregulated in the lipidomics, gave a positive docking energy, as did the lactone. That the ligand set separated along the same line as the lipidomics was the first sign the pocket was worth taking seriously.

**Each fatty acid sat differently in the pocket.** Oleic acid was predicted to hydrogen bond with Ser168 and Glu158 (S683 and E673 in full-length ORF1 numbering), while linoleic and palmitic acid contacted Gly159 and Leu161 instead. Those two oleic acid residues turned out to be conserved across the four human-infecting genotypes, whose FABD regions share 50.6% identity, but absent in the more distant _Avihepevirus_ and _Rocahepevirus_ species.

**The scores did not predict the biology.** This is the part I find most useful to remember. All four fatty acids scored within half a kcal/mol of each other, and MM/PBSA did not re-order them in a way that matched experiment either. But when the wet-lab team supplemented infected cells, the three behaved completely differently: oleic acid increased HEV replication, palmitic acid reduced it, and linoleic acid did nothing on its own. A binding score says a ligand can occupy a pocket. It says nothing about what occupying it does to the protein.

So the modelling answered a narrower question than it might appear to. It showed that the predicted FABD is a plausible fatty acid site, that the residues involved are conserved where the biology says they should be, and that compounds moving the wrong way in the lipidomics do not fit. The paper is explicit that proving the domain matters needs mutagenesis of those lipid-interacting residues, which is work still to come.

## What I take from it

Two things stayed with me.

First, the value of the computational part came from how tightly it was tied to the experiments. The ligands were chosen from the lipidomics, and the residues it flagged were then checked against sequence conservation across genotypes. On its own, a docking score for one fatty acid in one pocket would have meant very little.

Second, it is worth being precise in print about what a model does and does not establish. The gap between near-identical binding energies and three opposite biological outcomes is the honest result here, and saying so is more useful to the next group than a tidier story would be.

**Citation:** Yadav KK, Bhandari M, Mahsoub HM, Park YB, Le HV, Choi JM, Brown HM, Rajashekara G, Kenney SP, Meng X-J. Fatty acid regulation and phosphatidylethanolamine biosynthesis are important for hepatitis E virus replication. _PNAS_ 123(27), e2614150123 (2026). [doi:10.1073/pnas.2614150123](https://doi.org/10.1073/pnas.2614150123)
