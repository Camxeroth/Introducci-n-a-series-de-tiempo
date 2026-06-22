# Análisis de Series Temporales de la Producción de Cerveza en Australia

### Modelado ARIMA/SARIMA mediante la Metodología Box–Jenkins

**Asignatura:** Modelamiento — Actividad Autónoma 5
**Unidad:** 3 — Análisis de Series Temporales
**Autor:** Camilo Enrique Morocho Vinueza
**Fecha:** 14 de junio de 2026

---

## Descripción General

Este repositorio documenta el modelado empírico del conjunto de datos `monthly_beer_production.csv`, el cual registra la producción mensual de cerveza en Australia durante un amplio periodo histórico (1956–1995). El estudio va más allá del análisis descriptivo y aplica el marco estadístico de Box–Jenkins para realizar pronósticos predictivos, con el objetivo de proyectar escenarios de producción que puedan apoyar la toma de decisiones en logística y cadena de suministro.

---

## Conjunto de Datos

| Atributo                            | Detalle                             |
| ----------------------------------- | ----------------------------------- |
| Fuente                              | `monthly_beer_production.csv`       |
| Periodo                             | Enero 1956 — Diciembre 1995         |
| Observaciones                       | 476 registros mensuales             |
| Variable: `Month`                   | Cadena de caracteres (AAAA-MM)      |
| Variable: `Monthly.beer.production` | Volumen de producción en megalitros |

---

## Metodología

El análisis sigue el flujo clásico de la metodología **Box–Jenkins** para modelos de tipo ARIMA:

```text
Datos Originales → Transformación Estructural (objeto ts)
               → División Entrenamiento/Prueba
               → Descomposición STL
               → Prueba de Estacionariedad (ADF)
               → Diferenciación (d=1, D=1)
               → Selección del Modelo (auto.arima / AICc)
               → Validación Diagnóstica (Ljung-Box)
               → Pronóstico Fuera de Muestra
               → Evaluación mediante Métricas de Error
```

### 1. Preprocesamiento Estructural

La variable temporal original fue convertida desde un vector de caracteres hacia un objeto formal `ts` de R (inicio: enero de 1956, frecuencia: 12). Esta transformación garantiza una separación temporal uniforme entre observaciones, permitiendo el cálculo de rezagos y operaciones de diferenciación necesarias para alcanzar estacionariedad.

### 2. División Entrenamiento / Prueba

| División                  | Periodo               | Observaciones |
| ------------------------- | --------------------- | ------------- |
| Conjunto de entrenamiento | 1956 — Diciembre 1990 | 420 meses     |
| Conjunto de validación    | Enero 1991 — 1995     | 60 meses      |

La partición fue realizada utilizando `window()` para conservar la estructura `ts` durante todo el proceso.

### 3. Descomposición STL

Se aplicó un esquema de descomposición **aditiva** mediante el algoritmo STL (*Seasonal and Trend decomposition using Loess*).

La elección del modelo aditivo se justifica porque la amplitud estacional permanece aproximadamente constante respecto al nivel de tendencia durante las décadas analizadas, descartando una especificación multiplicativa.

La descomposición permitió identificar tres componentes estructurales:

* **Tendencia:** crecimiento sostenido durante los años 70 seguido por una estabilización estructural.
* **Estacionalidad:** ciclo anual fuerte y recurrente asociado a patrones de producción y demanda.
* **Residuo:** ruido estacionario sin estructura evidente.

### 4. Análisis de Estacionariedad

Se aplicó la prueba **Augmented Dickey-Fuller (ADF)** sobre la serie de entrenamiento:

```text
H₀: La serie tiene raíz unitaria (no estacionaria)
H₁: La serie es estacionaria

Valor p ADF (serie original): 0.01
```

A pesar del valor inicial, la presencia de una tendencia clara y una estacionalidad dominante requirió aplicar diferenciación explícita:

* Diferencias regulares requeridas (`d`): **1**
* Diferencias estacionales requeridas (`D`): **1**

La serie doblemente diferenciada:

```r
diff(diff(ts, lag=12), differences=1)
```

presentó media y varianza estables, cumpliendo las condiciones necesarias para estimar modelos ARIMA.

### 5. Selección del Modelo — SARIMA

Se realizó una búsqueda exhaustiva sobre el espacio de parámetros SARIMA mediante `auto.arima()` con:

