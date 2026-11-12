# CGMH ISS Test Set (n = 100)

Physician-verified test set used in:

> Lin Y-J, Fang K-Y, Liao C-H, Cheng C-T. **Automating Injury Severity Score Calculation From Free-Text Discharge Diagnoses Using Domain-Specific Retrieval-Augmented Large Language Models: Retrospective Validation Study.** *JMIR Med Inform.* 2026;14:e93922. doi:[10.2196/93922](https://doi.org/10.2196/93922)

Each record is one trauma admission from the Chang Gung Memorial Hospital (CGMH) trauma registry, with the free-text discharge diagnosis that was given to the models as input and the physician-verified Abbreviated Injury Scale (AIS) severities and Injury Severity Score (ISS) used as ground truth.

## Files

| File | Description |
|---|---|
| `data/cgmh_iss_testset_100.csv` | The test set, UTF-8, one row per case |
| `data/cgmh_iss_testset_100.json` | Same content as a JSON array |

## Columns

| Column | Type | Description |
|---|---|---|
| `ori_index` | int | Case identifier. Internal row index of the de-identified registry extract; it matches the `index` values printed in the model logs released as Multimedia Appendix 2 of the paper. Not a medical record number. |
| `discharge_diagnosis` | str | Free-text discharge diagnosis (English, clinician-written). This is the **only model input** used in the paper. Kept verbatim apart from date masking (see below): full-width commas, missing spaces between numbered items, typos and occasional Chinese section labels are present in the source and were not corrected. |
| `icd10cm_dx1` … `icd10cm_dx9` | str | Up to 9 ICD-10-CM diagnosis codes recorded by the hospital coders (with dots, e.g. `S52.562A`). Used only for the ICDPIC-R baseline; the LLMs and the physician reviewer did not see them. An unused slot contains a single space character (see note below). |
| `ais_<region>_1` … `ais_<region>_3` | int | Physician-verified AIS severity (1–6) of up to three injuries per ISS body region. Regions: `head_neck`, `face`, `chest`, `abdomen`, `extremities`, `external`. An unused slot contains a single space character (see note below). |
| `iss` | int | Ground-truth ISS. Equals the sum of squares of the three highest region maxima (75 if any AIS = 6); verified to recompute exactly from the AIS columns for all 100 cases. |
| `niss` | int | New Injury Severity Score from the registry (not evaluated in the paper). |
| `iss_stratum` | int | Sampling stratum used for stratified selection (registry column `group`): 1 = ISS 0–9, 2 = 10–16, 3 = 17–25, 4 = 26–36, 5 = 37–75. |
| `Blunt`, `Penetrating` | str | Injury-mechanism flags copied as is from the registry: `☉` in the ticked column and an empty string in the other (93 blunt, 4 penetrating); in 3 cases neither is ticked and both cells hold a single space. |

**Note on empty cells.** Cells are copied byte-for-byte from the registry extract, in which an unused ICD or AIS slot holds a single space character (`" "`), not an empty string. The CSV therefore shows `, ,` rather than `,,` and the JSON holds `" "`. Treat whitespace-only values as missing, e.g. `pd.read_csv(path, skipinitialspace=True)` or `df.replace(r"^\s*$", pd.NA, regex=True)`.

## Severity distribution

| Sampling stratum | n | | Reporting category (paper) | n |
|---|---|---|---|---|
| ISS 0–9 | 70 | | Mild (1–8) | 45 |
| ISS 10–16 | 12 | | Moderate (9–15) | 28 |
| ISS 17–25 | 12 | | Severe (16–24) | 13 |
| ISS 26–36 | 4 | | Profound (25–75) | 14 |
| ISS 37–75 | 2 | | | |

## De-identification

The registry extract was de-identified by the hospital before the research team received it (no names, medical record numbers, or contact details). For public release we additionally removed all demographic and encounter fields that were not used in the study (sex, age, admission/discharge timestamps, length of stay, discharge disposition, TRISS).

The free text is released exactly as it was given to the models, including procedure dates written by the clinicians (e.g. "s/p ORIF on 2022/06/09"), so that the published results can be reproduced verbatim.

## Caveat: one case states the answer in the text

Case `ori_index` 4168 is a discharge diagnosis in which the clinician wrote the AIS severities and the ISS arithmetic directly into the free text:

```
1.Right pelvic fracture 〈AIS:2〉2.Right high front-parietal minimal subdural hematoma 〈AIS:4〉〈附外片〉           ISS: 2x2 + 4x4 ＝ 20
```

A model can therefore copy the answer from this case instead of inferring it. We deliberately did **not** remove or edit it: discharge diagnoses in a real trauma registry sometimes contain such annotations, and the test set is meant to reflect what a deployed system would actually receive. The case is also part of the 100 cases evaluated in the paper, so removing it would break comparability with the published numbers.

Because the test set has 100 cases, this single case can shift any accuracy figure by at most 1 percentage point. If you want a benchmark in which the ISS must be inferred from the injury descriptions alone, exclude `ori_index` 4168 (n = 99) and say so when reporting.

## Reproducing the ICDPIC-R baseline

The paper's ICDPIC-R baseline (Clark et al, 2018) was run with `icdpicr::cat_trauma()` on the nine ICD-10-CM codes with dots removed (`icd10 = "cm"`). Strip the dots from `icd10cm_dx1` … `icd10cm_dx9` to obtain the identical input.

## Ethics

The study was reviewed and approved by the institutional research ethics board (approval number 202500438B0); the requirement for informed consent was waived for retrospective analysis of de-identified records. The full trauma registry cohort (n = 4,392) is not publicly available under CGMH data governance policy.

## License

- Data (`data/`): [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) (see `LICENSE-DATA`). Non-commercial use only; please cite the paper above.
- Any code in this repository: MIT (see `LICENSE`).

## Please cite our paper
Please cite the paper above if you use this test set in your research.
```
@Article{info:doi/10.2196/93922,
author="Lin, Ying-Jia
and Fang, Kuan-Yueh
and Liao, Chien-Hung
and Cheng, Chi-Tung",
title="Automating Injury Severity Score Calculation From Free-Text Discharge Diagnoses Using Domain-Specific Retrieval-Augmented Large Language Models: Retrospective Validation Study",
journal="JMIR Med Inform",
year="2026",
month="Oct",
day="13",
volume="14",
pages="e93922",
keywords="Abbreviated Injury Scale; automatic ISS calculation; large language models; retrieval-augmented generation; trauma",
issn="2291-9694",
doi="10.2196/93922",
url="https://medinform.jmir.org/2026/1/e93922",
url="https://doi.org/10.2196/93922"
}
```

## Contact

Intelligent Language Algorithms Laboratory (ILALab), Department of Artificial Intelligence, Chang Gung University.
Open a GitHub issue for questions about the data.
