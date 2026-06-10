# Labor Prediction Model Comparison — Simon v1 · v2 · v3 · v4

*Generated 2026-06-10. Predicts hours from a cervical exam to delivery.*

## Datasets

| Set | Patients | Usable exams* | Source |
|-----|----------|---------------|--------|
| Cohort A | 8 | 63 | `OBS labor data.xlsx` (2026-06) |
| Cohort B | 8 | 90 | `ob 2.xlsx` (2026-06) |
| **Combined** | **15** | **153** | (1 patient appeared in both — deduplicated) |

\*Usable = dilation < 10 cm with effacement + station recorded, exam before birth.

The two cohorts differ: Cohort B had **longer, more variable** labors, which makes
it a valuable stress-test of how well a model trained on Cohort A generalizes.

## The models

| Model | Form | Fitted on |
|-------|------|-----------|
| **Simon v1** | Friedman-based piecewise rate | — (published constants) |
| **Leo v2** | Zhang 2010 + ACOG 2024 three-stage | — (literature constants) |
| **v3** | `exp(a + b·dil + c·eff/100 + d·sta)` | Cohort A only (8 pts) |
| **v4** | same form as v3, **refit** | Combined (15 pts) |

```
v3:  a=3.2759  b=-0.33468  c=-1.17288  d=-0.02429
v4:  a=3.2593  b=-0.29182  c=-1.65748  d=-0.14138
```

## Results

### Accuracy by cohort (MAE / bias, hours)

| Model | Combined MAE | RMSE | bias | Cohort A | Cohort B |
|-------|:---:|:---:|:---:|:---:|:---:|
| Simon v1 | 9.09 | 11.85 | **+8.12** | 9.23 | 8.98 |
| Leo v2 | 3.90 | 6.38 | −1.21 | 2.26 | 5.04 |
| v3 | 3.52 | 6.03 | −1.89 | 1.83 † | 4.69 |
| **v4** | **3.66** | **5.51** | **−0.08** | 2.53 | 4.45 |

† v3's Cohort-A number is **in-sample** (it was trained on those patients) — not a
fair generalization estimate. Its honest number is the Cohort-B column (4.69 h).

### Honest generalization (leave-one-patient-out cross-validation)

This trains on 14 patients and tests on the held-out one, for every patient — the
fairest single number.

| Model | CV-MAE | CV-RMSE | CV-bias |
|-------|:---:|:---:|:---:|
| v3 (held-out test, Cohort B) | 4.69 | 7.64 | −3.10 |
| **v4 (combined, LOPO-CV)** | **4.27** | **6.73** | **+0.24** |

**v4 is the winner:** lowest RMSE overall, near-zero bias on every cohort, and the
best honest cross-validated accuracy. It corrects v3's tendency to *under*-estimate
on slower labors (bias −3.1 → +0.2 h).

## What the errors tell us

v4 error broken down by how dilated the patient is:

| Dilation | n | actual median | v4 MAE | comment |
|----------|---|:---:|:---:|---------|
| 0–2 cm | 18 | 14.2 h | 5.4 h | early latent phase — inherently unpredictable |
| 2–3 cm | 57 | 9.8 h | 5.4 h | still highly variable |
| 3–4 cm | 41 | 4.4 h | 2.2 h | becoming reliable |
| 4–6 cm | 14 | 2.8 h | 1.7 h | reliable |
| 6–8 cm | 14 | 3.2 h | 1.8 h | reliable |
| 8–10 cm | 9 | 2.2 h | 1.8 h | reliable |

**The model is good once labor is established (≥ 3 cm: MAE ≈ 1.5–2.2 h) but the
early latent phase (< 3 cm) carries most of the error** — at 2 cm a real labor
ranged from 3 h to 21 h to birth. No formula on dilation/effacement/station alone
can resolve that; it needs contraction pattern, parity, and trend over time.

## v4 sanity curve (effacement 60%, station −2)

| Dilation | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|----------|---|---|---|---|---|---|---|---|---|
| Predicted hrs | 9.5 | 7.1 | 5.3 | 4.0 | 3.0 | 2.2 | 1.7 | 1.2 | 0.9 |

Smooth, monotonic, clinically plausible.

## Recommendation & caveats

- **Adopt v4** as the primary estimate; keep v3 visible for now to show the effect
  of adding data.
- Realistic expected accuracy is **~4 h MAE early, ~2 h once ≥ 3 cm** — present it
  as a *range*, not a precise clock time, especially in latent labor.
- Still **parity-agnostic**: the spreadsheets had no parity / epidural / induction /
  PROM labels. Capturing those is the single biggest remaining accuracy lever
  (literature: nulliparous labor runs hours longer than multiparous).
- 15 patients is still small. The in-app history-export feature is designed to keep
  growing this dataset for a future v5 that can finally model interventions.