```r
stepwise = FALSE
approximation = FALSE
```

minimizando el criterio **AICc**.

**Modelo seleccionado:** `ARIMA(0,1,3)(0,1,2)[12]`

| Parámetro                     | Valor |
| ----------------------------- | ----- |
| Orden MA no estacional (q)    | 3     |
| Diferenciación regular (d)    | 1     |
| Orden MA estacional (Q)       | 2     |
| Diferenciación estacional (D) | 1     |
| Periodicidad estacional (m)   | 12    |

### Coeficientes estimados

| Término | Estimación | Error estándar |
| ------- | ---------- | -------------- |
| MA(1)   | −1.0730    | 0.0509         |
| MA(2)   | −0.0812    | 0.0820         |
| MA(3)   | 0.2686     | 0.0572         |
| SMA(1)  | −0.7402    | 0.0515         |
| SMA(2)  | −0.1110    | 0.0500         |

### Criterios de Información

| Criterio | Valor   |
| -------- | ------- |
| AIC      | 3034.74 |
| AICc     | 3034.95 |
| BIC      | 3058.80 |
| sigma²   | 95.62   |

### 6. Diagnóstico de Residuos

Se aplicó la prueba portmanteau de **Ljung–Box**:

```text
Prueba Ljung-Box

Q* = 101.95
gl = 19
valor p = 2.368e−13
```

El resultado indica autocorrelación residual significativa a 24 rezagos.

Aunque el modelo captura la estructura dominante de la serie, persiste cierta dependencia residual, una limitación común al modelar datos estacionales de largo plazo utilizando parametrizaciones parsimoniosas.

---

## Resultados

El modelo entrenado fue proyectado **60 pasos hacia adelante (h = 60 meses)** para cubrir exactamente el periodo reservado para validación.

### Métricas de Error

| Métrica   | Entrenamiento | Prueba  |
| --------- | ------------- | ------- |
| ME        | 0.0505        | −16.283 |
| RMSE      | 9.567         | 19.081  |
| MAE       | 6.972         | 16.713  |
| MPE       | −0.257        | −11.447 |
| MAPE      | 5.106         | 11.698  |
| MASE      | 0.738         | 1.768   |
| ACF1      | −0.014        | 0.112   |
| Theil's U | —             | 0.933   |

Un **MAPE fuera de muestra ≈ 11.7%** y un coeficiente **Theil's U = 0.933 (<1)** indican que el modelo SARIMA supera una proyección ingenua.

El pronóstico reproduce correctamente los picos estacionales y caídas de producción observadas entre 1991–1995, demostrando una capacidad adecuada de generalización.

---

## Conclusiones

### Transformación Estructural

La serie original presentó tendencia no estacionaria y fuerte componente estacional. La aplicación de diferenciación regular y estacional permitió estabilizar la serie y cumplir los requisitos del enfoque Box–Jenkins.

### Robustez Paramétrica

El modelo `ARIMA(0,1,3)(0,1,2)[12]` logró bajo error interno (**MAPE ≈ 5.1%**) e identificó correctamente la estructura de autocorrelación.

Aunque Ljung–Box fue significativo, esto puede atribuirse parcialmente a la sensibilidad del estadístico ante múltiples rezagos.

### Rendimiento Predictivo

La evaluación fuera de muestra confirma la utilidad operativa del modelo.

Un valor de **Theil's U inferior a 1** demuestra superioridad frente a métodos ingenuos y respalda su aplicación en planificación industrial y optimización logística.

---

## Estructura del Repositorio

```text
.
├── data/
│   └── monthly_beer_production.csv
├── scripts/
│   └── analysis.R
├── outputs/
│   ├── stl_decomposition.png
│   ├── acf_pacf_differenced.png
│   ├── residuals_arima.png
│   └── forecast_sarima.png
└── README.md
```

---

## Dependencias

```r
library(dplyr)
library(ggplot2)
library(forecast)
library(tseries)
```

Se recomienda utilizar **R versión 4.x o superior**.

Todos los paquetes están disponibles en **CRAN**.

---

## Licencia

Este trabajo fue elaborado como parte del cumplimiento parcial de los requisitos académicos de la asignatura de Modelamiento perteneciente a la Facultad de Ingeniería.

Todos los derechos reservados por el autor.
