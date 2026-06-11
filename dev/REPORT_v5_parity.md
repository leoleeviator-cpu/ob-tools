# v5 / Parity Analysis — does parity improve the prediction?

*Generated 2026-06-11. Adds parity labels for the 15 prior patients and one new
patient (ID 25723463), and tests whether a parity term improves the model.*

## Data

| Set | Patients | Usable exams (dil<10) | Notes |
|-----|----------|----------------------|-------|
| Previous (cohorts A+B) | 15 | 141 | now labelled with **parity** (11× P0, 3× P1, 1× P2) |
| New (ID 25723463) | 1 | 21 | **extreme prolonged latent**: ~40 h at 2 cm, then 3 cm → birth in <2 h. Parity not provided. |

Only **4 of 15** prior patients are multiparous (parity ≥ 1) — a very thin
multipara sample.

## The new patient is a textbook latent-phase case
Stuck at 2–2.5 cm for ~40 hours, then delivered <2 h after reaching 3 cm. No
model on dilation/effacement/station can predict that 40 h plateau — at 2 cm a
real labor in this dataset ranges from ~1.4 h to ~46 h to birth. It reinforces,
rather than fixes, the known latent-phase limit.

## Does parity help? (leave-one-patient-out CV)

| Regime | n | no parity (MAE / bias) | **+ parity (MAE / bias)** |
|--------|---|:---:|:---:|
| All (dil < 10) | 141 | 5.98 h / +0.60 | 6.92 h / +1.02 (worse) |
| **Established (dil ≥ 3 cm)** | 71 | 1.80 h / −0.23 | **1.50 h / +0.05** |
| ≥ 3 cm, outlier excluded | 66 | 1.87 h / −0.21 | **1.56 h / +0.06** |

**Finding:** parity helps *exactly where the tool is already reliable* (≥ 3 cm):
~17 % lower error and bias driven to ~zero. In the latent phase (< 3 cm) it does
not help — that error is irreducible with these inputs, and the prolonged-latent
cases (incl. the new patient) dominate the "all" row and mask/​reverse the benefit.

## Why we are NOT shipping a v5 yet
The fitted coefficients are **not stable enough for a clinical tool**:

```
v5 (+parity) full fit:  station coef = +0.27   ← wrong sign (should be negative:
                                                   more-descended = shorter labor)
multipara multiplier:   0.58×  (all data)  vs  0.31×  (≥3 cm)   ← swings with regime
```

- The **positive station coefficient** is a confounding artifact (station advances
  together with dilation; the many station = −3 latent points with both short and
  very long times destabilise it). A wrong-signed term could mislead in edge cases.
- The **multipara effect** is real in direction (multiparas faster) but its size is
  unreliable on only 4 multiparous patients.

15 patients / 4 multiparas is simply too thin to re-fit a clinical algorithm
responsibly.

## Recommendation
1. **Keep v4 live unchanged** for now.
2. **Parity is the right lever** — the ≥ 3 cm result (1.8 → 1.5 h) is promising and
   matches the literature (multiparas labour faster). Capture it.
3. **Keep collecting**: the in-app Firestore tracking now accumulates real exams
   centrally. Re-run this analysis once there are ~10+ multiparous patients; then a
   parity-aware v5 with stable, correctly-signed coefficients becomes defensible.
4. The new patient (ID 25723463) is added to the growing dataset for that future fit.

## Reference coefficients (for the future, not deployed)
```
v5 (+multipara), all dil<10:  exp(4.6712 - 0.1824·dil - 2.8640·(eff/100) + 0.2712·sta - 0.5368·multipara)
                              (station sign unreliable — do not deploy as-is)
```
