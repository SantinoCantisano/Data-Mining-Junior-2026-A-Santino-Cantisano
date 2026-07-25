# 9106 — BO recalibrada

## Objetivo

Testar si los HP hardcoded del 9103 usados en el 9105 seguían siendo óptimos con el dataset ampliado por ratios/indicadores. Hipótesis: `num_iterations=347` estaba calibrado sin ratios y podría cambiar.

## Cambio respecto al 9105

- Re-corre BO con el dataset del 9105 (con ratios)
- Bounds levemente ampliados: `num_leaves` cap 256 → **384**
- Presupuesto ampliado: 10 initPoints + 60 iters = **70 evaluaciones** (vs 36 del 9103)

## HP encontrados

| HP | 9103 | 9106 |
|---|---|---|
| learning_rate | 0.01318 | 0.01394 |
| num_leaves | 256 (topó) | 384 (topó) |
| feature_fraction | 0.4215 | 0.7158 |
| min_sum_hessian_in_leaf | 23.997 | 18.451 |
| num_iterations | 347 | 379 |
| AUC validation | 0.9337 | 0.9335 |

`feature_fraction` subió mucho (0.42 → 0.72) — con más features disponibles la BO quiso ver más columnas por split.

## Resultado

- Wilcoxon vs 9105 (5 pares): p=0.844 (dirección OPUESTA, 9105 mejor)
- Sd individual de las 5 semillas: **0.34** (extraordinariamente baja)
- Ensemble k=20 Public máximo: 88.97

## Decisión: EMPATE estadístico con 9105

Los HP son diferentes pero la AUC validation es idéntica y el Public es equivalente. La sd 0.34 en individuales indica modelos casi idénticos entre semillas (feature_fraction alto reduce diversidad implícita).

## Uso

El 9106 quedó como candidato de diversificación estructural para ensambles multi-modelo (ver `analysis/ensemble_multimodelo.ipynb`).

Ver [RESULTS.md § 9106](../../RESULTS.md#9106).
