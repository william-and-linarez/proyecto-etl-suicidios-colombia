# Modelo estadístico y validación predictiva

## 1. Objetivo

Analizar la asociación entre los casos registrados de presuntos
suicidios, el sexo, el grupo de edad, el año y el departamento
de ocurrencia en Colombia durante 2015-2024.

El análisis es asociativo y no permite establecer relaciones causales.

## 2. Modelo de asociación

Se ajustó un modelo de regresión Poisson con la población como
exposición y la siguiente estructura:

`casos ~ sexo * grupo_edad + año + departamento`

En el modelo principal, año y departamento se incluyeron como
variables categóricas. La interacción entre sexo y edad permite
que la asociación entre sexo y tasa cambie según el grupo etario.

La dispersión de Pearson obtenida fue 1.09, compatible con una
sobredispersión leve en el ajuste evaluado.

Los resultados se expresan como razones de tasas (IRR), con
intervalos de confianza del 95 %. Las asociaciones se interpretan
respecto de las categorías de referencia del modelo.

## 3. Validación temporal

Se entrenó un modelo con los datos de 2015-2023 y se reservaron
las observaciones de 2024 para evaluación.

Para esta validación, el año se trató como variable numérica,
permitiendo extrapolar la tendencia temporal hacia 2024. La
población de 2024 se utilizó como exposición.

### Resultados por celda analítica

La unidad de evaluación corresponde a la combinación de
departamento, sexo y grupo de edad para el año 2024.

| Métrica | Resultado |
|---|---:|
| Observaciones de prueba | 1.122 |
| MAE | 1.079 |
| RMSE | 1.812 |
| Correlación observado-predicho | 0.943 |
| Casos observados | 3.013 |
| Casos predichos | 3.160,915 |

El MAE representa el error absoluto medio por celda. El RMSE
penaliza con mayor intensidad los errores grandes. La correlación
describe la asociación lineal entre los valores observados y
predichos, pero no es una medida suficiente por sí sola para
evaluar precisión.

### Resultados agregados por departamento

| Métrica | Resultado |
|---|---:|
| Departamentos evaluados | 33 |
| MAE departamental | 11.402 |
| RMSE departamental | 19.304 |
| Correlación departamental | 0.989 |
| Sesgo neto (predicho - observado) | 147,915 |
| Suma de errores absolutos | 376,259 |
| WAPE | 12,49 % |

El modelo sobreestimó el total de casos de 2024 en aproximadamente
148 casos, equivalente a cerca del 4,9 % de los casos observados.
El WAPE, en cambio, suma los errores absolutos por departamento
antes de compararlos con el total observado; por ello es mayor que
el error neto porcentual.

Se observaron errores territoriales que merecen atención. Por
ejemplo, se predijeron aproximadamente 444 casos para Bogotá,
frente a 363 observados. Para Nariño, se predijeron cerca de 98,
frente a 137 observados.

## 4. Interpretación y limitaciones

- La validación temporal se realizó con un único año reservado:
  2024. Se recomienda ampliar la validación con ventanas temporales
  adicionales cuando sea posible.
- La correlación elevada no garantiza precisión en cada departamento
  ni ausencia de sesgo.
- Las estimaciones de territorios con pocos casos pueden ser
  inestables; deben interpretarse con cautela.
- El resultado depende de la calidad de los registros del INMLCF
  y de las proyecciones poblacionales del DANE.
- Los resultados muestran asociaciones estadísticas, no causalidad.
- La validación usa la población proyectada de 2024 como exposición.
  Una aplicación futura requeriría disponer de la exposición
  poblacional correspondiente al periodo que se desea predecir.

## 5. Archivos de resultados

Los resultados de la validación se encuentran en `analysis/resultados/`:

- `predicciones_2024_por_celda.csv`
- `evaluacion_departamental_2024.csv`
- `metricas_validacion_2024.csv`
