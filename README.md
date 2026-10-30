# Do DCE-MRI phases improve pCR prediction in breast cancer?

**A patient-level, cross-cohort ablation study on the public BreastDCEDL dataset**

> **Status:** work in progress (started October 2026). The study protocol is being revised. No results yet.

## Research question

Before treatment, do the **early** and **delayed** contrast phases of breast DCE-MRI improve the prediction of **pathologic complete response (pCR)** to neoadjuvant chemotherapy, beyond the pre-contrast image and clinical variables? And does any benefit **hold when the model is tested on a different cohort**?

## Why this matters

Deep learning models for pCR usually stack several DCE phases together without testing whether each phase helps. Studies that compare phases are small and single-centre. Large public benchmarks on the same cohorts report weak external performance from pre-treatment MRI ([MAMA-MIA challenge, 2026](https://arxiv.org/abs/2603.01250)). This project tests, phase by phase and across cohorts, where the predictive signal comes from.

## Study design

| Arm | Input | Purpose |
|---|---|---|
| C | Clinical variables only (age, HR, HER2) | Clinical baseline |
| I0 | Pre-contrast MRI | Imaging baseline |
| I1 | Pre-contrast + early phase | Value of early enhancement |
| I2 | Pre-contrast + early + delayed phase | Value of full dynamic information |
| C+I | Clinical + best imaging arm | Added value of imaging |

- **Model:** 2D ResNet-18 on tumour slices; slice features pooled into one prediction per patient.
- **Cohorts:** develop on **I-SPY2** (n = 982), external test on **I-SPY1** (n = 172) and **Duke** (n = 298 with pCR labels).
- **Evaluation:** AUROC, AUPRC, sensitivity, specificity, Brier score, calibration; patient-level bootstrap confidence intervals; subgroup results by molecular subtype.
- **Rules:** splits are by patient and by cohort; slices from one patient are never treated as independent samples; external cohorts are never used for tuning.
- **Reporting:** [TRIPOD+AI](https://pmc.ncbi.nlm.nih.gov/articles/PMC11025451/) and [CLAIM 2024](https://pmc.ncbi.nlm.nih.gov/articles/PMC11304031).

## Data

This project uses **[BreastDCEDL](https://github.com/naomifridman/BreastDCEDL)** (Fridman et al., *Scientific Data* 13:264, 2026), built from three TCIA collections. **No images or patient data are stored in this repository.** Download them yourself:

| Source | Where | Licence |
|---|---|---|
| BreastDCEDL (cropped release) | [Zenodo 18114231](https://zenodo.org/records/18114231) | CC BY-NC 4.0 |
| BreastDCEDL metadata and code | [GitHub](https://github.com/naomifridman/BreastDCEDL) | see repository |
| I-SPY1 | [TCIA, doi:10.7937/K9/TCIA.2016.HdHpgJLK](https://doi.org/10.7937/K9/TCIA.2016.HdHpgJLK) | CC BY 3.0 |
| I-SPY2 | [TCIA, doi:10.7937/TCIA.D8Z0-9T85](https://doi.org/10.7937/TCIA.D8Z0-9T85) | CC BY 4.0 |
| Duke-Breast-Cancer-MRI | [TCIA, doi:10.7937/TCIA.e3sv-re93](https://doi.org/10.7937/TCIA.e3sv-re93) | CC BY-NC 4.0 |

Use of the data must follow the [TCIA Data Usage Policies](https://www.cancerimagingarchive.net/data-usage-policies-and-restrictions/). Because Duke data are non-commercial, this project is for **non-commercial research only**.

## Repository structure

```
.
├── README.md
├── LICENSE
├── .gitignore
├── docs/                     # proposal and literature review
├── step1_literature/
│   └── make_litreview.py     # builds the literature review spreadsheet
├── step3_patient_table/
│   ├── build_patient_table.py  # one row per patient from BreastDCEDL metadata
│   └── make_workbook.py        # summary, data issues and column guide
├── preprocessing/            # planned: phase selection, normalisation, tumour crops
├── models/                   # planned: ResNet-18 + patient-level pooling
└── analysis/                 # planned: metrics, bootstrap CIs, calibration
```

## Quick start

```bash
git clone https://github.com/<your-username>/<this-repo>.git
cd <this-repo>
pip install pandas numpy openpyxl nibabel

# metadata only (no images needed)
git clone https://github.com/naomifridman/BreastDCEDL.git ../BreastDCEDL
python step3_patient_table/build_patient_table.py --repo ../BreastDCEDL --out data/patient_table.csv
cd data && python ../step3_patient_table/make_workbook.py
```

After downloading the NIfTI masks, recount tumour slices exactly:

```bash
python step3_patient_table/build_patient_table.py --repo ../BreastDCEDL --out data/patient_table.csv \
    --mask-dir spy2=/path/to/spy2_masks spy1=/path/to/spy1_masks duke=/path/to/duke_masks
```

## Known data issues

Found while building the patient table (full list in the workbook's *Data issues* tab):

- **"Early" and "late" are chosen by scan number, not by time after injection.** Early is the 2nd post-contrast scan in I-SPY2, but the 1st in Duke and in most I-SPY1 patients.
- **Treatment differs by cohort.** I-SPY2 has 13 treatment arms; treatment is not available for I-SPY1 or Duke.
- **Menopause is missing for all I-SPY1 patients,** and tumour volume is not measured the same way in every cohort.
- **Duke has bounding boxes, not 3D tumour masks.**

## Roadmap

- [x] Step 1: literature review and novelty check
- [ ] Step 2: download and inspect BreastDCEDL
- [x] Step 3: patient-level table (from metadata; exact slice counts pending)
- [ ] Step 4: preprocessing (phase timing, normalisation, crops)
- [ ] Step 5: fixed data splits
- [ ] Step 6: clinical baseline
- [ ] Steps 7-9: imaging arms I0, I1, I2 and C+I
- [ ] Steps 10-12: external evaluation, statistics, calibration, subgroups
- [ ] Step 13: paper


## Citation

If you use this code, please cite this repository (a citation file will be added with the first release) and the BreastDCEDL paper:

> Fridman N, Solway B, Fridman T, Barnea I, Goldstein A. BreastDCEDL: a standardized deep learning-ready breast DCE-MRI dataset of 2,070 patients. *Scientific Data* 13, 264 (2026). https://doi.org/10.1038/s41597-026-06589-6

## Licence

Code: MIT (see `LICENSE`). Data are **not** covered by this licence; they keep their original licences listed above.

## Disclaimer

This is a research project. The models are **not** intended for clinical use.
