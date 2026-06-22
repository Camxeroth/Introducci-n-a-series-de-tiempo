markdown_content = """# Análisis de Series Temporales de la Producción de Cerveza en Australia

### Modelado ARIMA/SARIMA con la Metodología de Box-Jenkins

**Curso:** Modelamiento — Actividad Autónoma 5  
**Unidad:** 3 — Análisis de Series Temporales  
**Autor:** Camilo Enrique Morocho Vinueza  
**Fecha:** 14 de junio de 2026  

---

## Descripción General

Este repositorio documenta el modelado empírico del conjunto de datos `monthly_beer_production.csv`, el cual registra la producción mensual de cerveza en Australia a lo largo de un período histórico significativo (1956–1995). El estudio va más allá del análisis descriptivo para aplicar el marco estadístico de Box-Jenkins en el pronóstico predictivo, con el objetivo de proyectar escenarios de producción capaces de orientar la toma de decisiones en la cadena de suministro y la logística.

---

## Conjunto de Datos

| Atributo | Detalle |
|---|---|
| Fuente | `monthly_beer_production.csv` |
| Período | Enero 1956 — Diciembre 1995 |
| Observaciones | 476 registros mensuales |
| Variable: `Month` | Cadena de caracteres (YYYY-MM) |
| Variable: `Monthly.beer.production` | Volumen de producción en megalitros |

---

## Metodología

El análisis sigue el **flujo de trabajo canónico de Box-Jenkins** para los modelos de la clase ARIMA:

```text
Datos Crudos → Transformación Estructural (objeto ts)
             → Partición Train/Test
             → Descomposición STL
             → Prueba de Estacionariedad (ADF)
             → Diferenciación (d=1, D=1)
             → Selección del Modelo (auto.arima / AICc)
             → Validación Diagnóstica (Ljung-Box)
             → Pronóstico Fuera de la Muestra (Out-of-Sample)
             → Evaluación de Métricas de Error
