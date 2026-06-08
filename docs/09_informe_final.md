<!--
ESQUELETO DEL INFORME FINAL — PI Grupo 8 (2026-1)
Estructura tomada literalmente de docs/material_entrega/formatos_y_reglas_entrega_pi.docx.
Cada sección anota: [REQUISITO] qué rúbrica satisface · [FUENTE] de dónde sale el contenido · [ESTADO].
Borrar estos comentarios HTML antes de exportar a DOCX/PDF.

Reglas de entrega (recordatorio): documento consolidado DOCX/PDF + PPTX/PDF + .txt con el repo,
montados en el canal del TEAM → carpeta "Entrega Final". Exposición ≤ 20 min, todos intervienen.
-->

# Optimización de la conversión en e-commerce mediante modelado de propensión de compra
### Proyecto Integrador 1 · Maestría en Ciencia de Datos y Analítica · EAFIT · 2026-1

---

## Portada
<!-- [REQUISITO] Formato oficial. [FUENTE] doc 00 encabezado. [ESTADO] redactar -->
- **Título:** Optimización de la conversión en e-commerce mediante modelado de propensión de compra.
- **Autores:** Kelly … , Sara … , Heider Zapata, Yeison …  *(completar nombres y afiliaciones)*
- **Materias:** SI7009 Aprendizaje Automático · SI7006 Almac. y Proces. de Grandes Datos · SI7007 Visualización de Datos.
- **Fecha de entrega:** 8 de junio de 2026 · **Exposición:** 9 de junio de 2026.
- **Repositorio:** `HeiderZapata/proyecto-integrador-g8`.

---

## 1. Introducción
<!-- [REQUISITO] Estructura oficial. [FUENTE] doc 00 §1, §5. [ESTADO] borrador -->

El comercio electrónico convierte una fracción muy pequeña de su tráfico: en la tienda analizada, **~98 de cada 100 visitas no terminan en compra y ~4 de cada 10 carritos se abandonan**. En un negocio de este volumen, cada punto de conversión recuperado tiene un impacto directo en el ingreso; por eso entender *dónde* y *por qué* se fuga la conversión —y *sobre quién* conviene intervenir— es una pregunta de negocio de primer orden.

El proyecto trabaja sobre **REES46**, un dataset abierto de *clickstream* multi-categoría de una tienda de e-commerce (Octubre–Noviembre 2019, ~14.5 GB, **109.5 M de eventos** `view`/`cart`/`purchase`). Su escala y su naturaleza de flujo de eventos lo hacen un caso realista tanto para la **ingeniería de grandes datos** como para el **modelado de propensión** sobre un **evento raro** (la compra, ~2–6 % según la unidad de medida).

**Pregunta de negocio (alcance completo):** ¿dónde y por qué se nos escapan las compras, qué tipos de visitante las explican, y dónde —y cómo lo mediríamos— conviene concentrar el esfuerzo para recuperar conversión? De ella se afila la **Pregunta de Oro** que guía el tablero: *¿dónde se concentra la fuga de conversión y qué segmento de visitantes representa la mayor oportunidad de recuperarla?*

El proyecto **no** construye un modelo para "adivinar quién compra"; construye un **sistema de decisión** con cuatro piezas: (1) un **clasificador de propensión** de compra a nivel sesión; (2) un **clustering de visitantes** para tipificar el comportamiento; (3) un **diagnóstico** del funnel y de las variables que explican la (no) conversión; y (4) el **diseño de un A/B test** que mediría el efecto de un incentivo sobre el segmento de mayor intención. La predicción es el **instrumento**; la **decisión de negocio** es el producto. *(El giro respecto a la propuesta original —que prometía un modelo de uplift causal, no estimable con datos observacionales— se fundamenta en §2.5 y se cierra en §3.5.)*

**Estructura del documento.** §2 establece el marco teórico; §3 desarrolla la metodología de ML (problema, EDA, características, modelos, entrenamiento, evaluación y el diseño del A/B); §4 describe la ingeniería de datos y la arquitectura tecnológica; §5 la visualización y comunicación; §6 las conclusiones generales; §7 las referencias.

## 2. Marco teórico y referencias
<!-- [REQUISITO] Estructura oficial. [FUENTE] docs/05 (ML), doc 02 (Big Data), docs/07 (Viz), doc 00 §5. [ESTADO] borrador -->

