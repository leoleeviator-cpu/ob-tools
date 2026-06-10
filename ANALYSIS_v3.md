# v3 Prediction Model — Optimization Against Real Labor Data

## Dataset
- Source: `OBS labor data.xlsx` (8 parturients, 2026-06)
- 94 cervical exams total; **63 usable** observations (dilation < 10 cm with
  effacement + station recorded). Each exam's *actual* hours-to-birth was
  derived from the recorded "baby birth time".
- No parity / epidural / induction / PROM labels were available, so v3 is a
  **parity-agnostic base curve**.

## Benchmark of existing models (vs real time-to-birth)

| Model | MAE | RMSE | Bias |
|-------|-----|------|------|
| Simon v1 (nullipara) | 22.6 h | 30.7 h | +22.2 h |
| Simon v1 (multipara, app default) | 9.2 h | 12.5 h | +8.5 h |
| Leo v2 (nullipara) | 5.4 h | 6.3 h | +5.3 h |
| Leo v2 (multipara, app default) | 2.3 h | 2.7 h | +0.6 h |

Key finding: the nullipara branches massively over-estimate; even the best
existing variant carries a systematic +0.6 h positive bias.

## v3 model

```
hours_to_birth = exp( a + b·dilation + c·(effacement/100) + d·station )

a = 3.27592   b = -0.33468   c = -1.17288   d = -0.02429
```

Fitted by least-squares; functional form chosen by comparing 6 candidate
forms under **leave-one-patient-out cross-validation** (the exponential
log-linear form was the most accurate *and* most robust, with no degenerate
parameters).

| | MAE | RMSE | Bias |
|---|-----|------|------|
| v3 (leave-one-patient-out CV) | **2.07 h** | **2.47 h** | **−0.12 h** |
| v3 (in-sample) | 1.83 h | 2.20 h | −0.17 h |

v3 beats v2 on accuracy and effectively removes the systematic over-estimation.

## Caveats
- Only 8 patients — treat v3 as a **first-pass empirical calibration**, not a
  validated clinical tool.
- Station's independent effect is weak/confounded in this sample (station
  advances together with dilation), so its coefficient is small.
- Interventions (epidural/induction/PROM) are shown as badges but do **not**
  yet shift the v3 estimate — they need labelled data to calibrate.
- Re-fit as more real birth-time data accumulates via the history-tracking
  feature.
