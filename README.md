# Competencia Kaggle Junior — Data Mining ITBA 2026 A - Santino Cantisano

Predicción de bajas de Paquete Premium (BAJA+2) para el mes de noviembre 2021, sobre la foto de septiembre 2021 (`foto_mes = 202109`). Métrica: ganancia en millones de pesos argentinos, escalada por Kaggle sobre Public LB (30%) y Private LB (70%).

## Submit final

`experiments/9105_FE_ratios/kaggle/KA9105_FEratios_k100_2000.csv`

- Public Score: **88.72**
- Pipeline: 9105 (FE con ratios + HP validados del 9103) escalado a 100 semillas
- Corte: 2000 envíos
- Sd entre cortes: 0.94 (la menor de los ensembles no-triviales probados)

Justificación de la elección: alineamiento con las recomendaciones del profesor a lo largo de la materia. El submit maximiza reducción de varianza (semillerío k=100 = máximo posible con recursos disponibles) sobre un pipeline con HP validados por test de Wilcoxon pareado.

## Estructura del repo

```
.
├── README.md                    Este archivo. Vista general del proyecto.
├── RESULTS.md                   Public Scores + Wilcoxons de todos los experimentos.
├── docs/
│   └── decisiones.md            Justificación de cada decisión de diseño.
├── experiments/                 Un directorio por experimento (9101 → 9107).
│   ├── 9101_baseline/           Grid Search + semillerío k=5 → k=20.
│   ├── 9102_BO_agresiva/        Descartado: overfit (HP topando bounds altos).
│   ├── 9103_BO_conservadora/    Ganador vs 9101 (Wilcoxon direccional).
│   ├── 9104_FE_lag3/            Descartado: lag3+delta3 empeoró vs 9103.
│   ├── 9105_FE_ratios/          Ganador FE (ratios + indicadores). Base del submit final.
│   ├── 9106_BO_recalibrada/     BO ampliada sobre 9105. Empate estadístico.
│   └── 9107_CV_temporal/        Validation multi-mes. AUC más alta pero Public peor.
└── analysis/                    Notebooks de análisis (no producen submits).
    ├── importancia_9103.ipynb   Análisis de importancia de variables.
    ├── ensemble_multimodelo.ipynb  Ensemble 9105+9106+9107.
    └── ensemble_final_9105_9106_k100.ipynb  Ensemble 100 predicciones.
```

## Flujo de decisiones

El desarrollo siguió el principio de **"cambio de una cosa a la vez"** para atribución causal. Cada experimento nuevo cambió exactamente una dimensión respecto del anterior, y se validó con test de Wilcoxon pareado sobre las 5 semillas base:

```
[Grid Search fijo]
   9101 baseline
       ↓ (cambio: método de tuning)
   9102 BO agresiva  →  DESCARTADO (num_leaves topó 1024, min_sum_hessian topó 1e-8 = overfit)
       ↓ (cambio: bounds de la BO más regularizados)
   9103 BO conservadora  →  GANADOR vs 9101 (media +4.25, Wilcoxon direccional p=0.156)
       ↓ (cambio: agregar lag3+delta3 al FE)
   9104 FE lag3  →  DESCARTADO (Wilcoxon direccional p=0.906 en contra)
       ↓ (cambio: FE con ratios + indicadores selectivos, mismos HP del 9103)
   9105 FE ratios  →  GANADOR vs 9103 (media +2.19)
       ↓ (cambio: re-correr BO sobre dataset del 9105)
   9106 BO recalibrada  →  EMPATE estadístico con 9105
       ↓ (cambio: validation multi-mes [202105-202107])
   9107 CV temporal  →  AUC validation más alta pero Public peor
```

Escalado del semillerío como paso ortogonal, sin experimentar sobre 9105:

```
9105 k=5  →  k=20 (baseline honesto)  →  k=50  →  k=100 (submit final)
```

## Cómo reproducir el submit final

1. Correr `experiments/9105_FE_ratios/9105_workflow.ipynb` entero. Esto ejecuta:
   - Carga del CSV competencia_02.csv.gz
   - Catastrophe Analysis + Data Drifting (deflación por IPC)
   - Feature Engineering intra-mes + ratios/indicadores
   - Feature Engineering histórico (lag1, lag2, delta1, delta2)
   - Training strategy (train [201901-202105], validate 202107, future 202109)
   - HP hardcoded del 9103 (no re-corre BO)
   - Semillerío k=20 con las 20 semillas base

2. Correr `experiments/9105_FE_ratios/9105_escalar_k50.ipynb` para escalar a k=50 (agrega 30 semillas más).

3. Correr `experiments/9105_FE_ratios/9105_escalar_k100_FINAL.ipynb` para escalar a k=100 (agrega otras 50 semillas). Este notebook genera `KA9105_FEratios_k100_2000.csv` que es el submit final.

Tiempo total estimado: ~4-5 horas de compute en una VM con 8 vCPU y 64 GB RAM.

## Metodología estadística

- **Semillerío**: cada modelo ensemble se compone de N modelos LightGBM idénticos salvo por la semilla, promediando probabilidades. Reduce varianza siguiendo la fórmula `sd_ensemble ≈ sd_individual / √N`.
- **Test de Wilcoxon signed-rank pareado**: para comparar dos modelos M1 y M2, se toman las 5 mismas semillas en ambos y se comparan las ganancias Public individuales. Con `alternative="less"` se testea `M1 < M2`. p-value < 0.15 con n=5 es evidencia direccional aceptable dado el poder estadístico limitado.
- **Corte de referencia 2100**: la mayoría de los experimentos individuales se subieron a Kaggle con corte 2100 para pareo directo entre experimentos. El corte final del submit ganador (2000) se eligió por ser el centro de la meseta.

## Dependencias

- R 4.x
- Paquetes: `data.table`, `lightgbm`, `ParBayesianOptimization`, `yaml`
- Google Cloud VM: 8 vCPU, 64 GB RAM
- Kaggle CLI (para submits automáticos)

## Notas sobre reproducibilidad

- `PARAM$semilla_primigenia = 102191` está fija en todos los notebooks. Determina la semilla del undersampling (que no aplica con `training_pct=1.0`) y de la BO. Cambiarla altera los HP encontrados pero no las conclusiones cualitativas.
- Los HP hardcoded del 9103 en el 9104 y 9105 pueden reemplazarse por re-corrida completa de la BO conservadora sobre el mismo dataset. Los HP resultantes deberían ser similares (`num_leaves` en la zona 200-300, `learning_rate` en la zona 0.01-0.02).
- Los 5 archivos `KA_semilla_<X>.csv` individuales necesarios para los tests de Wilcoxon se generan con los notebooks `submit_individuales.ipynb` en el directorio de cada experimento.
