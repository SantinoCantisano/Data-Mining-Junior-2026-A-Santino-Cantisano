# 9101 — Baseline con Grid Search + semillerío k=5

## Objetivo

Establecer el baseline honesto contra el que comparar todos los experimentos futuros. Utiliza el pipeline entregado por la cátedra sin modificaciones estructurales, agregando semillerío k=5 para medir la varianza intrínseca del modelo.

## Cambios respecto al pipeline original de la cátedra

Ninguno estructural. Se agregó semillerío k=5 (loop sobre 5 semillas del LightGBM final) con guardado individual de probabilidades por semilla para análisis posterior.

## Configuración

- Training: `[201901-202105]`
- Validation: `[202107]`
- Future: `202109`
- Tuning: Grid Search sobre `num_leaves × min_data_in_leaf × feature_fraction`
- Semillas base: `804043, 653561, 703903, 439693, 665857`

## Notebooks

1. `9101_workflow_k5.ipynb` — pipeline completo con semillerío k=5
2. `9101_submit_individuales.ipynb` — sube las 5 semillas individuales a Kaggle para tests de Wilcoxon posteriores
3. `9101_escalar_k20.ipynb` — extiende a k=20 sin re-entrenar las 5 iniciales

## Resultado

Ver [RESULTS.md § 9101](../../RESULTS.md#9101).

- Public máximo k=5: 85.65 (corte 2000)
- Sd individual: 3.74
- Sd ensemble k=20: 1.44