El hilo conductor metodológico no es "qué algoritmo", sino **cómo defender una decisión de modelado**: problema → baseline → métrica → validación → umbral → decisión. Un score alto no prueba nada por sí solo.

### 2.1 Clasificación supervisada sobre eventos raros
Cuando la clase positiva es minoritaria (la compra), el **accuracy engaña**: un clasificador "todo-negativo" alcanza ~94–98 % de accuracy y 0 % de recall. Por eso la métrica se alinea al error dominante: **PR-AUC** (*average precision*), que pone la clase positiva rara en primer plano, junto con *precision/recall* en el punto de operación; el ROC-AUC puede verse optimista con positivos raros. El **desbalance** se trata dentro del entrenamiento (`class_weight`/`scale_pos_weight`, o remuestreo) y **nunca** sobre el conjunto de prueba. Además, **discriminar ≠ calibrar**: una probabilidad bien ordenada puede estar mal calibrada, por lo que se reporta la **curva de calibración** y el **Brier score** ("si el modelo dice 0.8, ~80 % de casos similares deben ser positivos"). Finalmente, **el umbral es una política operativa**, no 0.5: se define por el costo del error y la capacidad de intervención ("el modelo estima; el umbral decide").

### 2.2 Validación que respeta el tiempo y anti-fuga
Cuando los datos tienen estructura temporal, "el futuro no puede entrenar el modelo que evalúa el pasado". Una *feature* es válida solo si su información existía **antes** del momento de predicción. De aquí se derivan dos decisiones: un **split temporal** *out-of-time* (entrenar con el pasado, evaluar con el futuro) y un **corte anti-fuga** (usar solo el comportamiento previo al evento que define la etiqueta). El *leakage* se audita sobre el **pipeline completo** (escalado/imputación/rebalanceo dentro de cada *fold*), no solo sobre el estimador. *(No se usa* forecasting *ni walk-forward: el problema es de clasificación; se transfiere el criterio temporal, no la maquinaria de series de tiempo.)*

### 2.3 Árboles, ensembles y boosting
Los árboles (CART) particionan el espacio de features y capturan no-linealidades e interacciones. Los **ensembles** mejoran por **diversidad** de errores: *bagging* (reduce varianza, p. ej. Random Forest) y **boosting** (corrección secuencial del residuo, p. ej. XGBoost/LightGBM/CatBoost). El boosting exige control de `learning_rate`, profundidad, regularización y *subsampling*. La **optimización de hiperparámetros** se trata como diseño experimental (Optuna/optimización bayesiana) optimizando la métrica objetivo bajo **validación cruzada estratificada**, y se selecciona por **promedio y variabilidad** entre *folds*, no por el mejor *trial* aislado.

### 2.4 Aprendizaje no supervisado: clustering
Representar es elegir una geometría: el **escalado es parte de la definición de similitud** (sin él, la variable de mayor magnitud domina la distancia). **K-Means** minimiza la inercia intra-cluster; **PCA** sirve para *inspeccionar* (no "prueba" clusters). El número de grupos *k* se elige con **codo + silhouette**, pero "el mejor *k* no es el que maximiza una métrica, es el que sostiene una partición defendible". Principio central: **"cluster ≠ segmento"** — un cluster es una partición del algoritmo; un segmento debe poder **nombrarse, dimensionarse, perfilarse, validarse y accionarse** de forma diferenciada.

### 2.5 Diseño experimental: propensión ≠ *uplift*
La **propensión** estima *quién comprará*; el **uplift** estima el *efecto causal* de un tratamiento (un incentivo). El uplift requiere un grupo **tratado** y uno de **control**; con datos **observacionales** (sin variable de tratamiento) **no es estimable**, y usar la propensión como si fuera uplift seleccionaría "compradores seguros" y afirmaría causalidad no verificable. La vía correcta es **diseñar el experimento** (A/B) que mediría el efecto, lo que ata el clasificador (targeting) con la decisión de negocio (§3.5).

### 2.6 Arquitectura de grandes datos
El **ciclo de vida del dato** va de la ingesta al consumo analítico. Se distingue una arquitectura **de referencia** (productiva) de la **implementada**. La **arquitectura Medallion** (Bronze → Silver → Gold) sobre **Delta Lake** da transacciones ACID, *time-travel* y *schema enforcement* sobre un lago de datos. Frente a **Lambda** (batch + streaming en paralelo), el estilo **Kappa** unifica el procesamiento en un único flujo de *streaming*; con **Auto Loader** + *triggers* + *checkpoint* se obtiene un **replay de streaming** reproducible sobre datos batch. El procesamiento distribuido se hace con **Apache Spark**; el particionamiento se decide por **tamaño** de cada capa (evitando micro-archivos), y los modelos se **persisten** con MLflow para *scoring* y consumo.

