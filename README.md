# Predicción de abandono de clientes en ventas y marketing

Proyecto integrador de ciencia de datos orientado al análisis y clasificación de abandono de clientes (`churn`) a partir de información demográfica, comercial, de experiencia y comportamiento digital.

## 1. Descripción

El problema consiste en identificar patrones asociados al abandono de clientes y construir un clasificador reproducible que sirva como referencia académica para acciones de retención. La unidad de análisis es **un cliente por fila** y la variable objetivo es `churn`, donde `0` representa permanencia y `1` abandono.

La solución incluye auditoría de calidad, limpieza, ETL, integración con una API externa, ingeniería de características, análisis exploratorio interactivo, comparación de modelos y un dashboard Plotly autónomo. Los resultados no se presentan como un sistema listo para producción: el desempeño sobre la clase minoritaria es limitado y las asociaciones observadas no demuestran causalidad.

## 2. Integrantes

- Sebastian Lagos
- Oscar Oreste

## 3. Funcionalidades

- Auditoría de estructura, identificadores, duplicados, nulos, categorías, edades, fechas y valores extremos.
- Limpieza reproducible y generación de banderas de inconsistencia.
- Exportación de datasets intermedio y procesado.
- Integración con la API del Banco Mundial mediante reintentos, caché local y fallback.
- Ingeniería de características temporales, categóricas y monetarias.
- Visualizaciones interactivas con Plotly.
- Separación estratificada entrenamiento/prueba.
- Preprocesamiento con `Pipeline` y `ColumnTransformer`.
- Comparación de `DummyClassifier`, regresión logística, árbol de decisión y bosque aleatorio.
- Evaluación mediante accuracy y matrices de confusión.
- Dashboard HTML estático, interactivo y utilizable sin servidor.

## 4. Estructura del repositorio

```text
.
├── data/
│   ├── raw/
│   │   └── sales_marketing_customer_dataset.csv
│   ├── interim/
│   │   └── customers_clean.csv
│   ├── external/
│   │   └── world_bank_indicators.csv
│   └── processed/
│       └── customer_churn_processed.csv
├── dashboard/
│   └── dashboard_churn.html
├── notebook/
│   └── Sebastian_lagos_Oscar_Oreste_ET_801V.ipynb
├── reports/
│   └── informe_tecnico.md
├── .gitignore
├── PROJECT_STATE.md
├── README.md
└── requirements.txt
```

## 5. Dataset

El archivo original es `data/raw/sales_marketing_customer_dataset.csv`.

- Dimensiones originales: **15.000 filas × 30 columnas**.
- Unidad de análisis: cliente.
- Identificador: `customer_id`.
- Variable objetivo: `churn`.
- Clientes sin abandono: **12.702 (84,68 %)**.
- Clientes con abandono: **2.298 (15,32 %)**.
- Países representados: Bangladesh, Germany, India, UK y USA.

Los principales grupos de variables corresponden a demografía, ubicación, fechas, adquisición, dispositivo, suscripción, actividad digital, marketing, gasto, soporte, entrega, satisfacción y abandono.

`data/raw` se trata como fuente inmutable. El notebook genera los archivos derivados en `data/interim`, `data/external` y `data/processed`.

### Procedencia

El dataset base corresponde a **Sales and Marketing DataSet**, publicado por **Bhasker Paul** en Kaggle:

- Página de origen: <https://www.kaggle.com/datasets/bhaskerpaul/sales-and-marketing-dataset>
- Nombre original del archivo: `Sales - Marketing customer dataset.csv`.
- Nombre utilizado en el repositorio: `data/raw/sales_marketing_customer_dataset.csv`.

El cambio de nombre se realizó únicamente para mantener una convención uniforme en el repositorio. La fuente debe citarse al entregar o publicar el proyecto. La licencia aplicable debe comprobarse directamente en la ficha vigente de Kaggle antes de redistribuir el CSV fuera del contexto académico.

El equipo debe conservar además evidencia de que este dataset es distinto de los utilizados anteriormente en el curso.

## 6. Fuente externa

Se utiliza la API v2 del Banco Mundial para obtener, por país y año de registro:

- `IT.NET.USER.ZS`: personas que utilizan Internet (% de la población).
- `NY.GDP.PCAP.CD`: PIB per cápita en dólares corrientes.

