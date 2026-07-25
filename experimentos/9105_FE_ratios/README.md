# 9105 — FE ratios (base del submit final)

## Objetivo

Después del fracaso del 9104 (lag3), atacar el FE de manera selectiva usando el análisis de importancia del 9103. Los datos mostraron que el 71.7% del Gain del modelo venía de features BASE (no temporales), sugiriendo que **más features derivadas del valor puntual actual** era la palanca correcta.

## Cambio respecto al 9103

Se agregaron 12 features nuevas ANTES del FE histórico (así el pipeline calcula automáticamente sus lag1/lag2/delta1/delta2):

**Ratios (8)**:
- `r_visa_uso` = Visa_msaldototal / (Visa_mfinanciacion_limite + 1)
- `r_tarjeta_ingreso` = mtarjeta_visa_consumo / (mpayroll + 1)
- `r_caja_ahorro_pct` = mcaja_ahorro / (mcuentas_saldo + mcaja_ahorro + 1)
- `r_prestamos_ingreso` = mprestamos_personales / (mpayroll + 1)
- `r_margen_saldo` = (mpasivos_margen + mactivos_margen) / (mcuentas_saldo + 1)
- `r_trx_productos` = ctrx_quarter / (cproductos + 1)
- `r_trx_visa` = ctarjeta_visa_transacciones / (ctrx_quarter + 1)
- `r_homebanking_trx` = chomebanking_transacciones / (ctrx_quarter + 1)

**Indicadores binarios (4)**:
- `ind_visa_al_limite` = Visa_msaldototal > 0.9 * Visa_mfinanciacion_limite
- `ind_girado` = mcuenta_corriente < 0
- `ind_sin_payroll` = cpayroll_trx == 0
- `ind_con_prestamo` = cprestamos_personales > 0

HP hardcoded del 9103 (`num_leaves=256, learning_rate=0.0132, feature_fraction=0.4215, min_sum_hessian_in_leaf=23.997, num_iterations=347`) para aislar el efecto del FE.

## Resultado

- Wilcoxon vs 9103 (5 pares): p=0.156 direccional, 4 de 5 pares favorables al 9105, media +2.19
- Wilcoxon vs 9101 (5 pares): **p=0.0625 (mínimo posible con n=5), 5 de 5 pares favorables**, media +6.45
- Sd ensemble entre cortes: 0.93 (la más baja de todos los experimentos hasta este punto)

## Decisión: ACEPTADO como base del submit final

Los ratios captaron señal real. La sd ensemble muy baja indica un modelo más honesto sobre lo que sabe.

## Notebooks

1. `9105_workflow.ipynb` — pipeline completo con FE ratios + semillerío k=20
2. `9105_escalar_k50.ipynb` — escala a k=50 agregando 30 semillas nuevas
3. `9105_escalar_k100_FINAL.ipynb` — **GENERA EL SUBMIT FINAL** (k=100)

Ver [RESULTS.md § 9105](../../RESULTS.md#9105).
