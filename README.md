[README.md](https://github.com/user-attachments/files/34234687/README.md)
# Variance Decomposition and the Detectability of Load–Recovery Effects in Consumer Wearable Data

Reproducibility package accompanying the paper submitted to **TCS 2027** (ACM International Conference Proceedings Series).

The study asks not whether training load and recovery are related — the literature assumes they are — but whether that relationship can be **detected** in the data consumer wearables actually produce, and which analysis decisions determine the answer. Three cohorts are used:

| Cohort | Source | n | Load | Outcome |
|---|---|---|---|---|
| A | Apple Watch, two volunteers, calendar year 2026 | 2 | device workout duration | next-day SDNN, resting HR |
| B | PMData (Simula Research Laboratory) | 16 | sRPE, very-/moderately-active minutes | next-day readiness (0–10) |
| C | LifeSnaps (Yfantidou et al., 2022, CC BY 4.0) | up to 71 | activity minutes, steps, calories | next-day RMSSD |

---

## Repository contents

### Analysis code

| File | Purpose | Produces |
|---|---|---|
| `scan_health.py` | Streaming parser for the Apple Health `export.xml` (7.06 M records, 2.9 GB — never loaded into memory). De-duplicates overlapping sleep-stage intervals and repeated workout writes. | `health_data/` |
| `analyze_health.py` | Coverage and feasibility assessment of the parsed streams. | `stats.json` |
| `qc_health.py` | Outlier, missingness and distribution checks on the day-level feature table. | `qc_report.json` |
| `compare_two.py` | Between- versus within-person variance decomposition for the two-person cohort; ICC, discriminability, pooling-versus-normalisation comparison. | `compare_two.json` |
| `pmdata_analysis.py` | Cohort B: variance components and load → next-day readiness models under three load definitions. | `pmdata_result.json` |
| `lifesnaps_analysis.py` | Cohort C: large-sample ICCs and the five device-only load definitions. | `lifesnaps_result.json` |
| `power_analysis.py` | Synthetic effect injection (0.10–0.60 z) and minimum detectable effect at 80% power. | `power_analysis.json` |
| `revision_checks.py` | Sleep-covariate sensitivity, harmonised sample sizes, clustered versus model-based standard errors. | `revision_checks.json` |
| `cohortB_se.py` | Cluster-robust standard errors for Cohort B under both error conventions. | `cohortB_se.json` |
| `remake_figure2.py` | Figure 1 of the paper (forest plot of every specification, at print size). | `figures/figure1_forest.{png,pdf}` |
| `make_figures.py` | Alternative figure renderings used during revision. | `figures/` |
| `verify_refs.py` | Crossref verification of every reference's DOI and metadata. | `ref_verify.json` |

### Data included

| File | Description |
|---|---|
| `daily_features_p1.csv` | Cohort A, Participant A (first author): 288 days × 19 day-level features. |
| `daily_features_p2.csv` | Cohort A, Participant B (third author): 224 days, same schema. Self-tracked data, published with his consent. |
| `references.bib` | Bibliography, 15 entries, all DOIs verified against Crossref. |

### Results included

`compare_two.json`, `pmdata_result.json`, `lifesnaps_result.json`, `power_analysis.json`, `cohortB_se.json`, `revision_checks.json`, `qc_report.json`, `ref_verify.json`. Every number reported in the paper can be traced to one of these files.

---

## Data not included

**Cohort B (PMData)** and **Cohort C (LifeSnaps)** are redistributed by their original providers and are therefore not duplicated here. Place them at the paths below before running the corresponding scripts.

```
pmdata/                    # PMData: https://datasets.simula.no/pmdata/
lifesnaps/rais_anonymized/csv_rais_anonymized/
                           # LifeSnaps: https://github.com/social-ai-uo/lifesnaps-dataset (CC BY 4.0)
export.xml                 # Apple Health export, or set APPLE_HEALTH_XML
```

By default the parser looks for `export.xml` next to this file; override with:

```bash
export APPLE_HEALTH_XML="/path/to/apple_health_export/export.xml"
```

The raw Apple Health exports (`health_data/`, `health_data_p2/`, 5.5 MB of per-record device data) are **not** published: they contain minute-level heart rate and location-adjacent records beyond what the study needs or the participants agreed to.

---

## Requirements

Python 3.9 or later. Analysis scripts use the standard library only; figure scripts need matplotlib.

```bash
pip install -r requirements.txt
```

## Reproducing the results

```bash
python compare_two.py        # Tables 1 and 2, Cohort A columns; Section 4.1
python pmdata_analysis.py    # Table 2, Cohort B rows; Section 4.2
python cohortB_se.py         # Table 2 clustered vs model-based t; Section 4.2
python lifesnaps_analysis.py # Section 4.3, five load definitions, n = 36
python power_analysis.py     # Section 3.5, synthetic effect injection
python revision_checks.py    # Section 3.5, sleep-covariate sensitivity
python remake_figure2.py     # Figure 1
```

Each script prints a summary and writes its JSON alongside itself, so a full re-run reproduces every table and figure in the paper.

---

## Method notes worth reading before the code

- **The independent unit is the participant, not the day-pair.** Coefficients carry standard errors clustered on participant wherever more than one contributes, with the correction `G/(G−1)·(n−1)/(n−k)`. Cohort A has one cluster per participant, so its model-based and clustered errors coincide by construction. Both are reported in Table 2 rather than only the flattering one.
- **The autoregressive form is deliberate.** The change-score form `ΔY = Y(t+1) − Y(t)` was rejected because on Cohort A it made higher load appear to *increase* next-day HRV (z ≈ 2.7) — regression to the mean, since high-load days were days on which same-day HRV was already depressed.
- **Rest days scored zero, not missing.** A zero-load day is information; a dropped day is not.
- **Normalisation is descriptive, not causal.** Within-subject standardisation uses each participant's own mean and SD over the observation window. A deployed system would need a trailing window instead.
- **Cohort A's variance ratios are descriptive.** With two units there is no sampling distribution from which a standard error could be built, so those columns support no parameter inference.

---

## Ethics

Both Cohort A participants are authors of the paper: Participant A is the first author and Participant B is the third author. The study therefore analyses the authors' own self-collected data rather than data from third-party participants, and both authors consented to the publication of the derived day-level feature tables. All parsing and analysis were performed locally; no health data were transmitted to external services. Cohorts B and C are public datasets used under their stated licences.

## Licence

Code: MIT (see `LICENSE`). Derived feature tables: CC BY 4.0.

## Citation

```bibtex
@inproceedings{pan2027variance,
  author    = {Pan, Cheng and Huang, Minghao and Chen, Chang},
  title     = {Variance Decomposition and the Detectability of Load--Recovery Effects in Consumer Wearable Data},
  booktitle = {Proceedings of TCS 2027},
  year      = {2027},
  publisher = {ACM}
}
```
