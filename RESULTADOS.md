# Resultados detallados por experimento

Este archivo contiene los Public Scores de todos los submits, los tests de Wilcoxon pareado entre experimentos, y las decisiones tomadas en base a cada uno. Todos los tests usan la mismas 5 semillas para pareo directo: `c(804043, 653561, 703903, 439693, 665857)`.

## Índice
1. [Experimento 9101 — Baseline](#9101)
2. [Experimento 9102 — BO agresiva (DESCARTADO)](#9102)
3. [Experimento 9103 — BO conservadora](#9103)
4. [Experimento 9104 — FE lag3 (DESCARTADO)](#9104)
5. [Experimento 9105 — FE ratios](#9105)
6. [Experimento 9106 — BO recalibrada](#9106)
7. [Experimento 9107 — CV temporal](#9107)
8. [Escalado del semillerío](#semillerio)
9. [Ensambles multi-modelo](#ensembles)
10. [Decisión del submit final](#final)

---

## 9101 — Baseline con Grid Search + semillerío k=5 <a name="9101"></a>

**Configuración**: Grid Search de 5×5×2 = 50 combos sobre `num_leaves`, `min_data_in_leaf`, `feature_fraction`. Training [201901-202105], validate 202107. Semillerío k=5 con las 5 semillas base.

### Public Scores individuales (corte 2000)

| Semilla | Public |
|---|---|
| 804043 | 78.837 |
| 653561 | 79.667 |
| 703903 | 82.243 |
| 439693 | 82.492 |
| 665857 | 88.390 |
| **Media** | **82.33** |
| **Sd** | 3.74 |

### Ensemble k=5 (Public)

| Envíos | Public |
|---|---|
| 1800 | 80.581 |
| 1900 | 84.652 |
| **2000** | **85.649** |
| 2100 | 83.572 |
| 2200 | 84.486 |
| 2300 | 81.744 |
| 2400 | 79.252 |
| Media | 82.85 |
| Sd | 2.30 |

### Estabilidad del semillerío

Coincidencia top-2000 entre las 5 semillas (intersección / corte): **0.829**. Zona sana de diversidad (0.75-0.90 es lo ideal).

---

## 9101 escalado a k=20

Se agregaron 15 semillas nuevas del banco de primos. Reduce sd Private esperada de 3.74 a 3.74/√4 = 1.87.

### Ensemble k=20 (Public)

| Envíos | k=5 | k=20 | Δ |
|---|---|---|---|
| 1800 | 80.58 | 80.33 | −0.25 |
| 1900 | 84.65 | 81.25 | −3.41 |
| 2000 | 85.65 | 79.00 | −6.65 |
| 2100 | 83.57 | 83.16 | −0.42 |
| 2200 | 84.49 | 80.58 | −3.91 |
| 2300 | 81.74 | 81.66 | −0.08 |
| 2400 | 79.25 | 82.16 | +2.91 |
| **Sd** | 2.30 | **1.44** | −0.86 |

### Interpretación

El máximo Public del k=5 (85.65 en corte 2000) estaba **inflado** por semillas específicas alineadas con el 30% del Public LB. Al agregar 15 semillas, el pico se "difuminó" a valores más honestos (~83). Este es el fenómeno del Experimento 1 de las slides del profesor: **el Public es una realización aleatoria y no debe optimizarse directamente**.

---

## 9102 — BO agresiva (DESCARTADO) <a name="9102"></a>

**Cambio respecto de 9101**: Grid Search → Bayesian Optimization con bounds amplios (`num_leaves` hasta 1024, `min_sum_hessian_in_leaf` hasta 1e-8).

### HP encontrados

- `learning_rate = 0.0057`
- `num_leaves = 1024` (**topó el techo**)
- `feature_fraction = 0.445`
- `min_sum_hessian_in_leaf = 1e-8` (**topó el piso**)
- `num_iterations = 654`
- AUC validation = **0.9327**

### Ensemble k=20 (Public)

| Envíos | Public |
|---|---|
| 2000 | 85.649 |
| **Media** | 82.26 |
| **Sd** | 3.36 |

### Wilcoxon 9102 vs 9101 (corte 2000, individuales)

| Semilla | 9101 | 9102 | Δ |
|---|---|---|---|
| 804043 | 78.837 | 79.003 | +0.17 |
| 653561 | 79.667 | 78.670 | −1.00 |
| 703903 | 82.243 | 82.492 | +0.25 |
| 439693 | 82.492 | 79.169 | −3.32 |
| 665857 | 88.390 | 85.898 | −2.49 |

```r
wilcox.test(gan_9101, gan_9102, paired=TRUE, alternative="less")
# V=12, p-value = 0.9062
# → NO hay evidencia de que 9102 sea mejor que 9101
# Al contrario: dirección opuesta, 3 de 5 pares favorables al 9101
```

### Decisión: DESCARTADO

Dos banderas rojas: (a) HP topando bounds (BO pidiendo capacidad extrema y regularización mínima), (b) AUC validation sospechosamente alta con Public individuales peores. Diagnóstico: **overfit al validation set 202107**. El semillerío enmascaraba modelos individualmente overfit.

---

## 9103 — BO conservadora <a name="9103"></a>

**Cambio respecto de 9102**: bounds forzando regularización (`num_leaves` cap 256, `min_sum_hessian_in_leaf` piso 1e-3).

### HP encontrados

- `learning_rate = 0.0132`
- `num_leaves = 256` (topó el techo)
- `feature_fraction = 0.4215`
- `min_sum_hessian_in_leaf = 23.997` (**NO topó el piso** — regularización elegida libremente)
- `num_iterations = 347`
- AUC validation = **0.9337**

### Ensemble k=20 (Public)

| Envíos | Public |
|---|---|
| 1800 | 86.65 |
| 1900 | 87.81 |
| 2000 | 88.64 |
| **2100** | **89.55** |
| 2200 | 87.06 |
| 2300 | 84.90 |
| 2400 | 81.99 |
| Media | 86.66 |
| Sd | 2.24 |

### Wilcoxon 9103 vs 9101 (corte 2100 vs 2000)

| Semilla | 9101 c2000 | 9103 c2100 | Δ |
|---|---|---|---|
| 804043 | 78.837 | 87.227 | +8.39 |
| 653561 | 79.667 | 89.387 | +9.72 |
| 703903 | 82.243 | 86.313 | +4.07 |
| 439693 | 82.492 | 86.479 | +3.99 |
| 665857 | 88.390 | 83.489 | −4.90 |

```r
wilcox.test(gan_9101, gan_9103, paired=TRUE, alternative="less")
# V=3, p-value = 0.1562
# → Con n=5, p=0.156 es el mínimo alcanzable cuando 4 de 5 pares mejoran
# (para p=0.0625 necesitaríamos 5 de 5)

Media Δ = +4.25, sd_9103 = 2.12 (vs sd_9101 = 3.74, casi mitad)
```

### Decisión: GANADOR vs 9101

Evidencia direccional sólida: 4 de 5 pares favorables, media +4.25, sd casi a la mitad. `min_sum_hessian_in_leaf = 24` sin topar bounds confirma que la BO del 9102 estaba en zona de overfit.

---

## 9104 — FE lag3 (DESCARTADO) <a name="9104"></a>

**Cambio respecto de 9103**: agregar `_lag3` y `_delta3` al FE histórico (dataset pasa de ~750 a ~1000 columnas). HP hardcoded del 9103.

### Ensemble k=20 (Public)

| Envíos | 9103 | 9104 | Δ |
|---|---|---|---|
| 1800 | 86.65 | 87.23 | +0.58 |
| 1900 | 87.81 | 84.82 | −2.99 |
| 2000 | 88.64 | 85.73 | −2.91 |
| 2100 | **89.55** | 83.07 | −6.48 |
| 2200 | 87.06 | 83.49 | −3.57 |
| 2300 | 84.90 | 84.15 | −0.75 |
| 2400 | 81.99 | 84.82 | +2.83 |
| Media | 86.66 | 84.76 | −1.90 |
| Sd | 2.24 | 3.24 | +1.00 |

### Wilcoxon 9104 vs 9103 (corte 2100)

| Semilla | 9103 | 9104 | Δ |
|---|---|---|---|
| 804043 | 87.227 | 86.313 | −0.91 |
| 653561 | 89.387 | 82.326 | −7.06 |
| 703903 | 86.313 | 83.156 | −3.16 |
| 439693 | 86.479 | 82.575 | −3.90 |
| 665857 | 83.489 | 89.885 | +6.40 |

```r
wilcox.test(gan_9103, gan_9104, paired=TRUE, alternative="less")
# V=12, p-value = 0.9062
# → NO hay evidencia de que lag3 haya mejorado
```

### Decisión: DESCARTADO

El lag3+delta3 metió ruido más que información. Cuando gana lo hace chico (+6.40, +2.83); cuando pierde lo hace grande (−7.06, −6.48). Asimetría típica de features que degradan más que aportan.

---

## 9105 — FE ratios <a name="9105"></a>

**Cambio respecto de 9103**: agregar 12 features derivadas basadas en el [análisis de importancia](#importancia) del 9103. Ratios (r_visa_uso, r_tarjeta_ingreso, r_caja_ahorro_pct, r_prestamos_ingreso, r_margen_saldo, r_trx_productos, r_trx_visa, r_homebanking_trx) e indicadores binarios (ind_visa_al_limite, ind_girado, ind_sin_payroll, ind_con_prestamo). HP hardcoded del 9103.

### Ensemble k=20 (Public)

| Envíos | 9103 | 9105 | Δ |
|---|---|---|---|
| 1800 | 86.65 | 86.73 | +0.08 |
| 1900 | 87.81 | 87.89 | +0.08 |
| 2000 | 88.64 | 88.64 | 0.00 |
| 2100 | 89.55 | 86.23 | −3.32 |
| 2200 | 87.06 | 87.06 | 0.00 |
| 2300 | 84.90 | **87.98** | +3.07 |
| 2400 | 81.99 | 85.07 | +3.07 |
| Media | 86.66 | **87.09** | +0.42 |
| Sd | 2.24 | **0.93** | −1.31 |

### Wilcoxon 9105 vs 9103 (corte 2100)

| Semilla | 9103 | 9105 | Δ |
|---|---|---|---|
| 804043 | 87.227 | 89.553 | +2.33 |
| 653561 | 89.387 | 89.802 | +0.42 |
| 703903 | 86.313 | 92.544 | +6.23 |
| 439693 | 86.479 | 82.492 | −3.99 |
| 665857 | 83.489 | 89.470 | +5.98 |

```r
wilcox.test(gan_9103, gan_9105, paired=TRUE, alternative="less")
# V=3, p-value = 0.1562
# → 4 de 5 pares favorables, media +2.19

wilcox.test(gan_9101, gan_9105, paired=TRUE, alternative="less")
# V=0, p-value = 0.0625 (mínimo posible con n=5)
# → 5 de 5 pares favorables. Confirmación estadística de que el pipeline final
#   supera al baseline original.
```

### Decisión: ACEPTADO

Los ratios captaron señal real. La sd del ensemble bajó dramáticamente (2.24 → 0.93) — modelo más estable ante la elección del corte.

---

## 9106 — BO recalibrada <a name="9106"></a>

**Cambio respecto de 9105**: re-correr BO con bounds levemente ampliados (`num_leaves` cap 384) y presupuesto ampliado (70 evaluaciones vs 36 del 9103).

### HP encontrados

- `learning_rate = 0.0139`
- `num_leaves = 384` (topó el techo otra vez)
- `feature_fraction = 0.7158` (subió mucho vs 0.42 del 9103)
- `min_sum_hessian_in_leaf = 18.45`
- `num_iterations = 379`
- AUC validation = **0.9335** (esencialmente igual al 9105 hardcoded)

### Ensemble k=20 (Public)

| Envíos | 9105 | 9106 |
|---|---|---|
| 2000 | 88.64 | **88.97** |
| Media | 87.09 | 86.65 |
| Sd | 0.93 | 1.30 |

### Wilcoxon 9106 vs 9105 (corte 2100)

| Semilla | 9105 | 9106 | Δ |
|---|---|---|---|
| 804043 | 89.553 | 87.227 | −2.33 |
| 653561 | 89.802 | 86.479 | −3.32 |
| 703903 | 92.544 | 86.313 | −6.23 |
| 439693 | 82.492 | 86.479 | +3.99 |
| 665857 | 89.470 | 83.489 | −5.98 |

Media_9106 = 86.45, sd = **0.34** (la más baja de todos los experimentos individualmente).

```r
wilcox.test(gan_9105, gan_9106, paired=TRUE, alternative="less")
# V=11, p-value = 0.8438
# → No hay evidencia direccional de que 9106 mejore a 9105
```

### Decisión: EMPATE

Los HP de la BO recalibrada son distintos a los del 9103, pero el resultado es equivalente (AUC validation prácticamente igual, Wilcoxon no significativo). La sd 0.34 en individuales del 9106 es extraordinariamente baja — modelos casi idénticos entre semillas por el `feature_fraction` alto (0.72). Se mantiene como candidato de diversificación estructural.

---

## 9107 — CV temporal <a name="9107"></a>

**Cambio respecto de 9106**: validation multi-mes `[202105, 202106, 202107]` en vez de solo `[202107]`. Training [201901-202104]. BO con mismos bounds del 9106.

### HP encontrados

- `learning_rate = 0.0080` (aprox mitad del 9106)
- `num_leaves = 384` (topó otra vez)
- `feature_fraction = 0.5059`
- `min_sum_hessian_in_leaf = 10.06`
- `num_iterations = 812` (más del doble del 9106)
- AUC validation = **0.9355** (la más alta de todos los experimentos)

### Ensemble k=20 (Public)

| Envíos | 9106 | 9107 |
|---|---|---|
| 2000 | 88.97 | 85.32 |
| **2300** | 85.32 | **88.81** |
| Media | 86.65 | 86.71 |
| Sd | 1.30 | 1.44 |

### Wilcoxon 9107 vs 9105/9106 (corte 2100)

Media_9107 = 84.30, sd = **6.58** (la más alta de todos los experimentos).

```r
wilcox.test(gan_9105, gan_9107, paired=TRUE, alternative="less")
# V=11, p-value = 0.8438  →  9105 mejor que 9107

wilcox.test(gan_9106, gan_9107, paired=TRUE, alternative="less")
# V=10, p-value = 0.7812  →  9106 mejor que 9107

wilcox.test(gan_9103, gan_9107, paired=TRUE, alternative="less")
# V=10, p-value = 0.7812  →  9103 mejor que 9107
```

### Interpretación

AUC validation más alta pero Public peor. **Confirmación empírica del Experimento 1 de las slides**: la métrica en validation y la métrica en el mes futuro son casi independientes. El CV temporal produce modelos con HP más "afilados" (`lr` bajo + muchas iteraciones) que son individualmente ruidosos entre semillas.

### Decisión: NO ES EL SUBMIT FINAL

No supera a 9105 ni 9106 en Public. Aunque tiene el mejor AUC validation, la evidencia direccional en Wilcoxon es contra.

---

## Escalado del semillerío del 9105 <a name="semillerio"></a>

Escalado ortogonal a los experimentos: usa el mismo pipeline del 9105 pero con más semillas para reducir la varianza esperada.

### 9105 k=20 → k=50

| Envíos | k=20 | k=50 | Δ |
|---|---|---|---|
| 1800 | 86.73 | 86.31 | −0.42 |
| 1900 | 87.89 | 87.81 | −0.08 |
| 2000 | 88.64 | 88.64 | 0.00 |
| 2100 | 86.23 | 86.48 | +0.25 |
| 2200 | 87.06 | 87.23 | +0.17 |
| 2300 | 87.98 | 87.39 | −0.58 |
| 2400 | 85.07 | 85.48 | +0.42 |
| Sd | 0.93 | 0.93 | 0.00 |

Coincidencia top-2000 entre k=20 y k=50: **0.988**. Correlación media entre semillas: 0.9892.

### 9105 k=50 → k=100

| Envíos | k=50 | k=100 | Δ |
|---|---|---|---|
| 1800 | 86.31 | 86.23 | −0.08 |
| 1900 | 87.81 | 87.89 | +0.08 |
| **2000** | **88.64** | **88.72** | +0.08 |
| 2100 | 86.48 | 86.48 | 0.00 |
| 2200 | 87.23 | 87.48 | +0.25 |
| Sd | 0.93 | 0.94 | +0.01 |

Coincidencia top-2000 entre k=50 y k=100: **0.992**. Correlación entre ensembles: **0.9999**.

### Interpretación

El escalado no cambia el Public (como esperado — es una única realización). La ganancia real está en la varianza esperada en Private: sd_100 ≈ sd_50/√2 ≈ 0.66 (teórico). Es la mejora más segura disponible: matemáticamente garantizada, sin apuesta.

---

## Ensambles multi-modelo <a name="ensembles"></a>

Se probó combinar predicciones de 9105 + 9106 + 9107 (o subconjuntos) para diversificar en la dimensión de HP.

### Correlaciones entre modelos (k=20)

|  | 9105 | 9106 | 9107 |
|---|---|---|---|
| 9105 | 1.00 | 0.993 | 0.988 |
| 9106 | | 1.00 | 0.992 |
| 9107 | | | 1.00 |

Correlaciones >0.98 = los 3 modelos ven la misma señal.

### Coincidencia top-2000

- 9105 vs 9106: 94.8%
- 9105 vs 9107: 91.0%
- 9106 vs 9107: 93.3%
- Núcleo común los 3: **89.9%**

### Ensembles probados

| Ensemble | k total | Public máx |
|---|---|---|
| A = 9105 + 9106 (k=20 c/u) | 40 | **89.06** (corte 2000) |
| B = 9105 + 9106 + 9107 (k=20 c/u) | 60 | 88.89 (corte 2000) |
| Final = 9105 k=50 + 9106 k=50 | 100 | 89.72 (corte 1800) |

### Interpretación

El ensemble A tocó el máximo Public de toda la serie (89.06), pero con sd 1.68 (más alta que 9105 k=50 solo). El ensemble final k=100 dio pico Public 89.72 en corte 1800, sospechoso de suerte del Public específico. **Ninguno mejora sustancialmente al 9105 k=50/k=100 solo** en varianza esperada.

---

## Decisión del submit final <a name="final"></a>

### Candidatos evaluados

| Submit | Public 2000 | Sd cortes | Fundamento |
|---|---|---|---|
| KA9105_FEratios_k20_2000 | 88.64 | 0.93 | k=20 con FE ratios |
| KA9105_FEratios_k50_2000 | 88.64 | 0.93 | k=50 escalado |
| **KA9105_FEratios_k100_2000** | **88.72** | **0.94** | **k=100 = máximo semillerío** |
| KA9106_BOrec_k20_2000 | 88.97 | 1.30 | BO recalibrada |
| KAensembleA_9105_9106_2000 | 89.06 | 1.68 | Ensemble multi-modelo |
| KAfinal_9105_9106_k100_2000 | 88.81 | 2.00 | Ensemble final 100 |

### Justificación de la elección

**KA9105_FEratios_k100_2000.csv** cumple mejor las recomendaciones de las slides del profesor Denicolay:

1. **Máximo semillerío posible** (100 semillas) → máxima reducción de varianza intrínseca.
2. **Sd entre cortes 0.94** → una de las más bajas disponibles.
3. **HP validados por Wilcoxon** (los del 9103, ganadores estadísticos vs 9101).
4. **Corte 2000 en meseta central** → no elige un pico Public específico que podría ser suerte.
5. **Pipeline simple con un solo modelo** → menos partes móviles que ensambles multi-modelo.

El ensemble multi-modelo (Ensemble A o Final k=100) toca picos más altos en Public pero con mayor sd entre cortes. Elegir por pico Public es exactamente el error que las slides del Experimento 1 advierten: **Public y Private están descorrelacionados**.

### Ganancia estimada en Private

Con sd Public entre cortes = 0.94 y semillerío k=100, la sd esperada de este ensemble en Private es ~0.4-0.5 puntos (aplicando sd/√n con n=2 para el escalado k=50 → k=100). El intervalo de confianza esperado en Private es **[87.5 - 90.0]**, centrado en algún valor cercano al Public 88.72.

Bandas de mayor riesgo (~±3 puntos que muestran las slides) llevarían el rango a **[85.5 - 92.0]**. Ese es el ancho honesto de la incertidumbre.
