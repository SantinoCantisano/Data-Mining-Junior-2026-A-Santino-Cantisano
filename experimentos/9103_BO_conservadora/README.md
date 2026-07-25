# 9103 — BO conservadora

## Objetivo

Testar la hipótesis de que el 9102 estaba en zona de overfit, forzando bounds con regularización mínima obligatoria.

## Cambio respecto al 9102

- `num_leaves` cap: **1024 → 256** (modelo menos complejo)
- `min_sum_hessian_in_leaf` piso: **1e-8 → 1e-3** (regularización mínima obligatoria)
- Resto igual

## Resultado

- HP encontrados: `num_leaves=256` (topó techo), `min_sum_hessian_in_leaf=23.997` (**NO topó piso**, elegido libremente)
- AUC validation: 0.9337 (igual al 9102)
- Wilcoxon vs 9101: p-value = 0.156 direccional, media +4.25, sd cae de 3.74 a 2.12

## Decisión: ACEPTADO como nuevo baseline

Con AUC validation prácticamente igual pero Public mejor, se confirma que el 9102 estaba en overfit. Los HP moderadamente regularizados generalizan mejor. `min_sum_hessian_in_leaf` sin topar bound es evidencia positiva.

## Lección

Cuando la BO tiene libertad de elegir regularización mínima y no lo hace, la regularización moderada es genuinamente óptima — no un artefacto de bounds.

Ver [RESULTS.md § 9103](../../RESULTS.md#9103).
