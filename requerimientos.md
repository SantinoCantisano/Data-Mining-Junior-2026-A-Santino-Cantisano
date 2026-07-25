# Requisitos técnicos

## Entorno de ejecución

- **Sistema operativo**: Linux (probado en Google Cloud VM con Ubuntu)
- **RAM**: 64 GB (mínimo recomendado; los datasets con FE ampliado ocupan ~15-20 GB)
- **CPU**: 8 vCPU (los tiempos estimados en los notebooks asumen esta configuración)
- **Almacenamiento**: al menos 20 GB libres para dataset + outputs intermedios

## Lenguaje

- R 4.x

## Paquetes de R

```r
install.packages(c(
  "data.table",             # Manipulación de datos
  "lightgbm",               # Modelo principal (>= 3.0)
  "ParBayesianOptimization",# BO en los experimentos 9102, 9103, 9106, 9107
  "yaml"                    # Lectura de PARAM.yml
))
```

## Kaggle CLI

Necesario para los submits automáticos desde los notebooks.

```bash
pip install kaggle
# Configurar credenciales en ~/.kaggle/kaggle.json
```

## Datos

El dataset `competencia_02.csv.gz` no se incluye en el repo (ver `.gitignore`). Se asume disponible en el path que use la cátedra durante la corrida. El path se define en `PARAM$input$dataset` al inicio de cada notebook.

## Reproducibilidad

- Todas las semillas están fijas y documentadas en los notebooks.
- `PARAM$semilla_primigenia = 102191` controla las aleatoriedades del pipeline (undersampling si aplica, semilla de la BO).
- Las 100 semillas del semillerío están hardcodeadas en los notebooks correspondientes.
- Los resultados exactos dependen de la versión de LightGBM (probado con 3.3.5). Diferencias marginales entre versiones son esperables pero las conclusiones cualitativas se mantienen.
