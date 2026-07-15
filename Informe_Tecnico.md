# Informe técnico

## Predicción de abandono de clientes en ventas y marketing

- **Integrantes:** Sebastian Lagos y Oscar Oreste
- **Tipo de proyecto:** evaluación transversal de ciencia de datos
- **Fecha de validación técnica:** 15 de julio de 2026

## 1. Resumen ejecutivo

El proyecto desarrolla una solución reproducible para analizar y clasificar abandono de clientes. El flujo integra un dataset comercial de 15.000 clientes con indicadores nacionales obtenidos desde la API del Banco Mundial, ejecuta procesos de auditoría, limpieza, transformación e ingeniería de características, construye visualizaciones interactivas y compara cuatro referencias de clasificación.

La variable objetivo presenta desbalance: 15,32 % de los clientes corresponde a churn. La regresión logística fue seleccionada por estabilidad, con 84,80 % de accuracy de prueba y una brecha train-test de 0,042 puntos porcentuales. Sin embargo, su mejora frente al baseline es solo 0,133 puntos porcentuales y detecta 89 de 460 churn. Por tanto, el resultado es válido como ejercicio académico, pero no suficiente para un sistema operativo de retención.

## 2. Contexto y problema

Una organización de ventas y marketing dispone de información demográfica, comercial, digital y de experiencia de clientes. El problema consiste en analizar los factores asociados con el abandono y construir un modelo que estime la clase `churn`.

La unidad de análisis es cada cliente. La variable objetivo se interpreta como:

- `0`: cliente sin abandono.
- `1`: cliente con abandono.

La solución busca apoyar análisis de retención, no automatizar decisiones ni demostrar relaciones causales.

## 3. Objetivo general

Diseñar e implementar un flujo reproducible de ciencia de datos para analizar y clasificar el abandono de clientes, integrando gestión de datos, ETL, una fuente externa, modelos de machine learning, visualizaciones interactivas y documentación técnica.

## 4. Objetivos específicos

1. Auditar la calidad y estructura del dataset.
2. Aplicar reglas de limpieza trazables.
3. Crear variables derivadas relevantes.
4. Integrar indicadores externos por país y año.
5. Desarrollar análisis exploratorio con Plotly.
6. Evitar fuga de información mediante split previo y pipelines.
7. Comparar modelos contra un baseline explícito.
8. Interpretar resultados mediante accuracy y matrices de confusión.
9. Generar un dashboard HTML autónomo.
10. Documentar reproducibilidad, limitaciones y riesgos.

## 5. Descripción del dataset

El archivo crudo es `data/raw/sales_marketing_customer_dataset.csv`.

| Elemento | Valor |
|---|---:|
| Filas | 15.000 |
| Columnas | 30 |
| Clientes únicos | 15.000 |
| Duplicados exactos | 0 |
| Customer ID duplicados | 0 |
| No churn | 12.702 (84,68 %) |
| Churn | 2.298 (15,32 %) |
| Países | 5 |

Las variables cubren género, edad, país, ciudad, fechas, canal de adquisición, dispositivo, suscripción, navegación, marketing, gasto, soporte, entrega, satisfacción, valor de vida y abandono.

El dataset procesado contiene 15.000 filas y 40 columnas.

El dataset base corresponde a **Sales and Marketing DataSet**, publicado por **Bhasker Paul** en Kaggle. El nombre original del archivo es `Sales - Marketing customer dataset.csv`; dentro del repositorio fue renombrado como `data/raw/sales_marketing_customer_dataset.csv` para mantener una convención uniforme. La página de origen es <https://www.kaggle.com/datasets/bhaskerpaul/sales-and-marketing-dataset>.

La licencia debe comprobarse directamente en la ficha vigente de Kaggle antes de redistribuir el CSV fuera del contexto académico. El equipo también debe conservar evidencia de que el conjunto es distinto de los utilizados previamente en el curso.

## 6. Auditoría de calidad

La auditoría detectó:

- 0 duplicados exactos.
- 0 duplicados ignorando `customer_id`.
- 6.133 ausencias en `coupon_code` (40,89 %).
- 1.200 ausencias originales en `age` (8,00 %).
- 1.050 ausencias en `total_spent` (7,00 %).
- 738 ausencias en `gender` (4,92 %).
- 702 ausencias en `satisfaction_score` (4,68 %).
- 4 edades menores o iguales a cero.
- 3.762 compras anteriores a la fecha de registro (25,08 %).
- 11.818 combinaciones país-ciudad inconsistentes según la relación interna utilizada (78,79 %).
- 78 valores extremos de `total_spent` según IQR (0,52 %).
- 0 variables constantes.