La consulta cubre 5 países y los años 2022–2024, produciendo 15 combinaciones país-año. La integración conserva las 15.000 filas y no deja clientes sin coincidencia.

El flujo implementa:

- sesión HTTP con reintentos y backoff;
- timeout de conexión y lectura;
- validación de la estructura JSON;
- caché local en `data/external/world_bank_indicators.csv`;
- fallback automático cuando la API no está disponible;
- unión `many_to_one` por país y año.

La conectividad con la API fue validada directamente el **15 de julio de 2026** desde el entorno virtual del proyecto. Los dos indicadores respondieron con estado HTTP 200 y entregaron las 15 observaciones esperadas, sin combinaciones país-año duplicadas ni valores faltantes. La API informó `2026-07-13` como fecha de última actualización de ambos recursos.

El notebook conserva además un caché local validado. Si la consulta en vivo falla por red, timeout o respuesta inválida, se utiliza `data/external/world_bank_indicators.csv`; si tampoco existe un caché válido, la ejecución se detiene.

Los indicadores son nacionales y se repiten para clientes del mismo país-año. Aportan contexto agregado, no características individuales ni evidencia causal.

## 7. Metodología

1. Carga del dataset crudo.
2. Auditoría de estructura y calidad.
3. Limpieza reproducible sin eliminar observaciones válidas.
4. Creación de variables y banderas de inconsistencia.
5. Exportación del dataset intermedio.
6. Consulta o recuperación desde caché de indicadores externos.
7. Integración por país y año.
8. Exportación del dataset procesado.
9. Análisis exploratorio interactivo.
10. Exclusión justificada de variables con riesgo de fuga, redundancia o inconsistencia.
11. Split estratificado 80/20 con `RANDOM_STATE = 42`.
12. Imputación, escalamiento y One-Hot Encoding dentro del pipeline.
13. Baseline de clase mayoritaria.
14. Entrenamiento y comparación de modelos.
15. Evaluación en prueba y construcción del dashboard.

La imputación y las transformaciones se ajustan exclusivamente con `X_train`; `X_test` no participa en el ajuste del preprocesamiento.

## 8. Modelos evaluados

La validación final utilizó 12.000 clientes para entrenamiento y 3.000 para prueba.

| Modelo | Accuracy entrenamiento | Accuracy prueba | Brecha train-test | Mejora frente al baseline | Churn detectados | Churn omitidos |
|---|---:|---:|---:|---:|---:|---:|
| Clase mayoritaria | 84,68 % | 84,67 % | 0,017 pp | 0,000 pp | 0 | 460 |
| Regresión logística | 84,84 % | 84,80 % | 0,042 pp | +0,133 pp | 89 | 371 |
| Árbol de decisión | 100,00 % | 86,17 % | 13,833 pp | +1,500 pp | 233 | 227 |
| Bosque aleatorio | 100,00 % | 84,87 % | 15,133 pp | +0,200 pp | 90 | 370 |

### Matrices de confusión en prueba

Orden: filas = clase real; columnas = clase predicha (`0`, `1`).

- Regresión logística: `[[2455, 85], [371, 89]]`.
- Árbol de decisión: `[[2352, 188], [227, 233]]`.
- Bosque aleatorio: `[[2456, 84], [370, 90]]`.

### Criterio de selección

La regla implementada exige superar el baseline y penaliza brechas train-test iguales o superiores a cinco puntos porcentuales. Bajo este criterio se selecciona **regresión logística** por estabilidad, aunque su mejora sobre el baseline es marginal.

El árbol obtiene la mayor accuracy de prueba y detecta más churn, pero presenta sobreajuste importante. El bosque también alcanza 100 % en entrenamiento y una brecha elevada.

## 9. Resultados principales

- Tasa global de churn: **15,32 %**.
- Baseline de prueba: **84,67 %**.
- Modelo seleccionado: **regresión logística**.
- Accuracy de prueba del modelo seleccionado: **84,80 %**.
- Mejora frente al baseline: **0,133 puntos porcentuales**.
- Churn detectados: **89 de 460 (19,35 %)**.
- Churn omitidos: **371 de 460 (80,65 %)**.
- Mayor tasa observada por país: India, **15,93 %**.
- Mayor tasa observada por canal: Organic, **15,88 %**.
- `total_spent_log` y `satisfaction_score` muestran las asociaciones lineales negativas de mayor magnitud con churn.
- El grupo con cinco o más tickets de soporte presenta una tasa descriptiva de churn de **50,43 %**.

