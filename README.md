# Predicción de factores asociados a la tasa de presuntos suicidios en Colombia (2015-2024)

**Proyecto final - Curso de ETL, semestre 2026-2**  
**Línea:** Ciencia de Datos e Inteligencia Artificial

## 1. Problema y objetivo

Este proyecto integra registros de presuntos suicidios del Instituto Nacional de Medicina Legal y Ciencias Forenses (INMLCF) con proyecciones poblacionales del Departamento Administrativo Nacional de Estadística (DANE).

El objetivo es analizar las diferencias territoriales de las tasas de casos registrados y estudiar su asociación con el sexo, el grupo de edad, el año y el departamento de ocurrencia.

La solución desarrolla un pipeline ETL reproducible, implementa transformaciones y validaciones de calidad y genera un producto analítico almacenado en CSV y SQLite.

Los resultados representan asociaciones estadísticas. No permiten establecer relaciones causales ni estimar directamente el riesgo individual de una persona.

## 2. Fuentes de datos

### 2.1. INMLCF: presuntos suicidios en Colombia

**Archivo:** `Presuntos_Suicidios._Colombia,_2015-2024..csv`

Contiene registros de presuntos suicidios durante el periodo 2015-2024, con variables demográficas, temporales y territoriales.

Fuente: [Datos Abiertos de Colombia](https://www.datos.gov.co/)

### 2.2. DANE: proyecciones de población 2005-2017

**Archivo:** `proypoblacion2005-2017.xlsx`

Contiene información poblacional por departamento, sexo, edad y área geográfica.

### 2.3. DANE: proyecciones de población 2018-2050

**Archivo:** `proypoblacion2018-2050.xlsx`

Contiene proyecciones poblacionales por departamento, sexo, edad y área geográfica.

Fuente: [DANE - Proyecciones de población](https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion/proyecciones-de-poblacion)

Los tres archivos originales se conservan en `data/raw/`.

## 3. Arquitectura del pipeline ETL

El proceso se organiza en las etapas de extracción, transformación, integración y carga.

### 3.1. Extract

El módulo `extract/extract_data.py`:

- Verifica que existan los tres archivos originales.
- Lee los registros del INMLCF desde CSV.
- Lee las dos fuentes poblacionales del DANE desde Excel.

### 3.2. Transform

El módulo `transform/transform_data.py` implementa las siguientes operaciones:

- Estandarización de nombres de columnas.
- Exclusión de cuatro registros con código territorial 999, que no permiten asignación a una entidad territorial válida.
- Selección de los años 2015-2024 y del total poblacional por área.
- Conversión de las fuentes de población de formato ancho a formato largo.
- Armonización de los grupos de edad.
- Agregación de los casos por departamento, año, sexo y grupo de edad.
- Integración de casos y población mediante las claves territoriales y demográficas.
- Cálculo de la tasa por cada 100.000 habitantes.
- Creación de la variable `baja_frecuencia`, que identifica celdas con menos de cinco casos.
- Validación de dimensiones, claves, duplicados, valores y tasas.

El producto analítico utiliza población de cinco años o más. Las celdas de baja frecuencia se conservan y se identifican para facilitar su interpretación cautelosa.

### 3.3. Load

El módulo `load/load_data.py` almacena la tabla analítica en dos formatos:

- CSV para interoperabilidad y exploración.
- SQLite para almacenamiento estructurado y consultas SQL.

### 3.4. Orquestación

El archivo `pipeline.py` coordina la extracción, las transformaciones, las validaciones y la carga del producto final.

## 4. Estructura del repositorio

```text
PROYECTO_ETL_SUICIDIOS/
├── analysis/
│   ├── modelo.py
│   └── resultados/
│       ├── evaluacion_departamental_2024.csv
│       ├── metricas_validacion_2024.csv
│       └── predicciones_2024_por_celda.csv
├── data/
│   ├── raw/
│   │   ├── Presuntos_Suicidios._Colombia,_2015-2024..csv
│   │   ├── proypoblacion2005-2017.xlsx
│   │   └── proypoblacion2018-2050.xlsx
│   └── processed/
│       ├── suicidios_departamento_edad_sexo_2015_2024.csv
│       └── suicidios_departamento_edad_sexo_2015_2024.db
├── docs/
│   └── modelo_y_validacion.md
├── extract/
│   └── extract_data.py
├── load/
│   └── load_data.py
├── transform/
│   └── transform_data.py
├── pipeline.py
├── requirements.txt
├── .gitignore
└── README.md
```

## 5. Requisitos e instalación

El proyecto utiliza Python y las dependencias declaradas en `requirements.txt`.

Las principales dependencias son:

- `pandas`: manipulación, integración y transformación de datos.
- `openpyxl`: lectura de archivos Excel.
- `statsmodels`: estimación y evaluación del modelo estadístico.
- `sqlite3` y `pathlib`: almacenamiento en SQLite y gestión de rutas; forman parte de la biblioteca estándar de Python.

### Instalación

Desde la carpeta raíz del proyecto, ejecutar:

```bash
pip install -r requirements.txt
```

## 6. Ejecución

Para ejecutar el pipeline completo, ubicarse en la raíz del proyecto y ejecutar:

```bash
python pipeline.py
```

El pipeline lee los archivos de `data/raw/`, realiza las transformaciones y validaciones y genera los archivos finales en `data/processed/`.

Se recomienda ejecutar el proceso desde la raíz del repositorio, donde se encuentra `pipeline.py`.

## 7. Producto de datos y almacenamiento

El producto analítico final contiene **11.220 filas**, correspondientes a las combinaciones de:

- 33 entidades territoriales: 32 departamentos y Bogotá D. C.
- 10 años: 2015-2024.
- 2 categorías de sexo.
- 17 grupos de edad, desde 5-9 años hasta 80 años y más.

Las filas representan combinaciones territoriales y demográficas, incluyendo combinaciones sin casos registrados.

### 7.1. Archivos generados

**CSV:**

`data/processed/suicidios_departamento_edad_sexo_2015_2024.csv`

**Base de datos SQLite:**

`data/processed/suicidios_departamento_edad_sexo_2015_2024.db`

La tabla dentro de SQLite se denomina `suicidios_analitico`.

### 7.2. Diccionario de datos

| Campo | Descripción |
|---|---|
| `DP` | Código territorial del DANE |
| `AÑO` | Año de observación |
| `serie_poblacion` | Serie poblacional de origen |
| `sexo` | Sexo registrado |
| `grupo` | Grupo de edad armonizado |
| `poblacion` | Población del grupo demográfico |
| `casos` | Número de casos registrados |
| `tasa_x100k` | Casos por cada 100.000 habitantes |
| `baja_frecuencia` | Indicador: 1 si hay menos de cinco casos; 0 en caso contrario |
| `departamento` | Nombre de la entidad territorial |

La tasa se calcula mediante:

`tasa_x100k = casos / poblacion * 100000`

Las celdas con baja frecuencia se conservan y se identifican. No se eliminan automáticamente por tener pocos casos.

## 8. Modelo estadístico

El módulo `analysis/modelo.py` implementa funciones para cargar y preparar los datos, ajustar un modelo Poisson, evaluar su dispersión y realizar validación temporal.

La estructura del modelo de asociación es:

`casos ~ sexo * grupo_edad + año + departamento`

La población se utiliza como exposición. El modelo principal incorpora sexo, grupo de edad, año y departamento como variables categóricas. La interacción entre sexo y grupo de edad permite que la asociación entre sexo y tasa cambie según la edad.

Las razones de tasas (IRR) se interpretan respecto de las categorías de referencia del modelo. Estas estimaciones describen asociaciones y no efectos causales.

La dispersión de Pearson calculada para el modelo Poisson principal fue de 1,09, lo que indica una sobredispersión leve en el ajuste evaluado.

## 9. Validación predictiva

Se realizó una validación temporal entrenando el modelo con los datos de 2015-2023 y reservando 2024 para la evaluación.

Para esta validación, el año se trató como variable numérica con el fin de extrapolar la tendencia temporal hacia el periodo reservado. La población de 2024 se utilizó como exposición.

### 9.1. Resultados por celda analítica

| Métrica | Resultado |
|---|---:|
| Observaciones de prueba | 1.122 |
| MAE | 1,079 |
| RMSE | 1,812 |
| Correlación observado-predicho | 0,943 |
| Casos observados en 2024 | 3.013 |
| Casos predichos en 2024 | 3.160,915 |

La unidad de evaluación corresponde a cada combinación de departamento, sexo y grupo de edad de 2024.

El MAE resume el error absoluto promedio por celda. El RMSE penaliza más los errores grandes. La correlación mide la asociación lineal entre los valores observados y predichos; no demuestra por sí sola una alta precisión.

### 9.2. Resultados agregados por departamento

| Métrica | Resultado |
|---|---:|
| Departamentos evaluados | 33 |
| MAE departamental | 11,402 |
| RMSE departamental | 19,304 |
| Correlación departamental | 0,989 |
| Sesgo neto (predicho menos observado) | +147,915 casos |
| Suma de errores absolutos | 376,259 |
| WAPE | 12,49 % |

El modelo sobreestimó el total de 2024 en aproximadamente 148 casos, equivalente a cerca del 4,9 % de los casos observados.

La suma de errores absolutos es mayor que el error neto porque los errores de sobreestimación y subestimación se compensan al calcular el sesgo neto.

Aunque la correlación territorial fue alta, se presentaron errores relevantes en algunos departamentos. Por ejemplo, Bogotá tuvo aproximadamente 444 casos predichos frente a 363 observados, mientras que Nariño tuvo aproximadamente 98 predichos frente a 137 observados.

### 9.3. Archivos de resultados

Los resultados detallados están disponibles en `analysis/resultados/`:

- `predicciones_2024_por_celda.csv`
- `evaluacion_departamental_2024.csv`
- `metricas_validacion_2024.csv`

## 10. Calidad de los datos y limitaciones

- Cuatro registros con código territorial 999 fueron excluidos del producto territorial.
- El producto analítico utiliza población de cinco años o más.
- Algunas combinaciones de departamento, sexo y grupo de edad presentan baja frecuencia, lo que puede producir tasas inestables.
- Los registros administrativos y las proyecciones poblacionales tienen limitaciones propias de sus fuentes.
- Las diferencias territoriales observadas no demuestran causalidad.
- La validación predictiva se realizó con un único año reservado: 2024. Es recomendable evaluar ventanas temporales adicionales.
- La evaluación utiliza la población de 2024 como exposición; una aplicación futura requiere la población correspondiente al periodo que se desee predecir.
- Las métricas calculadas dentro de la muestra de entrenamiento no deben confundirse con el desempeño sobre datos futuros.

## 11. Reproducibilidad y trazabilidad

Los archivos originales se conservan en `data/raw/`.

Las transformaciones e integraciones están implementadas en `transform/transform_data.py`; la extracción, en `extract/extract_data.py`; la carga, en `load/load_data.py`; y la orquestación, en `pipeline.py`.

Las funciones del modelo y la validación están en `analysis/modelo.py`. La documentación adicional se encuentra en `docs/modelo_y_validacion.md`.

Esta estructura permite seguir el proceso desde las fuentes originales hasta el producto analítico y los resultados del modelo.

## 12. Referencias

- Instituto Nacional de Medicina Legal y Ciencias Forenses (INMLCF). Registros de presuntos suicidios en Colombia, 2015-2024. Datos Abiertos Colombia.
- Departamento Administrativo Nacional de Estadística (DANE). Proyecciones de población por departamento, sexo y edad. [Portal oficial](https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion/proyecciones-de-poblacion).