### 2.7 Visualización y comunicación
La visualización **abre una hipótesis, no cierra una decisión**. Para una audiencia de negocio, la comunicación se organiza alrededor de la **Pregunta de Oro**, con un tablero ejecutivo desplegado que prioriza **narrativa y diseño** sobre el detalle técnico, y filtrado/interactividad para sostener la defensa en vivo.

### 2.8 Referencias
<!-- [ESTADO] completar formato bibliográfico (APA) al consolidar -->
- **Dataset:** Kechinov, M. *eCommerce behavior data from multi category store* (REES46), Kaggle. *[completar URL/fecha de acceso]*
- **Aprendizaje Automático:** material del curso SI7009 (M. Terán, EAFIT, 2026) — destilado en `docs/05_contexto_aprendizaje_automatico.md`.
- **Grandes Datos:** material del curso SI7006 — y `docs/02_arquitectura_bigdata_y_databricks.md` (arquitectura, Medallion, Kappa, particionamiento).
- **Visualización:** material del curso SI7007 — destilado en `docs/07_contexto_visualizacion.md`.
- **Herramientas/librerías:** scikit-learn, LightGBM/XGBoost, Optuna, Apache Spark + Delta Lake, MLflow, Power BI. *[completar versiones y citas]*
- *(Referencias adicionales de PR-AUC, calibración, K-Means y diseño de experimentos a completar en la consolidación.)*

## 3. Desarrollo metodológico de modelos de ML
<!-- [REQUISITO] SI7009: modelado sup+no sup, evaluación, selección, métricas, caso de uso. ESTE ES EL CORAZÓN PARA AA. -->

### 3.1 Entendimiento del problema, pregunta de negocio e hipótesis
<!-- [FUENTE] doc 00 §1, §5. [ESTADO] redactar -->
- **Pregunta de negocio** y **Pregunta de Oro** (doc 00 §1).
- Definición del target: unidad = sesión; positivo = la sesión contiene `purchase`.
- Por qué propensión y NO uplift causal (datos observacionales, sin tratamiento). *(doc 00 §5.1, §5.3)*

### 3.2 Análisis Exploratorio de Datos (EDA)
<!-- [REQUISITO] SI7009 + estructura oficial. [FUENTE] notebooks/analysis/eda_ecommerce + doc 00 §6, §17. [ESTADO] borrador -->

El EDA oficial (`notebooks/analysis/eda_ecommerce.ipynb`) se ejecuta sobre las capas Silver/Gold mediante una capa de agregados en Spark que baja a pandas únicamente tablas resumen, de modo que ningún gráfico re-escanea los 14 GB. Todos los números de esta sección corresponden a la **base limpia** (con la ventana corrupta 14–17 de noviembre en cuarentena; ver §3.2.2).

#### 3.2.1 Entendimiento de los datos
El dataset es **REES46**, un *clickstream* multi-categoría de una tienda de e-commerce (Octubre–Noviembre 2019). Cada registro es un evento con esquema: `event_time`, `event_type` (`view` / `cart` / `purchase`), `product_id`, `category_id`, `category_code`, `brand`, `price`, `user_id` y `user_session`. El volumen crudo es de ~14.5 GB (Octubre 5.5 GB + Noviembre 9 GB), **109.5 M de eventos**.

La unidad de análisis del proyecto es la **sesión** (`user_session`): la tabla Gold tiene grano *1 fila = 1 sesión*, con **22.99 M de sesiones** y una tasa de etiqueta (sesiones que contienen `purchase`) de **0.0610** sobre la base completa y **0.0597** sobre la base limpia. Tras la limpieza y la cuarentena quedan ~87.6 M de eventos / **56.76 M de unidades** producto-en-sesión.

#### 3.2.2 Preparación de los datos
La preparación se codifica íntegramente en el pipeline Medallion (no en el notebook de análisis): tipado, deduplicación, filtro `price > 0`, reconstrucción de la sesión y **corte anti-fuga** (cada feature de la sesión se calcula usando solo el comportamiento **previo** al primer `cart`/`purchase`, de modo que el modelo nunca observa información posterior al evento que define la etiqueta).