Las inconsistencias de ubicación deben interpretarse con cautela: la referencia se obtiene de patrones internos del dataset y no de una fuente geográfica oficial.

## 7. Limpieza y transformaciones

Las reglas aplicadas fueron:

- conservación de las 15.000 filas al no existir duplicados exactos;
- reemplazo de `gender` y `coupon_code` faltantes por `Sin registro`;
- creación de `coupon_code_present_flag`;
- conversión de edades menores o iguales a cero en valores nulos;
- conversión de fechas a tipo `datetime`;
- creación de `date_inconsistency_flag`;
- cálculo de `tenure_days` solo cuando las fechas son consistentes;
- creación de `signup_year` y `last_purchase_year`;
- creación de `location_inconsistency_flag`;
- transformación `log1p` de `total_spent` en `total_spent_log`;
- conservación de nulos para posterior imputación dentro del pipeline.

El dataset intermedio resultante contiene 15.000 filas y 37 columnas.

## 8. Integración de la API

La fuente externa es la API v2 del Banco Mundial. Se consultan:

- usuarios de Internet como porcentaje de la población (`IT.NET.USER.ZS`);
- PIB per cápita en dólares corrientes (`NY.GDP.PCAP.CD`).

La consulta abarca Bangladesh, Germany, India, UK y USA para 2022–2024. El flujo implementa sesión HTTP, reintentos, backoff, timeout, validación JSON, caché local y fallback.

La conectividad fue comprobada directamente el **15 de julio de 2026** desde el entorno virtual del proyecto. Los endpoints de `IT.NET.USER.ZS` y `NY.GDP.PCAP.CD` respondieron con estado HTTP 200. Cada indicador entregó 15 observaciones válidas, correspondientes a cinco países y tres años, sin combinaciones duplicadas ni valores faltantes. La API informó `2026-07-13` como fecha de última actualización.

El flujo conserva `data/external/world_bank_indicators.csv` como caché validado para contingencias. Cuando la API no está disponible, el notebook utiliza el archivo local; si la API y el caché fallan, la ejecución se detiene en lugar de inventar datos.

La unión mantuvo 15.000 filas, 15.000 clientes únicos y 0 casos sin coincidencia país-año.

## 9. Ingeniería de características

Se incorporaron:

- `coupon_code_present_flag`;
- `date_inconsistency_flag`;
- `tenure_days`;
- `signup_year`;
- `last_purchase_year`;
- `location_inconsistency_flag`;
- `total_spent_log`;
- `country_code`;
- `internet_users_pct`;
- `gdp_per_capita_usd`.

Para modelado se excluyeron:

- `churn`, por ser la variable objetivo;
- `customer_id`, por ser identificador;
- `city`, por alta inconsistencia interna;
- fechas originales, reemplazadas por variables derivadas;
- `coupon_code`, reemplazado por su bandera de presencia;
- `country_code`, redundante con `country`;
- `total_spent`, reemplazado por `total_spent_log`;
- `lifetime_value`, por ambigüedad temporal y riesgo preventivo de fuga.

## 10. Análisis exploratorio

Los hallazgos descriptivos principales fueron:

- churn global de 15,32 %;
- correlación de `total_spent_log` con churn de -0,32;
- correlación de `satisfaction_score` con churn de -0,30;
- correlación de `support_tickets` con churn de 0,13;
- tasa de churn por canal entre 14,81 % y 15,88 %;
- satisfacción media de 3,73 para no churn y 2,82 para churn;
- tasa de churn de 50,43 % para clientes agrupados con 5 o más tickets;
- India presenta la tasa por país más alta observada: 15,93 %.

Estas relaciones son descriptivas. No prueban que una variable cause abandono.

## 11. Preparación para modelado

El conjunto de predictores contiene 31 columnas: 25 numéricas y 6 categóricas.

La partición se realizó mediante `train_test_split` con:

- 80 % entrenamiento: 12.000 clientes;
- 20 % prueba: 3.000 clientes;
- `random_state=42`;
- `stratify=y`.

