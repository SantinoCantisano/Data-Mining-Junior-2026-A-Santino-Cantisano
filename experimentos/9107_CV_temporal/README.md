# 9107 — CV temporal

## Objetivo

Atacar el problema estructural del pipeline: validation con 1 solo mes (202107) puede overfitear a ese mes específico. Testar si validation multi-mes mejora la generalización.

## Cambio respecto al 9106

- Training: `[201901-202104]` (era [201901-202105])
- Validation: `[202105, 202106, 202107]` (era [202107])
- Resto igual (mismos bounds, mismo presupuesto BO, mismo FE con ratios)

Implementación: validation set concatenado (los 3 meses como un único `dvalidate` de ~100k filas). Early stopping y AUC objetivo se calculan sobre el conjunto agregado.

## HP encontrados

| HP | 9106 | 9107 |
|---|---|---|
| learning_rate | 0.01394 | 0.00804 |
| num_leaves | 384 (topó) | 384 (topó) |
| feature_fraction | 0.7158 | 0.5059 |
| min_sum_hessian_in_leaf | 18.451 | 10.060 |
| num_iterations | 379 | 812 |
| AUC validation | 0.9335 | **0.9355** (la más alta) |

Con validation más estable, la BO descubre que `lr` bajo + muchas iteraciones + regularización moderada converge mejor.

## Resultado

- Wilcoxon vs 9105: p=0.844 (dirección OPUESTA, 9105 mejor)
- Wilcoxon vs 9106: p=0.781 (dirección OPUESTA, 9106 mejor)
- Sd individual de las 5 semillas: **6.58** (la más alta)

## Interpretación

**Confirmación empírica del Experimento 1 de las slides del profesor**: AUC en validation y Public en el mes futuro son casi independientes. El CV temporal produce modelos con HP más "afilados" (lr bajo, muchas iters) que rinden mejor en la métrica de validation pero peor (y más volátil) en Public.

## Decisión: NO ES EL SUBMIT FINAL

A pesar del argumento teórico a favor (validation más honesta), la evidencia empírica va en contra. Se mantiene como experimento válido para el reporte académico.

Ver [RESULTS.md § 9107](../../RESULTS.md#9107).