- **Calidad de datos — cuarentena de la ventana 14–17 nov.** Se detectó una ventana corrupta con un criterio doble (etiqueta rota *o* volumen de eventos anómalo): el `purchase` está roto el 15–17 (15-nov registra 0 compras pese a ~467 k carritos; el backlog se vuelca el 16–17), y el 14-nov presenta un volcado de eventos (2.86 M views / 165 k carts, por encima del propio Black Friday) con conversión a la mitad (0.77 %). Estos días se marcan con `label_window_corrupt` y se **excluyen tanto de entrenamiento como de evaluación**. El 13-nov **no** se cuarentena (volumen normal). El hallazgo coincide con un problema reportado por la comunidad del dataset.
- **Taxonomía / `"Unknown"`.** El `category_id` es limpio y completo, pero ~32 % de los eventos tienen la macro-categoría como `"Unknown"` y no es re-derivable (0 de 413 ids con macro desconocida la tienen conocida en otra fila). Se mantiene `"Unknown"` como categoría legítima; para el modelado se usa `category_id` (mejor señal que el string macro) y para el tablero se etiqueta como *"Sin taxonomía"*.
- **Outliers de precio.** No los hay en sentido estricto: el precio está **acotado en la fuente** (p99.9 = 2562, máximo = 2574), por lo que no se winsoriza. Esto refuerza el hallazgo de que el precio no es el freno de la conversión.
- **Sesiones-bot.** Cola despreciable (sesiones con > 200 eventos = 0.002 %); no ameritan un tratamiento especial.

#### 3.2.3 Análisis descriptivo e insights
**Funnel sobre la base limpia (unidad = producto-en-sesión):** `cart` **3.86 %**, **conversión 2.27 %**, cierre 58.81 % y **abandono 41.19 %** (56.76 M unidades; 903 k carritos abandonados; ~$250.4 M en juego). *(La ventana corrupta inflaba el abandono a 51.7 % y deprimía el cierre a 48.3 %.)*

El negocio es la **electrónica**: conversión 3.58 %, el mayor volumen (20.03 M unidades) y el mayor pool de carritos abandonados (~$187.1 M de los $250.4 M totales). La concentración del revenue es extrema: **electronics 76.9 %**, top-3 categorías 87.3 %, y el 50 % del revenue lo generan apenas **50 productos (0.1 %)**.

De aquí salen **dos palancas de negocio**:
- **(A) Recuperar carritos de alta intención que no cierran** (electrónica; Samsung + Apple concentran el 68.6 % de los carritos de electrónica, con abandono ~39–40 %).
- **(B) Retener al núcleo recurrente**: el 35.7 % de compradores genera el 73.7 % del revenue, con ticket recurrente ($1,460) muy superior al one-time ($289) y una segunda compra a 2.9 días de mediana.

**Insights para el modelado (clave y contraintuitivos):** el precio **no es el freno** y la decisión es casi inmediata (mediana 2.2 min). Las features conductuales tienen correlación **débil y negativa** con la compra —los compradores navegan *menos* y deciden rápido— y existe **multicolinealidad** alta (`total_views` ≈ `distinct_products_viewed`, corr 0.92). La conclusión metodológica es que el poder predictivo **no es lineal** (está en interacciones y no-linealidades), lo que justifica el salto de una logística baseline a un GBM (§3.3).

**Contexto temporal.** La divergencia Octubre→Noviembre es mayormente **real** (estacionalidad pre-fiestas), no un artefacto de limpieza; el Black Friday (29-nov) es un pico real pero modesto (~+30 %), y el pico ×6 del 17-nov era el artefacto de la ventana corrupta. *(Contexto de la comunidad, no oficial: tienda de Rusia/CIS, precios en USD, `event_time` en UTC.)*

### 3.3 Selección de modelos, Ingeniería de Características, Entrenamiento, Evaluación
<!-- [REQUISITO] SI7009 núcleo. [FUENTE] notebooks/modeling/02,03,04 + doc 00 callout modelado + docs/05. -->
<!-- [ESTADO] ⏳ DEPENDE DE SARA: números finales tras re-correr con snapshot 14–17. Dejar placeholders [PR-AUC=…]. -->

