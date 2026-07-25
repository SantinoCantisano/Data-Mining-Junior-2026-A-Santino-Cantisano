# 9102 — BO agresiva (DESCARTADO)

## Objetivo

Reemplazar el Grid Search del 9101 por Bayesian Optimization con bounds amplios, siguiendo la guía del profesor Denicolay sobre HP a optimizar y sus rangos teóricos.

## Cambio respecto al 9101

- Grid Search → BO con `ParBayesianOptimization`
- Bounds según guía del profesor: `learning_rate ∈ [0.001, 1.5]`, `num_leaves ∈ [2, 1024]`, `feature_fraction ∈ [0.05, 1.0]`, `min_sum_hessian_in_leaf ∈ [1e-8, 1000]`
- HP fijos según guía: `min_data_in_leaf=0`, `max_bin=31`, `is_unbalance=FALSE`, `boosting=gbdt`, lambdas=0, bagging=1.0/0
- Presupuesto: 8 initPoints + 40 iters = 48 evaluaciones

## Resultado

- HP encontrados: `num_leaves=1024` (topó techo), `min_sum_hessian_in_leaf=1e-8` (topó piso)
- AUC validation: 0.9327 (sospechosamente alta)
- Wilcoxon vs 9101: p-value = 0.9062 (dirección OPUESTA — 9101 mejor)

## Decisión: DESCARTADO

Los HP topando los bounds indican que la BO estaba pidiendo capacidad extrema y regularización mínima — firma de overfit al validation. El semillerío enmascaraba modelos individualmente sobreajustados.

## Lección

Cuando un HP topa un bound extremo, es señal de que **el espacio de búsqueda no era suficiente** — pero antes de ampliarlo, chequear si el AUC validation es sospechosamente alta (sugiere overfit).

Ver [RESULTS.md § 9102](../../RESULTS.md#9102).
