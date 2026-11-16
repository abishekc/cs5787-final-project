# Aims 1 and 2: cell-type classification with frozen DinoBloom-B, and comparison with the published benchmark

**Created 2026-11-15.** Answers the first two aims of the course proposal:

** Results and next steps summarized by Claude**

1. **Aim 1:** how well do frozen DinoBloom-B embeddings classify bone-marrow cell types in this paediatric leukaemia
   data set (BoneMarrowWSI-PediatricLeukemia), evaluated with patient-level train/test separation?
2. **Aim 2:** how does that compare with what the DinoBloom paper reports for adult bone marrow (the BMC data set,
   Matek et al. 2021)?

**No downloads.** Uses the 28,395 expert-annotated cell crops (113 patients) already on disk. The one comparison that
would need a download (plain DINOv2 ImageNet weights, to show what the haematology training adds) is switched off
until its size is quoted and approved.

Code and outputs: `classification_benchmark.ipynb`. Result tables are written to `data/benchmark_aims12/` (full-field
embeddings, out-of-fold predictions, result tables).

## Plan (written 2026-11-15, before any result of this notebook)

**Reference protocol** (DinoBloom paper, arXiv v1, Table 3, BMC): frozen features → 1-NN, 20-NN and a linear probe
(scikit-learn logistic regression); metrics weighted F1 (wF1), accuracy (Acc) and balanced accuracy (bAcc);
80/20 train/test split. Published DinoBloom-B on BMC (21 classes): 1-NN 79.6 / 79.5 / 65.8; 20-NN 83.7 / 84.1 / 57.1;
linear probe 85.5 / 85.6 / 70.7 (wF1 / Acc / bAcc).

**Representations (both frozen DinoBloom-B, CLS token, 224 px input):**
- **R-full (primary for Aim 2):** the stored 320 px field centred on the cell, resized to 224 px – closest to the
  single-cell images of BMC. Computed in this notebook.
- **R-tight:** the letterbox 5% tight crop used for the pseudotime work (already on disk).

**Class schemes:**
- **S-BMC (primary for Aim 2):** expert labels mapped to the BMC class definitions (mapping in the code below,
  fixed before results). BMC classes with no counterpart here (abnormal eosinophil, faggot cell, hairy cell) are
  absent. Excluded as not a white cell / not classifiable: erythrocytes, platelets and aggregates, spicules,
  neutrophil extracellular traps, `no_consensus_found`, `myeloid_precursor_cell`, degranulated myelocytes and
  metamyelocytes.
- **S-native:** the data set's own expert labels with ≥ 50 cells from ≥ 3 patients (same exclusions of
  `no_consensus_found`).

**Classifiers (as the paper):** 1-NN and 20-NN (cosine distance on L2-normalised features, majority vote);
linear probe = standardise → multinomial logistic regression, C = 1, lbfgs, no class weights. (The paper gives the
regularisation as "c×n/100" without further detail; C = 1 is our fixed choice, not tuned.)

**Splits:**
- **Primary:** 5-fold cross-validation **grouped by patient** (stratified group k-fold, seed 0) – no patient's cells
  in both training and test. Out-of-fold predictions pooled over the 5 folds.
- **Sensitivity:** 5-fold **cell-level** random stratified split (seed 0), as a stand-in for a non-patient-level
  80/20 split. The gap between the two shows how much patient leakage inflates scores.

**Reported:** wF1, Acc, bAcc for every combination; 95% CI by patient-cluster bootstrap (1,000 resamples) for the
primary rows; per-class F1 and confusion matrix for the primary linear probe; primary linear-probe bAcc and wF1 by
age band (≤ 5 y, 7–9.5 y, ≥ 12.5 y) and by CP-CML vs other diagnoses.

**Interpretation rules (fixed now):** the comparison with BMC is **descriptive** – different patients, labels,
class mix, stain and scanner – so no test of "better/worse". Two built-in reasons the published BMC numbers are
expected to be higher: BMC's training split was part of DinoBloom's pre-training data, and the BMC split may not have
been patient-level. This data set was not in DinoBloom's pre-training, so it is a genuine external test.

