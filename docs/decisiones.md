# Justificación de decisiones de diseño

Este documento explica las decisiones metodológicas tomadas durante el desarrollo, más allá de los resultados numéricos. Está pensado como material de lectura para entender el porqué del enfoque, no solo el qué.

## Principio guía: cambiar UNA cosa a la vez

Durante todos los experimentos se cambió exactamente una dimensión del pipeline por vez. Esto permite atribución causal: si el 9105 mejora sobre el 9103, el motivo es exclusivamente el FE nuevo (porque los HP son idénticos). Si el 9106 empata al 9105, el motivo es exclusivamente los HP recalibrados (porque el FE es idéntico).

Alternativa rechazada: cambiar varias cosas a la vez para "acelerar" el ciclo. Esto habría producido resultados irrepetibles porque no se podría saber qué cambio aportó qué.

## Por qué semillerío desde el inicio

El **Experimento 1 de las slides** muestra que con distintas semillas del mismo modelo, la ganancia Public varía en el rango ~19.0 a 20.5 (sd ~0.35 en el ejemplo del profesor; en este proyecto medimos sd 3.74 con n=5). Una sola corrida es una tirada de dado.

Sin semillerío, cualquier comparación entre experimentos habría sido inútil: no se puede saber si un cambio de +3 puntos entre modelos A y B es un cambio real o dos tiradas distintas del mismo modelo.

**Cuántas semillas**: el profesor sugiere k=100 como referencia. En este proyecto se usó k=5 para experimentación rápida (Wilcoxon con 5 pares), k=20 para validación intermedia, k=50 y k=100 para el submit final. Cada duplicación reduce la sd Private esperada por factor √2.

## Por qué test de Wilcoxon pareado

Alternativa: t-test pareado. Rechazado porque asume normalidad en las diferencias, lo cual no se puede validar con n=5.

El Wilcoxon signed-rank pareado no requiere normalidad, solo simetría de la distribución de las diferencias. Con n=5, el p-value mínimo posible es 0.0625 (cuando los 5 pares van en la misma dirección). Un p<0.15 con n=5 es "evidencia direccional aceptable" — no significativo por convención pero informativo cuando las 5 tiradas van en la misma dirección.

## Por qué corte 2100 para los tests de Wilcoxon, corte 2000 para el submit

- Los tests de Wilcoxon se hicieron con el corte donde el ensemble del experimento tuvo su pico Public (típicamente 2000 o 2100).
- El submit final usa corte 2000 porque es la meseta central de casi todos los modelos. Los picos aislados en cortes atípicos (1800, 2300) son sospechosos de suerte del Public.

## Por qué min_sum_hessian_in_leaf como HP a optimizar (y no min_data_in_leaf)

El profesor recomienda `min_data_in_leaf=0` fijo (distinto al default 20) y `min_sum_hessian_in_leaf` a optimizar. Rationale técnico: ambos parámetros controlan la mínima "masa" que puede tener una hoja del árbol, pero `min_sum_hessian_in_leaf` está mejor calibrado con el gradiente de la función de pérdida (que es lo que realmente importa), mientras que `min_data_in_leaf` es puramente sobre conteo de filas.

En la práctica: `min_sum_hessian_in_leaf = 24` (encontrado por 9103) es equivalente a decir "cada hoja debe tener al menos 24 unidades de segundo derivada de la log-loss agregada", lo cual regulariza de manera adaptativa.

## Por qué se descartó el CV temporal (9107) aunque tuviera mejor AUC validation

Este es el caso más interesante metodológicamente. El 9107 tiene:
- Mejor AUC validation (0.9355 vs 0.9337 del 9105)
- HP calibrados con validation multi-mes (más honesto)
- Public individuales peores que el 9105 (Wilcoxon p=0.844 en contra)

La contradicción se explica por lo que dice el Experimento 1 de las slides: **AUC en validation y Public en el mes futuro son casi independientes**. El CV temporal produce modelos que optimizan mejor validation, pero eso no se traduce necesariamente en mejor Public.

**Argumento a favor del 9107 en Private (no realizado)**: si el shake-up Public → Private es "otra tirada del dado", el modelo con HP más honestos podría ganar en Private aunque pierda en Public. **No hay manera de saberlo hasta que se destape.**

**Decisión pragmática**: el Wilcoxon con 5 pares es el mejor proxy disponible para Private. Si Public está claramente en contra del 9107, la apuesta racional es no usarlo como submit final.

## Por qué el submit final es el 9105 k=100 y no el ensemble multi-modelo

El ensemble A (9105 + 9106) tocó el máximo Public de toda la serie (89.06 en corte 2000), pero:

1. **Sd cortes 1.68** (vs 0.94 del 9105 k=100 solo). Más sensibilidad al corte.
2. **Correlaciones entre 9105 y 9106 = 0.993**. No hay diversidad estructural genuina.
3. **Elegir por pico Public es exactamente el error del Experimento 1**. Public y Private están descorrelacionados.

Las slides de la cátedra recomiendan **reducir varianza** (más semillas, menor sd), no maximizar pico Public. El 9105 k=100 hace exactamente eso: máximo semillerío disponible sobre un pipeline validado por Wilcoxon.

## Sobre la sospecha razonable del final ensemble k=100 (pico 89.72 en corte 1800)

El ensemble final de 100 predicciones (50 del 9105 + 50 del 9106) tocó su máximo Public en corte 1800, con **sd cortes = 2.00**. Un pico en el corte más chico con sd tan alta es indicador clásico de haber "encontrado" el corte que se alinea con el 30% Public específico. En Private ese pico probablemente no sobreviva.

Comparar con el 9105 k=100 solo: pico 88.72 en corte 2000, sd cortes 0.94. Meseta amplia, sin pico específico. Este es el modelo más "honesto" en el sentido riguroso.

## Lo que este trabajo NO hace y por qué

- **No hace undersampling**: `training_pct=1.0` en todos los experimentos. LightGBM maneja bien el desbalance vía GBDT. El costo/beneficio de undersampling estratificado se juzgó bajo.
- **No usa `boosting=dart`**: siguiendo la guía del profesor (`gbdt` únicamente para clasificación).
- **No optimiza min_data_in_leaf, lambda_l1, lambda_l2, bagging_fraction**: fijos según recomendaciones del profesor.
- **No hace stacking**: se probó ensemble multi-modelo (promedio) pero no meta-modelo (stacking). Con n=5 pares y correlaciones 0.99+ entre modelos base, un meta-modelo estaría sobreajustado por definición.
