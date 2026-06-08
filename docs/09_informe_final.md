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
<!-- [REQUISITO] Estructura oficial. [FUENTE] doc 00 §1, §5. [ESTADO] redactar -->
- Contexto del problema (e-commerce, fuga de conversión) y motivación.
- Dataset REES46 (clickstream multi-categoría, Oct–Nov 2019, ~14.5 GB).
- **Frase de ascensor** y alcance del proyecto en una frase (doc 00 §1).
- Qué entrega el proyecto: propensión + clustering + diagnóstico + diseño de A/B test.

## 2. Marco teórico y referencias
<!-- [REQUISITO] Estructura oficial. [FUENTE] docs/05 (ML), doc 02 (Big Data), docs/07 (Viz). [ESTADO] redactar -->
- Clasificación supervisada y evento raro (PR-AUC, calibración). *(docs/05 §1)*
- Clustering / aprendizaje no supervisado (K-Means, silhouette, "cluster≠segmento"). *(docs/05 §1.4)*
- Anti-fuga y validación que respeta el tiempo. *(docs/05 §1.3)*
- Arquitectura de grandes datos (Medallion, Kappa, Delta Lake). *(doc 02)*
- Diseño experimental / por qué A/B y no uplift causal. *(doc 00 §5.1, docs/05 §5)*
- Referencias (REES46, papers/cursos, librerías).

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
<!-- [REQUISITO] cierra el alcance de ML (docs/05 §5) y responde al revisor de la propuesta. [FUENTE] por redactar (Yeison); alcance doc 00 §5.2(4), §18.7. [ESTADO] PENDIENTE. -->
- Por qué A/B y no uplift causal (datos observacionales, sin tratamiento/control). *(doc 00 §5.1)*
- Hipótesis, unidad de aleatorización, métrica primaria, MDE, tamaño de muestra/poder, duración.
- Segmento objetivo (C0) y cómo el modelo de propensión hace el targeting fino dentro del segmento.

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
<!-- [REQUISITO] SI7007 (rúbrica 35%: despliegue/funcionalidad/narrativa/defensa). [FUENTE] reports/powerbi + docs/07 + doc 00 §8. -->
- **Requerimientos de comunicación:** Pregunta de Oro, audiencia (jurado BA/Diseño). *(doc 00 §8)*
- **Análisis y diseño:** mapa pregunta→gráfico, paleta, narrativa. *(doc 00 §8, reports/data/README)*
- **Implementación:** Tablero Power BI (4 páginas) publicado en Power BI Service. *(doc 00 §2.3, reports/powerbi/)*
  - Análisis global · Detalle Electronics · Contexto temporal · (v2) matriz de oportunidad + scores. *(⏳ Contrato 2)*
- **Validación:** coherencia de cifras vs EDA (12 CSV verificados). *(doc 00 §17.1)*
- Enlace público del tablero desplegado (requisito de despliegue 10%).

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