#### 3.3.1 Características e Ingeniería de Características
- Set de 22 columnas de la Gold (anti-fuga, pre-corte). *(doc 00 §13)*
- FE: decisividad/ritmo (`revisit_intensity`…), codificación cíclica de hora, manejo de `category_id`. *(nb 02 §4)*
- Decisiones: features excluidas (IDs), `sin_navegacion_previa`, multicolinealidad. *(doc 00 §13, revisión modeling)*

#### 3.3.2 Modelos (selección)
- **Supervisado — propensión:** baseline trivial (Dummy) → logística → comparación de familias (RF/XGB/LightGBM) → LightGBM. *(nb 02)*
- **No supervisado — clustering:** K-Means k=4 (por accionabilidad) + DBSCAN como contraste. *(nb 04)*
- Justificación de cada elección (criterio defendible, no moda).

#### 3.3.3 Entrenamiento
- **Split temporal Opción C** (train Oct + Nov ≤23 / test 24–30 nov), con evidencia de drift/PSI. *(nb 03, doc 00 §17.3)*
- Cuarentena 14–17 aplicada a train y test; `StratifiedKFold` solo en CV interna; desbalanceo con `scale_pos_weight`. *(nb 02)*
- HPO con Optuna optimizando PR-AUC en CV (≥50 trials para el fit final). *(nb 02 §8)*

#### 3.3.4 Evaluación
- **Métricas:** PR-AUC + Brier (calibración), nunca accuracy. Reporte con/sin Black Friday. *(nb 02 §9–10)*
- Curva de calibración, curva PR, umbral como política operativa (no 0.5). *(nb 02 §9, §11)*
- Importancia por permutación → top features (`max_price_viewed`, `electronics_view_share`). *(nb 02 §12)*
- Clustering: perfilado, "cluster≠segmento", validación (silhouette + estabilidad). *(nb 04 §5, §9)*
- **El cruce clasificador × clustering** → segmento objetivo C0 (el insight). *(nb 04 §6, §9)*

### 3.4 Análisis y conclusiones del componente de ML
<!-- [FUENTE] nb 02/04 + doc 00. [ESTADO] depende de números finales. -->
- Lectura del resultado (qué predice la compra; nivel de precio y foco electrónica).
- Conexión con la decisión de negocio: segmento C0 → A/B test.
- Limitaciones y trabajo futuro (historia de usuario, uplift como extensión).

### 3.5 Cierre del alcance: diseño del A/B test
<!-- [REQUISITO] cierra el alcance de ML (docs/05 §5) y responde al revisor de la propuesta. [FUENTE] doc 00 §5.1, §17.1; clustering nb 04. [ESTADO] borrador -->

#### 3.5.1 Por qué un diseño de A/B y no una estimación de *uplift*
La propuesta original prometía un modelo de *uplift* (efecto incremental de un incentivo). **No es estimable con estos datos**: el uplift mide un efecto causal y requiere un grupo **tratado** y uno de **control**; el dataset es **observacional** y no contiene ninguna variable de tratamiento. Por eso el proyecto no estima causalidad: entrega el **diseño del experimento** que *generaría* esos datos y permitiría medir el efecto. De hecho, el A/B es precisamente el mecanismo que produce los pares tratamiento/control sobre los que, en una segunda fase, sí podría entrenarse un modelo de uplift. El clasificador de propensión (§3.3) no reemplaza al experimento: **selecciona a quién vale la pena exponer al incentivo**; el experimento mide si el incentivo funciona.

#### 3.5.2 Objetivo e hipótesis
**Pregunta:** ¿un incentivo inmediato (p. ej. envío gratis o un descuento por tiempo limitado) mostrado a visitantes de **alta intención de electrónica** que están por abandonar, **aumenta la conversión** sin destruir el margen?

- **H₀:** el incentivo no cambia la tasa de conversión del grupo objetivo (`p_tratamiento = p_control`).
- **H₁:** el incentivo aumenta la conversión (`p_tratamiento > p_control`).
- **Prueba:** comparación de dos proporciones (z-test) sobre la métrica primaria; intervalo de confianza sobre el *lift*.