| Conjunto | Clase 0 | Clase 1 | Total |
|---|---:|---:|---:|
| Entrenamiento | 10.162 | 1.838 | 12.000 |
| Prueba | 2.540 | 460 | 3.000 |

El preprocesamiento se implementó con `ColumnTransformer`:

- numéricas: imputación por mediana y estandarización;
- categóricas: imputación por moda y One-Hot Encoding;
- categorías desconocidas: `handle_unknown="ignore"`.

Todo el ajuste ocurre dentro del pipeline y exclusivamente sobre entrenamiento.

## 12. Modelos evaluados

Se evaluaron:

1. `DummyClassifier` de clase mayoritaria.
2. `LogisticRegression`.
3. `DecisionTreeClassifier`.
4. `RandomForestClassifier`.

No se realizó balanceo, optimización de hiperparámetros, ajuste de umbral ni entrenamiento de modelos adicionales.

## 13. Resultados comparativos

| Modelo | Accuracy train | Accuracy test | Brecha | Mejora vs. baseline | Churn detectados | Churn omitidos |
|---|---:|---:|---:|---:|---:|---:|
| Clase mayoritaria | 84,68 % | 84,67 % | 0,017 pp | 0,000 pp | 0 | 460 |
| Regresión logística | 84,84 % | 84,80 % | 0,042 pp | +0,133 pp | 89 | 371 |
| Árbol de decisión | 100,00 % | 86,17 % | 13,833 pp | +1,500 pp | 233 | 227 |
| Bosque aleatorio | 100,00 % | 84,87 % | 15,133 pp | +0,200 pp | 90 | 370 |

Accuracy es la métrica académica principal del proyecto. Debido al desbalance, se complementa con los conteos derivados de la matriz de confusión. No se introducen precision, recall, F1 o ROC-AUC porque no forman parte de la evaluación implementada.

## 14. Matrices de confusión

### Regresión logística

| Real / Predicho | No churn | Churn |
|---|---:|---:|
| No churn | 2.455 | 85 |
| Churn | 371 | 89 |

### Árbol de decisión

| Real / Predicho | No churn | Churn |
|---|---:|---:|
| No churn | 2.352 | 188 |
| Churn | 227 | 233 |

### Bosque aleatorio

| Real / Predicho | No churn | Churn |
|---|---:|---:|
| No churn | 2.456 | 84 |
| Churn | 370 | 90 |

## 15. Selección del modelo

La regla de selección exige superar el baseline y considera una brecha train-test de 5 puntos porcentuales o más como señal importante de inestabilidad.

La regresión logística fue seleccionada porque:

- supera el baseline, aunque marginalmente;
- presenta la brecha train-test más baja;
- no exhibe sobreajuste relevante bajo accuracy.

La selección no implica que el modelo sea suficientemente eficaz. Detecta solo 19,35 % de los churn y omite 80,65 %. El árbol reconoce más churn, pero su 100 % en entrenamiento y brecha de 13,833 puntos porcentuales evidencian sobreajuste.

## 16. Dashboard

El archivo `dashboard/dashboard_churn.html` integra:

- cuatro KPIs;
- churn por país;
- churn por canal de adquisición;
- accuracy y porcentaje de churn detectado por modelo;
- selector global y por cinco países.

Plotly se encuentra embebido mediante `include_plotlyjs=True` y `full_html=True`. El HTML puede abrirse sin ejecutar Python y no requiere un framework web ni servidor.

El selector modifica indicadores y vistas descriptivas por país. Las métricas de los modelos permanecen globales porque no se realizó una evaluación independiente por país.

## 17. Recomendaciones de negocio

Las siguientes recomendaciones deben entenderse como hipótesis operativas para evaluación, no como efectos causales demostrados:

- priorizar la revisión de clientes con múltiples tickets de soporte;
- incorporar señales de satisfacción y gasto en análisis de segmentación;
- investigar la calidad del proceso que genera fechas y ubicaciones inconsistentes;
- evitar decisiones automáticas basadas únicamente en la predicción actual;
- evaluar intervenciones controladas antes de atribuir impacto a una variable;
- en una etapa futura, estudiar métricas orientadas a clase minoritaria y costos de error.

## 18. Riesgos, sesgos y limitaciones

