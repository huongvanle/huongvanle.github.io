---
layout: page
title: Structure-based modelling of biomolecular complexes
description: Working out how two molecules interact in three dimensions — where they bind, through which contacts, and what to change to bind better
img: assets/img/projects/sbm-workflow.png
importance: 1
category: research
related_publications: true
---

**2021 – present**

## Why three dimensions

A two-dimensional view will tell you that two molecules are likely to interact. Feed a protein sequence and a ligand SMILES string to a model and it will give you a probability, and that is useful as a filter. What it will not tell you is _how_ they interact, or _where_ on the protein the interaction happens. Without that, there is no rational way to improve the ligand — only trial and error.

Working in three dimensions changes the question. Docking and molecular dynamics show the binding mode itself: which residues on the protein carry the interaction, which groups on the ligand are doing the work, and which contacts are stable once the complex is allowed to move. That is what turns a prediction into a design strategy. If the analysis shows a hydrogen bond to a particular residue holding the complex together, the chemist knows what to keep; if a region of the ligand sits in solvent contributing nothing, they know what to change.

## How the work runs

{% include figure.liquid loading="eager" path="assets/img/projects/sbm-workflow.png" class="img-fluid rounded z-depth-1" caption="From sourcing structures, through the computational cycle, to experimental validation and what follows it." %}

Every project starts away from the computer, with the biology: what the target does, what is known about its mechanism, and what interaction we are actually trying to explain or create.

Then the structures have to exist. For the target protein I look for an experimental structure in the [PDB](https://www.rcsb.org/) first and build an AlphaFold model only when there is none. Ligands come from PubChem where they are available, and are otherwise built and energy-minimised with RDKit. Peptides are built the same way, and complexes that need it are refined by MD before anything else happens.

With structures in hand the pipeline runs: dock, take the lowest-energy conformers forward, run MD with MM/PBSA to check that the complex is stable and to get a binding free energy, predict binding affinity with an AI model, and select the hits. Those hits are profiled for ADMET, and then optimised — either to bind more tightly or to fix an ADMET liability.

Optimisation is not the end of it. Each modified compound goes back through the same cycle, so that a change made to improve binding is checked for what it costs elsewhere, and the loop runs until the predictions settle. The interaction analysis is what makes each round rational rather than a guess: it says which contacts to preserve and which part of the molecule has room to change. What comes out is a set of candidate leads.

Those predictions then have to survive contact with an experiment. Our collaborators test the candidates at the bench, and that is the only step that settles whether the modelling was right — including about the compounds it ranked highly that turn out to do nothing, which are as informative as the ones that work. When the leads are confirmed, the work goes forward: a paper, a patent filing, and the deeper studies that follow. That last stage is also what keeps the method honest, because every project ends with a measurement of how good the predictions actually were.

## Systems and methods

The same approach covers a range of complexes, which is most of the reason the work stays interesting:

- **Molecular dynamics** with GROMACS and Amber, across protein–ligand, protein–ligand–ion, protein–protein, protein–peptide, protein–RNA and antibody–antigen systems
- **Binding free energy** by MM/PBSA
- **Docking** — AutoDock and AutoDock Vina for small molecules, Rosetta for peptides, rDock and RLDock for RNA
- **Interaction analysis** in 2D and 3D with LigPlot, ProLIF and PyMOL
- **Binding energy and affinity** — docking scores alongside affinity predicted by a 4D-CNN model
- **Structure work** — 3D structure prediction for proteins, ligands, antibodies and RNA, active site prediction, and complex refinement by MD
- **Pose prediction that accounts for water** in the binding site

Protein–ligand work runs as a complete pipeline on Calici's [Pharmaco-Net](https://pharmaco-net.org/) platform: AI-Dock for docking, DeepCalici Plus for binding affinity, InteractionViewer for contact analysis, ADME-Tox for property prediction, and ChemEdit for optimisation. The platform is built around protein–ligand systems, so peptide, antibody–antigen and RNA work runs on internal servers instead, with Python, RDKit, AutoDock Tools, AmberTools, GROMACS and PyTorch.

## Published work

**Fatty acid binding in the hepatitis E virus ORF1 polyprotein.** A collaboration with Professor Scott Kenney's group at The Ohio State University, together with the Meng laboratory at Virginia Tech {% cite yadav2026hev %}. A fatty acid–binding domain had been predicted inside the viral protease region but never tested. Starting from the crystal structure, we located the pocket, docked six lipids chosen from the study's own lipidomics, and ran MD with MM/PBSA on the resulting complexes. Four long-chain fatty acids bound favourably and the two compounds that fell during infection did not, and the residues the modelling identified turned out to be conserved across exactly the genotypes that carry the domain. I have written about [what that study did and did not settle]({{ '/blog/2026/docking-md-hepatitis-e-virus/' | relative_url }}).

**Targeting the TEAD4–VGLL1 interface with peptides.** A collaboration with OneCureGen, presented at the AACR Annual Meeting 2024 {% cite choi2024vgll1 %}. This one is a protein–protein interface rather than a small-molecule pocket, which is the harder case: there is no obvious cavity to dock into, and the contact surface is large and shallow.

We worked it in stages. There was no structure of the complex, so we built one, using the TEAD4–YAP1 crystal structure as a template, and ran MD with MM/PBSA on both the full-length and truncated forms of each complex. Comparing them showed which parts of the interface actually carry the binding and which are along for the ride — the information needed to design a peptide that competes with the natural partner.

With the interface understood, we ran global docking of the collaborator's peptide library against TEAD4, analysed the contacts each peptide made, and returned a shortlist for experimental testing. We then modified the strongest candidates to improve binding and predicted ADMET properties for the optimised versions. The collaborator took those forward into the laboratory and identified lead peptides from them, which is the part that matters: the modelling narrowed a large library down to a set worth testing, and the bench work confirmed it.