#### 3.5.3 Población objetivo y targeting con el modelo
El experimento se restringe al **segmento C0** ("comprador de electrónica de alto valor": 42.6 % del tráfico, conversión 7.5 %, el único que combina propensión + valor + volumen — §3.3.4). Dentro de C0, el **modelo de propensión** define el subconjunto elegible (los de probabilidad calibrada por encima del umbral operativo), de modo que el incentivo se gasta donde hay intención real y no en toda la base. *(Matiz honesto y defendible: targetear por alta propensión puede incluir "compradores seguros" que habrían comprado igual; el A/B mide el efecto **promedio** sobre el grupo tratado, y son justamente sus datos los que después permitirían distinguir a los persuadibles vía uplift. No invertimos el orden.)*

#### 3.5.4 Unidad de aleatorización y asignación
- **Unidad = visitante** (asignación "pegajosa" por `user_id`/cookie), no la sesión, para evitar que una misma persona vea ambas experiencias (contaminación) entre visitas.
- Asignación **50/50** aleatoria a **Control** (experiencia actual, sin incentivo) y **Tratamiento** (incentivo). Un solo brazo de tratamiento para que el efecto sea atribuible; variantes de incentivo quedan como diseño multivariante futuro.
- **Validez:** test A/A previo + chequeo de *Sample Ratio Mismatch* (la división real debe ser ~50/50); horizonte **fijo** (o corrección secuencial) para no "espiar" y inflar el falso positivo.

#### 3.5.5 Métricas
- **Primaria (OEC):** tasa de conversión por visitante (¿compró?).
- **Secundarias:** tasa de abandono de carrito, ingreso por visitante, ticket promedio.
- **Guardarraíl (no negociables):** **margen/utilidad por visitante** (un descuento puede subir conversión y destruir margen) y tasa de devoluciones. El experimento solo "gana" si sube la primaria **sin** romper el guardarraíl.

#### 3.5.6 Tamaño de muestra, MDE y duración
Línea base de C0: conversión **p₀ = 7.5 %**. Con **α = 0.05 (bilateral)** y **potencia = 80 %**, el tamaño por brazo según el efecto mínimo detectable (MDE) es:

| MDE | p₀ → p₁ | n por brazo | n total |
|---|---|---|---|
| +1.0 pp (absoluto) | 7.5 % → 8.5 % | ~11,600 | ~23,200 |
| +10 % (relativo) | 7.5 % → 8.25 % | ~20,300 | ~40,600 |
| +0.5 pp (absoluto) | 7.5 % → 8.0 % | ~44,900 | ~89,800 |

*(Fórmula de dos proporciones: n ≈ (z_{α/2}+z_β)²·[p₀(1−p₀)+p₁(1−p₁)] / (p₁−p₀)².)*

**Duración.** C0 ≈ 42.6 % de ~346 k sesiones/día → del orden de **10⁵ visitantes-C0 elegibles por día**, así que el tamaño de muestra se alcanza en **pocos días** incluso para el MDE exigente de +0.5 pp. El cuello de botella no es el volumen sino la **validez temporal**: se corre un mínimo de **2 semanas** para cubrir ≥2 ciclos semanales completos y se **excluyen periodos anómalos** (Black Friday y la ventana 14–17 nov) para no confundir el efecto del incentivo con estacionalidad.

#### 3.5.7 Regla de decisión y análisis
Análisis por **intención de tratar**. Se declara ganador el Tratamiento si el *lift* de conversión es **estadísticamente significativo** (IC al 95 % que excluye 0) **y** el guardarraíl de margen no se deteriora; en ese caso se despliega al segmento. Como extensiones: reducción de varianza (CUPED) y análisis de heterogeneidad del efecto dentro de C0 (primer paso hacia el uplift de segunda fase).

## 4. Tecnología: Ingeniería de Datos y uso de tecnología
<!-- [REQUISITO] SI7006 (obligatorio): ciclo de vida + arquitectura. [FUENTE] doc 02 + doc 00 §7. [ESTADO] borrador -->

El proyecto distingue dos arquitecturas: la **implementada** (lo que de verdad corrimos en Databricks Free Edition) y la **de referencia** (cómo se vería en producción real), tal como se detalla en `docs/02_arquitectura_bigdata_y_databricks.md`.

