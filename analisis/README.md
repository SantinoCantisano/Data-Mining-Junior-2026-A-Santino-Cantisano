# Análisis auxiliares

Notebooks que no producen experimentos nuevos sino que analizan resultados o combinan predicciones existentes.

## importancia_9103.ipynb

Extrae la importancia de variables (Gain) de un modelo LightGBM entrenado con los HP del 9103. Agrupa por familia de variable y calcula qué fracción del Gain viene de features base vs temporales.

**Hallazgos clave**:
- Variables usadas: 375 de 380 (5 con Gain=0)
- Aporte por tipo: `base = 71.7%`, `lag1 = 10.8%`, `delta2 = 6.5%`, `lag2 = 5.9%`, `delta1 = 5.2%`
- Variable dominante: `ctrx_quarter` (12.97% del Gain total)

**Implicaciones estratégicas**:
- El 9104 (más lags temporales) tenía baja probabilidad a priori de aportar, porque el modelo ya usaba solo 28% de temporales.
- El 9105 (ratios/indicadores base) tenía alta probabilidad a priori de aportar, porque atacaba la dimensión con 71.7% de apetito.

## ensemble_multimodelo.ipynb

Combina predicciones de 9105 + 9106 (Ensemble A) y de 9105 + 9106 + 9107 (Ensemble B) sin re-entrenar, promediando probabilidades.

**Hallazgos**:
- Correlaciones entre los 3 modelos ~0.99 (los 3 ven la misma señal, ensemble multi-modelo aporta poco)
- Ensemble A pico Public 89.06 (corte 2000), sd cortes 1.68
- Ensemble B pico Public 88.89 (peor por incluir 9107 más ruidoso)

## ensemble_final_9105_9106_k100.ipynb

Combina 9105 k=50 + 9106 k=50 = 100 predicciones totales.

**Hallazgos**:
- Pico Public 89.72 en corte 1800 (sospechoso de suerte del Public)
- Sd cortes 2.00 (más alta que 9105 k=100 solo con 0.94)
- Correlación con 9105 k=50: 0.998

**Conclusión**: el ensemble multi-modelo no mejora sustancialmente sobre el 9105 k=100 solo. Se descarta como submit final por mayor sd entre cortes.