La accuracy elevada debe interpretarse junto al desbalance. El modelo seleccionado reproduce gran parte del rendimiento del baseline y detecta una fracción reducida de los abandonos.

## 10. Instalación

Se recomienda Python 3.13, que corresponde al entorno utilizado en la validación final.

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Instalación:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## 11. Ejecución

Ejecutar desde la raíz del repositorio:

```bash
python -m jupyter notebook
```

Abrir `notebook/Sebastian_lagos_Oscar_Oreste_ET_801V.ipynb` y ejecutar todas las celdas desde un kernel reiniciado.

Validación no interactiva:

```bash
python -m jupyter nbconvert \
  --to notebook \
  --execute notebook/Sebastian_lagos_Oscar_Oreste_ET_801V.ipynb \
  --output Sebastian_lagos_Oscar_Oreste_ET_801V_validado.ipynb \
  --ExecutePreprocessor.timeout=1200
```

El notebook temporal de validación no debe agregarse a Git.

## 12. Dashboard

Ruta:

```text
dashboard/dashboard_churn.html
```

El archivo contiene cuatro KPIs, tasa de churn por país, tasa por canal de adquisición y resultados globales de modelos. El selector incluye seis opciones: vista global y cinco países.

- Se abre mediante doble clic en un navegador moderno.
- Plotly está incorporado en el HTML.
- No requiere Python, un framework web ni servidor.
- No realiza consultas nuevas ni reentrena modelos.
- Puede publicarse como archivo estático en un servicio compatible.
- Para actualizarlo se debe volver a ejecutar el notebook.

El selector actualiza los KPIs y análisis descriptivos por país. Las métricas de modelos permanecen globales porque la evaluación no fue segmentada por país.

## 13. Reproducibilidad

- `RANDOM_STATE = 42`.
- Split estratificado 80/20.
- Un cliente por fila e IDs únicos.
- `Pipeline` y `ColumnTransformer`.
- Imputación y transformación ajustadas solo con entrenamiento.
- Caché local de la API.
- Datos derivados versionados.
- Dependencias explícitas en `requirements.txt`.
- Dashboard exportado con `include_plotlyjs=True` y `full_html=True`.
- `data/raw` no es modificado por la ejecución.

La ejecución limpia validada finalizó sin errores ni warnings almacenados. Se detectó que distintas versiones de Scikit-learn pueden producir pequeñas variaciones en árbol y bosque; por ello se fijaron las versiones del entorno validado.

## 14. Limitaciones

- No existe un corte temporal explícito para evaluar generalización futura.
- La clase churn está desbalanceada.
- Accuracy es insuficiente para valorar por sí sola la clase minoritaria.
- La regresión logística mejora muy poco respecto del baseline.
- La detección de churn del modelo seleccionado es limitada.
- Árbol y bosque presentan sobreajuste.
- `lifetime_value` se excluye preventivamente por ambigüedad temporal.
- Los indicadores externos son nacionales y se repiten por cliente.
- Las banderas de ubicación reflejan consistencia interna del dataset, no una verdad geográfica externa.
- No se aplicó balanceo, ajuste de umbral ni optimización de hiperparámetros.
- El dashboard es estático y no actualiza datos automáticamente.
- El análisis es observacional; no establece causalidad.
- La procedencia del dataset está documentada; la licencia debe comprobarse en la ficha vigente de Kaggle antes de redistribuir el CSV.
- El equipo debe conservar evidencia de que el dataset no fue utilizado previamente en el curso.

## 15. Conclusiones

El proyecto implementa un flujo completo y reproducible de ciencia de datos, desde la auditoría del dataset hasta la comunicación interactiva de resultados. La comparación de modelos evidencia que una accuracy alta puede ser engañosa bajo desbalance. La regresión logística es el candidato más estable, pero su capacidad de detectar abandono es insuficiente para recomendar uso operativo sin nuevas etapas de validación.

El valor académico principal reside en la trazabilidad del ETL, el control de fuga de información, la comparación contra baseline, la interpretación crítica de matrices de confusión y la generación de un dashboard autónomo.