### 4.1 Desarrollo del proyecto (arquitectura implementada)
- **Fuentes de datos y naturaleza.** REES46: datos de *clickstream* estructurados, entregados como archivos *batch*. El crudo se aloja en un **Volume de Databricks** (`ecommerce_raw`), que ya está respaldado por almacenamiento de objetos.
- **Ingesta (modo).** Pese a que la fuente es batch, la ingesta se implementa como **replay de streaming estilo Kappa**: **Auto Loader** (`cloudFiles`) con `Trigger.AvailableNow` + checkpoint. Esto demuestra un pipeline de streaming reproducible (no en bucle) sin necesidad de un broker (Kafka) ni de AWS. Verificado: **109.95 M filas ingeridas sin duplicar**.
- **Almacenamiento — lago de datos en Delta.** Arquitectura **Medallion** sobre **Delta Lake**: Bronze (crudo tipado) → Silver (limpio, deduplicado, `price>0`) → Gold (features por sesión, contrato congelado de 22 columnas, §3.3.1). El lago de datos es el Volume + las tablas Delta.
- **Particionamiento por capa (decisión por tamaño).** Bronze/Silver se particionan por **fecha** (capas grandes → particiones de ~128 MB+; Silver además `ZORDER (category_id)`). La **Gold de sesión NO se particiona** (~1.33 GB ≪ 1 TB → `OPTIMIZE` + `ZORDER (session_date, user_id)`; particionarla por fecha generaría micro-archivos de ~22 MB, el anti-patrón). La frontera train/test es un filtro sobre `session_date` con *data-skipping*. Evidencia medida con `DESCRIBE DETAIL` (doc 02 §3).
- **Framework de procesamiento.** **Apache Spark** (PySpark + Spark SQL) para todo el pipeline y la capa de agregados del EDA/BI.
- **Persistencia de modelos.** **MLflow** para *tracking* (params, PR-AUC/Brier), el modelo calibrado y su *signature*; el *scoring batch* materializa el **Contrato 2** (`user_session`, probabilidad calibrada, segmento) en Delta. *(Nota Free Edition: hay que llamar `set_registry_uri("databricks-uc")` antes de `set_experiment`.)*

### 4.2 Despliegue (escenario hipotético de implementación real)
- **Arquitectura de referencia productiva.** Kafka/Kinesis → Structured Streaming/Flink → Delta sobre S3/GCS → warehouse/Athena → *serving* en tiempo real. La implementada es la traducción de esta referencia a los límites de la plataforma del curso.
- **Aplicaciones (consumo).** La Gold **agregada** (12 CSV pequeños) alimenta el **tablero Power BI** publicado en Power BI Service; el scoring batch enriquece el tablero y define el segmento del A/B.
- **Decisión de diseño — Volume vs S3 (Q&A de defensa).** Se usa el Volume y no un bucket S3 externo porque (1) el Volume ya está respaldado por object storage y Auto Loader puede apuntar a su path, así que el replay Kappa se demuestra sin bucket externo; (2) S3 agregaría una cuenta AWS y credenciales que gestionar (choca con "no exponer secretos") y las *external locations* son limitadas en Free Edition; (3) re-ingestar 14 GB quemaría tiempo y cuota sin resolver ningún problema actual. En la arquitectura de referencia, S3 + Auto Loader sí aparecen como la capa productiva.

## 5. Visualización y comunicación de datos
<!-- [REQUISITO] SI7007 (rúbrica 35%: despliegue/funcionalidad/narrativa/defensa). [FUENTE] reports/powerbi + reports/data/README + docs/07 + doc 00 §8. [ESTADO] borrador (tablero en finalización por Kelly: falta v2 con scores) -->

### 5.1 Requerimientos de comunicación
El tablero responde la **Pregunta de Oro** (*¿dónde se concentra la fuga y qué segmento es la mayor oportunidad?*) para una audiencia **ejecutiva y de jurado** que pesa **narrativa, diseño e interactividad** por encima del detalle técnico. El requisito de la rúbrica (SI7007) es un tablero **desplegado y accesible**, **fluido**, con un **pitch de negocio** que termine en recomendaciones accionables y una **defensa** filtrando en vivo.

### 5.2 Análisis (mapa pregunta → gráfico)
Cada visual responde una pregunta de negocio, organizada alrededor de las **dos palancas** del EDA (§3.2.3):
- **Palanca A — dónde está la fuga:** funnel por categoría (`agg_funnel_categoria` → treemap/barras), dinero en carritos abandonados (`agg_revenue_en_juego` → treemap por área), y drill de marcas del premio (`agg_marca_electronics` → barras).
- **Palanca B — a quién retener:** segmentos de comprador (`agg_segmentos_comprador` → combo de doble eje, %compradores vs %revenue).
- **Contexto/diagnóstico:** embudo global (`agg_funnel_embudo`), tipología de visitante (`agg_tipologia_visitante`), cuándo intervenir (`agg_hora_dow` → heatmap hora×día), y evolución temporal con *toggle* día↔hora y slicers de categoría/marca (`agg_metricas_dia_hora`, `agg_metricas_diarias_categoria`, `agg_electronics_marca_diaria`).