- Desbalance de clases.
- Accuracy cercana al baseline.
- Detección limitada de churn.
- Sobreajuste en árbol y bosque.
- Ausencia de evaluación temporal.
- Ambigüedad de `lifetime_value`.
- Datos externos agregados a nivel nacional.
- Posibles inconsistencias sintéticas o de generación en país-ciudad y fechas.
- Ausencia de balanceo y optimización.
- Dashboard estático.
- Posibles variaciones de resultados entre versiones de Scikit-learn.
- La licencia del dataset debe verificarse en la ficha vigente de Kaggle antes de redistribuir el CSV.
- Ausencia de evidencia Git dentro del ZIP, al no incluirse el directorio `.git`.

## 19. Conclusiones

El proyecto cumple el flujo técnico central solicitado: gestión de datos, ETL, integración de API, análisis exploratorio, pipelines de machine learning, comparación de modelos y dashboard interactivo. También mantiene una lectura crítica de resultados y evita presentar la accuracy como evidencia suficiente.

La principal conclusión predictiva es negativa: los modelos evaluados no ofrecen una mejora robusta y útil sobre el baseline para detectar churn. La regresión logística es estable, pero su sensibilidad práctica frente a la clase positiva es baja. El valor del trabajo está en la reproducibilidad, trazabilidad metodológica y comunicación crítica de limitaciones.

## 20. Reproducibilidad

La ejecución integral fue validada desde un kernel limpio mediante `nbconvert`.

- Duración medida: 37,28 segundos en la primera validación completa del paquete.
- Celdas de código ejecutadas: todas las celdas ejecutables.
- Errores almacenados: 0.
- Salidas `stderr`: 0.
- Modificación de `data/raw`: ninguna.
- Conectividad de la API: validada en vivo mediante HTTP 200 para ambos indicadores.
- Uso de caché externo: validado como mecanismo de contingencia.
- Dashboard: generado correctamente.

Las dependencias están declaradas en `requirements.txt`. Las instalaciones dentro del notebook fueron eliminadas para separar entorno y lógica analítica.

## 21. Estado frente al encargo

| Requisito | Estado | Evidencia |
|---|---|---|
| Caso realista de ciencia de datos | Cumplido | Problema de churn definido |
| Dataset seleccionado por el equipo | Cumplido con verificación documental pendiente | Fuente Kaggle identificada; falta comprobar la licencia vigente y conservar evidencia de novedad académica |
| Gestión y limpieza de datos | Cumplido | Auditoría y dataset intermedio |
| ETL | Cumplido | Raw → interim → external → processed |
| Integración de distintas fuentes | Cumplido | CSV + Banco Mundial |
| Pandas | Cumplido | Carga, limpieza, agrupación e integración |
| Scikit-learn | Cumplido | Pipelines y cuatro referencias de clasificación |
| API | Cumplido | Consulta en vivo validada, más caché y fallback |
| Visualizaciones interactivas | Cumplido | Plotly en notebook |
| Dashboard interactivo | Cumplido | HTML autónomo |
| Despliegue revisable | Cumplido con alcance estático | Archivo HTML sin servidor |
| Git colaborativo | No verificable desde el ZIP | Debe mostrarse historial real del repositorio |
| Docker y CI/CD | No implementado | El enunciado los condiciona a pertinencia; no son necesarios para el HTML estático |
| Documentación | Cumplido tras cierre | README, informe y estado |
| Evidencias para AVA | Parcial | Falta adjuntar historial Git, licencia vigente del dataset y evidencias de presentación |
| Presentación de 15 minutos | Pendiente académico | Requiere PPT, demo y ensayo individual |

## 22. Referencias

- Bhasker Paul. *Sales and Marketing DataSet*. Kaggle. <https://www.kaggle.com/datasets/bhaskerpaul/sales-and-marketing-dataset>
- World Bank. *About the Indicators API Documentation*. <https://datahelpdesk.worldbank.org/knowledgebase/articles/889392-about-the-indicators-api-documentation>
- World Bank. *Individuals using the Internet (% of population)*, código `IT.NET.USER.ZS`. <https://data.worldbank.org/indicator/IT.NET.USER.ZS>
- World Bank. *GDP per capita (current US$)*, código `NY.GDP.PCAP.CD`. <https://data.worldbank.org/indicator/NY.GDP.PCAP.CD>
- Scikit-learn Developers. Documentación de `Pipeline`, `ColumnTransformer`, clasificación y métricas.
- Plotly Technologies Inc. Documentación de Plotly para Python y exportación HTML.
- Pandas Development Team. Documentación de Pandas.