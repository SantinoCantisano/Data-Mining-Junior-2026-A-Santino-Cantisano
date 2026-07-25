# 9104 — FE lag3 (DESCARTADO)

## Objetivo

Testar si extender el FE histórico con `_lag3` y `_delta3` aporta señal adicional. Es la extensión natural del FE base del 9101/9103 (que tenía lag1, lag2, delta1, delta2).

## Cambio respecto al 9103

- FE histórico ampliado: agrega `_lag3` y `_delta3` para todas las columnas lagueables
- Dataset pasa de ~750 a ~1000 columnas (+250 features)
- HP hardcoded del 9103 (no re-corre BO — para aislar el efecto del FE)

## Resultado

- Wilcoxon vs 9103: p-value = 0.906 en la dirección OPUESTA
- Media Δ = −1.73 (9104 peor)
- Sd individual sube de 2.12 a 3.24

## Decisión: DESCARTADO

El lag3+delta3 metió más ruido que información. Cuando ganó, ganó chico. Cuando perdió, perdió grande (asimetría clara). Se descarta y se continúa buscando FE más selectivo.

## Lección

**Más features no es mejor por defecto**. Agregar 250 columnas de una vez sin selección informada aumenta la varianza más que la señal. La lección informó el diseño del 9105 (selección basada en análisis de importancia).

Ver [RESULTS.md § 9104](../../RESULTS.md#9104).