### 5.3 Diseño
Cuatro páginas con **portada y navegación** (ver `reports/powerbi/`): (1) **Análisis global** (KPIs + funnel + tipología + segmentos con Top-N dinámico), (2) **Detalle Electronics** (el negocio: ticket, abandono, marcas, scatter ticket×abandono), (3) **Contexto temporal** (tráfico vs conversión diaria, con **anotaciones de calidad de datos**: ventana 14–17 nov y Black Friday). Paleta púrpura coherente. Principio: cada página sostiene un paso de la narrativa, no una pila de gráficos.

### 5.4 Implementación
- **Power BI** conectado a la **Gold agregada** (12 CSV en `reports/data/`), no a las 23 M filas crudas — la estrategia de cuota del proyecto (doc 02). El `.pbix` vive en `reports/powerbi/Tablero_PI.pbix`.
- Buenas prácticas de BI aplicadas: la **conversión se crea como *medida*** (`SUM(purchases)/SUM(views)`) para que respete el filtro activo, no como porcentaje pre-agregado; *field parameters* para el *toggle* día↔hora y *slicers* de categoría/marca.
- **Desplegado en Power BI Service** (requisito de despliegue, 10 %). *[completar: enlace público accesible]*.
- **Pendiente (v2):** conectar los **scores del modelo** (Contrato 2 de §3.3) y la **matriz de oportunidad** (probabilidad × valor) para cerrar el targeting del A/B. *(El tablero está en finalización por Kelly.)*

### 5.5 Validación
Los CSV se generan con las **mismas definiciones del EDA** → los cortes por categoría/marca/segmento **cuadran al céntimo** con `eda_ecommerce.ipynb`. Dos validaciones explícitas: (1) la migración a Spark SQL produce los **mismos 12 CSV** que la versión PySpark (verificado; solo ±0.01 en `ticket_medio` por redondeo); (2) **gotcha documentado** — `agg_funnel_categoria`/`agg_revenue_en_juego` **excluyen `Unknown`** (~32 %), por lo que el **KPI global** del titular (cart 3.86 % / conv 2.27 % / abandono 41.19 % / 903 k carritos / $250.4 M) se toma de `agg_funnel_global` (incl. Unknown), **no** de sumar las filas por categoría.

## 6. Conclusiones generales del proyecto
<!-- [FUENTE] doc 00 §1, §5, §17 + resultados finales. [ESTADO] redactar al cierre. -->
- Respuesta a la Pregunta de Oro: dónde se concentra la fuga y qué segmento es la oportunidad.
- Recomendaciones accionables (las dos palancas + segmento C0 + A/B test).
- Aporte por materia (ML / Grandes Datos / Visualización).

## 7. Referencias
<!-- [FUENTE] consolidar. [ESTADO] redactar -->

---

<!--
MAPA DE COBERTURA DE LA RÚBRICA (verificación — borrar antes de exportar)
- SI7009 Aprendizaje Automático → §3 completo (sup §3.3.2, no sup §3.3.2, evaluación §3.3.4, métricas §3.3.4, caso de uso §3.1).
- SI7006 Grandes Datos → §4 completo (ciclo de vida, arquitectura ref §4.2, pipeline/ingesta/almacenamiento/Spark §4.1, despliegue/persistencia §4.2, visualización §5).
- SI7007 Visualización → §5 (despliegue + funcionalidad + narrativa) + la defensa se cubre en el ensayo/PPTX.

DEPENDENCIAS ABIERTAS
- §3.3.3/§3.3.4/§3.4: números finales del modelo (Sara, tras snapshot 14–17). Placeholders [..].
- §5: tablero v2 con scores (Kelly, tras Contrato 2).
- §3.5: diseño del A/B (Yeison) — se puede redactar ya.

ESTRUCTURA: el top-level (§1–§7) replica el "Contenido sugerido" oficial 1:1;
el A/B va como §3.5 (cierre del alcance de ML), no como sección suelta.
-->