**Correction before any result (2026-11-15):** the first run stopped at the check that every label has a mapping
rule; four rare labels (11 cells) had none. Added: `basophilic_band` → BAS; `promegakaryocyte`,
`micromegakaryocyte`, `macrophage` → OTH. Nothing else changed.

## 6. Plain DINOv2 (ImageNet) baseline – switched off until the download is approved

What the haematology-specific pre-training adds over the general-purpose model it was built from. Needs the
DINOv2 ViT-B/14 ImageNet weights (`dinov2_vitb14_pretrain.pth`, ≈ 330–350 MB; exact size to be confirmed before
download). Same crops, same splits, same classifiers.

## Summary (written after the run, 2026-11-15)

**Run.** 28,395 crops from 113 patients. Of these, 24,716 cells fall in 18 BMC-mapped classes and 27,624 in 40 native classes. Full-field embedding took 9.5 min on MPS, classification 4.5 min, and the patient bootstrap about 40 min. No downloads; existing files were checked unchanged.

**Aim 1 – primary result** (BMC-mapped classes, full field, patient-level 5-fold CV, 95% CI from the patient bootstrap)

| Classifier | wF1 | Acc | bAcc |
|---|---|---|---|
| 1-NN | 63.3 (60.9–65.6) | 63.8 (61.5–66.2) | 50.2 (47.6–52.3) |
| 20-NN | 68.7 (66.7–70.6) | 70.7 (68.8–72.7) | 50.5 (48.6–52.7) |
| Linear probe | 68.9 (66.7–71.3) | 68.9 (66.7–71.3) | 56.2 (54.5–58.2) |

- **Tight crops do better throughout.** The letterbox crop used for the pseudotime work scores +2.3 to +3.4 points on every metric (linear probe 71.2 / 71.1 / 59.5), consistent with the earlier finding that the surrounding field adds noise.
- **Native labels (40 classes) are harder.** Linear probe gives wF1 63.7 and bAcc 43.4 on the full field, and 67.0 / 47.9 on tight crops.
- **Strong classes (F1):** blasts 0.88, "other" (mostly megakaryocytes) 0.88, erythroblasts 0.84, segmented neutrophils 0.80.
- **Weak classes:**
  - myelocyte 0.47 and metamyelocyte 0.43, confused with their neighbouring stages, which is consistent with a continuum;
  - non-identifiable/damaged cells 0.49;
  - lymphocytes 0.32, monocytes 0.22, immature lymphoid cells 0.15 and plasma cells 0.22 (16 cells).
- **Subgroups:**
  - By age band, bAcc is 54.9 / 57.0 / 56.0, so there is no clear age effect.
  - wF1 is higher in other diagnoses (75.2) than in CP-CML (63.8), because blast-dominated class mixes are easier. bAcc is similar (51.7 vs 54.1).

**Patient-level vs cell-level split.** Leakage inflates scores only modestly here: linear probe +2.3 wF1 (full field) and +1.6 (tight). 1-NN on the full field is actually *worse* with a cell-level split (57.0 vs 63.3). A plausible reason, which was not tested: neighbouring annotated cells have overlapping 35 µm fields, so a cell's nearest training neighbour is often the crop of an adjacent, differently labelled cell. With tight crops the drop disappears (67.4 vs 66.1).

**Aim 2 – vs the published BMC numbers (descriptive).** DinoBloom-B on adult BMC reports linear probe 85.5 / 85.6 / 70.7 and 1-NN 79.6 / 79.5 / 65.8. Here it scores 68.9 / 68.9 / 56.2 and 63.3 / 63.8 / 50.2. That is about 15–17 points lower in wF1 and about 14–16 points lower in bAcc. The gap combines several factors this design cannot separate:
1. a genuinely external data set, whereas BMC's training split was part of DinoBloom's pre-training;
2. strict patient-level evaluation;
3. a mapped, approximate label scheme with a large "non-identifiable/damaged" class (13% of cells);
4. a different scanner, stain and paediatric morphology.

The plain-DINOv2 baseline, which would show what the haematology pre-training adds on this data set, was not run because its download has not been approved.
