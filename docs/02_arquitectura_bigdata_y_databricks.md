# Arquitectura de Grandes Datos y uso de Databricks — Proyecto Integrador G8

**Propósito:** dejar documentadas (1) las dos arquitecturas del proyecto y su alineación con el temario del curso de Almacenamiento y Procesamiento de Grandes Volúmenes de Datos, (2) la estrategia de particionamiento por capa, (3) las reglas de uso para cuidar la cuota de Databricks Free Edition, y (4) las limitaciones técnicas conocidas, preparadas como posibles preguntas del profesor.

Criterio rector del profesor: *usar las herramientas y métodos adecuados según el problema*. Por eso la estrategia no es "meter toda la tecnología posible", sino **justificar por qué cada herramienta corresponde a cada capa** y por qué, para un dataset histórico estático, lo implementado es la elección correcta.

---

## 1. Las dos arquitecturas y por qué existen

El proyecto distingue dos arquitecturas, y esa distinción es en sí misma un entregable que demuestra criterio:

- **Arquitectura de referencia (producción):** cómo se construiría el sistema en una empresa real con eventos en vivo. Es donde viven Kafka/Kinesis, Flink, el serving en tiempo real, etc. No se implementa; se documenta y se defiende.
- **Arquitectura implementada (académica):** lo que de verdad corre sobre Databricks Free Edition con los datos históricos de Kaggle. Usa ingesta batch + un *replay* de streaming sobre los CSV, Medallion en Delta, y consumo vía Spark SQL + Power BI.

### Encuadre Lambda / Kappa (Unidad 1)

El temario enseña arquitecturas Batch, Streaming e Híbridas de tipo **Lambda** y **Kappa**. Nuestro diseño se ubica explícitamente ahí:

- **Lambda** = capa batch + capa de velocidad (streaming) + capa de servicio, en paralelo.
- **Kappa** = todo se trata como un flujo; una sola tubería de streaming, y el reprocesamiento se hace **reproduciendo (replay) el log de eventos**.

Nuestra implementación adopta un **diseño de estilo Kappa**: tratamos los CSV históricos como un log reproducible y los procesamos con la *misma* ruta de código de Structured Streaming (Auto Loader). En producción, ese mismo código consumiría un flujo en vivo de Kafka/Kinesis en lugar del replay. Un solo camino de procesamiento para datos históricos y en tiempo real: ese es el argumento de defensa.

---

## 2. Mapeo por capa: referencia vs. implementada vs. temario

| Capa del ciclo de vida | Arquitectura de referencia (producción) | Arquitectura implementada (Databricks Free) | Unidad / tecnologías del curso |
|---|---|---|---|
| Ingesta / Streaming | Kafka o AWS Kinesis / GCP Pub/Sub como bus de eventos | Descarga batch de Kaggle a un Volume (Unity Catalog) | U1, U4 — Kafka, Kinesis, Pub/Sub |
| Procesamiento de flujo | Spark Structured Streaming o Apache Flink (baja latencia) | Structured Streaming + Auto Loader (`cloudFiles`) con `Trigger.AvailableNow()` leyendo los CSV como micro-batches (replay estilo Kappa) | U4 — Flink, Structured Streaming |
| Almacenamiento | Delta Lake (o Iceberg/Hudi) sobre object storage S3/GCS; Parquet columnar; particionamiento | Delta Lake (Bronze/Silver/Gold) sobre el Volume; Parquet por debajo | U2 — Delta/Iceberg/Hudi, Parquet/ORC, S3/GCS, HDFS |
| Procesamiento batch / transformación | Spark (PySpark) sobre EMR/Dataproc; patrón ELT; Medallion | Spark (PySpark) en serverless; ELT; Medallion (ya implementado) | U3 — Spark, EMR/Dataproc, ETL/ELT |
| Consumo / analítica | Warehouse (Snowflake / BigQuery / Redshift) o Athena sobre S3; dbt; serving en tiempo real | Spark SQL sobre Gold; MLflow para el modelo; scoring batch a tabla Delta | U2, sección consumo — Snowflake, BigQuery, Redshift, Athena, dbt |
| Visualización | BI sobre el warehouse | Power BI publicado en Power BI Service, conectado a la Gold agregada | (curso de Visualización) |

**Herramientas del temario que NO implementamos pero sí justificamos en la arquitectura de referencia:** Kafka, Flink, Kinesis/Pub/Sub, AWS Glue/Dataflow, Iceberg/Hudi, HDFS, EMR/Dataproc, Snowflake, BigQuery, Redshift, Athena, dbt. Mencionarlas con el *por qué* de cada una demuestra amplitud sin reventar el alcance.

---

## 3. Estrategia de particionamiento por capa (por patrón de consulta)

La estrategia de particionamiento se define **por capa**, según los **patrones de consulta reales** y no los de ingesta —principio rector del curso—. El anti-patrón crítico, explícitamente penalizado, es **particionar por alta cardinalidad** (`user_id`, `user_session`, timestamp exacto): generaría millones de micro-archivos y un *overhead* de metadatos que degrada cada consulta. Un buen particionamiento reduce las particiones escaneadas por consulta, lo que **mejora el tiempo de respuesta y, de paso, cuida la cuota de cómputo serverless** de Databricks Free Edition (menos datos escaneados = menos cómputo = no agotar la cuota).

| Capa | Particionamiento | Por qué | Optimización física |
|---|---|---|---|
| **Bronze** | Por **fecha de evento** (`event_date`) | Alineada al replay incremental (los CSV llegan por mes/día); permite reprocesar un día sin tocar el resto | Tamaño de archivo objetivo 128 MB–1 GB; evita micro-archivos |
| **Silver** | Por **fecha** (+ **categoría** si la selectividad lo amerita) | Eventos limpios/tipados y sesiones reconstruidas se consultan por ventana temporal y por categoría | `ZORDER` sobre columnas de mayor selectividad (p. ej. `category_id`); compactación con `OPTIMIZE` |
| **Gold (features por sesión)** | **Sin particionar** (~23M filas, ~1–2 GB) | Particionar por fecha daría ~25 MB/partición → **anti-patrón de archivos pequeños**; por tamaño (≪ 1 TB) se **clusteriza**, no se particiona | `OPTIMIZE` + `ZORDER BY (session_date, user_id)`: el *data-skipping* de Delta poda el split train/test por `session_date` sin micro-archivos; `user_id` agrupa el join de clustering |
| **Gold agregada (BI)** | n/a (tablas pequeñas) | Funnel/métricas ya agregados (≤ cientos de filas) que consume Power BI | Se exportan como CSV/Parquet pequeños a `reports/data/` |

**Reglas concretas (firmadas en la propuesta corregida, Curso 2):**
1. **Nunca** particionar por columnas de alta cardinalidad (`user_id`, `user_session`, timestamp exacto).
2. Particionar por columnas de **filtro frecuente** (fecha y categoría), no por las de ingesta.
3. **Tamaño de archivo** en el rango **128 MB–1 GB** por partición; usar `OPTIMIZE` para compactar y `ZORDER` para clustering por las columnas de mayor selectividad.
4. **Decidir la partición por TAMAÑO, no solo por columna.** Particionar por fecha aplica a las capas **grandes** (Bronze/Silver: 100M+ eventos, multi-GB → particiones de ~128 MB+). Una tabla **pequeña** (≪ 1 TB, p. ej. la **Gold de features ~1–2 GB**) **no se particiona**: hacerlo daría ~25 MB/partición (micro-archivos, el mismo anti-patrón que penalizamos). En su lugar `ZORDER`/clustering + *data-skipping* de Delta. Es la aplicación del rector *"la herramienta adecuada al problema"*: conocer la regla **y su excepción** (guía de Databricks: no particionar tablas ≪ 1 TB).
5. **Medir** —tamaño en disco, particiones escaneadas y tiempo de ejecución (`DESCRIBE DETAIL`)— para sustentar la optimización con evidencia, no por intuición.

**Frontera train/test en el pipeline.** El **split temporal octubre/noviembre** (octubre entrena, noviembre prueba) debe quedar **visible en el diagrama del pipeline** (Ilustración 2): Bronze/Silver están particionadas por fecha y la Gold lleva `session_date` con `ZORDER`/*data-skipping*, así que la separación entrenamiento/prueba se materializa como un **corte/filtro sobre la columna temporal** (partición en las capas grandes; data-skipping en la Gold), no como una mezcla aleatoria de filas. Esto refuerza el argumento anti-fuga del componente de Aprendizaje Automático.

> **Evidencia medida (4-jun · `DESCRIBE DETAIL`):** **Silver = 6.41 GB en 61 particiones por fecha** (~105 MB/partición, dentro del rango 128 MB–1 GB). **Gold de sesión = 1.33 GB, sin particionar, 6 archivos** (~220 MB) con `ZORDER (session_date, user_id)`. Particionar esa Gold por fecha habría dado ~22 MB/partición → el anti-patrón de archivos pequeños, confirmado con números. *(Esto es la regla 5 "medir, no por intuición" en acción.)*

> *Nota: esta sección sincroniza el doc con la estrategia de particionamiento que el Curso 2 de la propuesta corregida ya firma, y cierra el GAP marcado en `08_feedback_exposiciones_pregrado.md` §5 (el doc no la detallaba).*

---

## 4. Reglas de uso para cuidar la cuota de Databricks Free Edition

Databricks Free Edition es **solo cómputo serverless** y está sujeta a una política de uso justo: si se excede la cuota, **el cómputo se apaga por el resto del día** (en casos extremos, el resto del mes); los datos y configuraciones se conservan. El riesgo no es de capacidad (ya procesamos los 14 GB), sino de **quedarnos sin cómputo justo antes de una entrega**. Reglas:

1. **Desarrollar sobre muestra, ejecutar completo solo cuando haga falta.** Iterar features/modelos sobre una fracción de los datos (p. ej. unos días); correr el barrido de 14 GB únicamente para validar resultados finales.
2. **Leer de las capas ya materializadas, no reprocesar desde cero.** Bronze/Silver/Gold ya están como tablas Delta; partir de ellas en lugar de releer los CSV crudos cada vez.
3. **Usar checkpoints en el streaming** para que los `readStream` sean incrementales y no reprocesen todo en cada corrida.
4. **Evitar `count()`/`collect()`/full scans repetidos** sobre datos completos; usar `cache()` cuando se itere sobre el mismo DataFrame.
5. **Trigger correcto y lotes acotados:** usar `Trigger.AvailableNow()` (es obligatorio en serverless) y ajustar `maxFilesPerTrigger` / `maxBytesPerTrigger` para mantener la memoria predecible y respetar el tope de 9000 s por consulta.
6. **No correr cargas pesadas la noche anterior a una entrega.** Si la cuota se agota, se pierde el día. Dejar buffer.
7. **Regla de colaboración (clave):** en Free Edition los datos **no se comparten entre cuentas** (cada quien tiene su Volume y su cuota). Por eso: que **una sola persona** construya y materialice las capas pesadas; la **Gold agregada (pequeña)** se exporta al repo / almacenamiento compartido para que el resto trabaje modelado y Power BI sobre ella, sin re-correr los 14 GB en cada cuenta.

> **MLflow en serverless — gotcha verificado (5-jun).** En cómputo serverless (Free Edition),
> `mlflow.set_experiment(...)` falla con `CONFIG_NOT_AVAILABLE: spark.mlflow.modelRegistryUri` porque ese
> config no viene seteado y Spark Connect bloquea su lectura. **Fix:** antes de `set_experiment`, llamar
> `mlflow.set_tracking_uri("databricks")` y `mlflow.set_registry_uri("databricks-uc")`. Es un problema conocido
> de Databricks, **no del código**. El **logging de experimento** (params/métricas/modelo, §18.4 del doc 00) **no
> requiere** el model registry; el fix solo evita que la línea de *setup* se caiga. Verificado con smoke test el
> 5-jun (notebook `notebooks/analysis/_verif_post_audit.ipynb`); cierra el "MLflow setup" del riesgo 3. *(Insumo
> directo para Sara.)*

---

## 5. Limitaciones conocidas y preguntas probables del profesor (Q&A de defensa)

### Limitaciones de la plataforma (Databricks Free / serverless)
- Solo cómputo serverless, con cuota de uso justo (apaga el cómputo al excederla).
- Un solo workspace y un metastore por cuenta; datos no compartibles entre cuentas.
- Sin Scala ni R (solo Python/SQL); sin APIs RDD ni `SparkContext`.
- Sin Spark UI (se usa el *query profile*); logs solo del lado cliente.
- Tope de 9000 s (~2.5 h) por consulta serverless.
- Streaming: solo `Trigger.AvailableNow()` / `Trigger.Once()`; no hay triggers continuos ni por intervalo de tiempo.
- Sin *online tables* → no hay serving de features en tiempo real (por eso el serving real vive en la arquitectura de referencia y nosotros hacemos scoring batch).

### Preguntas probables y respuestas

**¿Por qué batch y no streaming real?**
No existe una fuente de eventos en vivo: el dataset es histórico. Adoptamos un diseño estilo Kappa que reproduce los datos como un flujo con la misma ruta de código de Structured Streaming; en producción ese código consumiría Kafka/Kinesis. Demostramos la habilidad de streaming sin inventar una fuente en tiempo real.

**¿Por qué `Trigger.AvailableNow()`?**
Es el trigger soportado en cómputo serverless y el modo "triggered" más eficiente en costo: procesa todo lo disponible como micro-batches y se detiene, que es justo lo que necesita un replay.

**¿Esto escalaría?**
Sí. El mismo código Spark/Delta escala horizontalmente. En Free Edition estamos limitados por volumen/cuota, pero la arquitectura es la de producción; bastaría cómputo mayor y una fuente en streaming.

**¿Por qué Delta y no Parquet plano, Iceberg o Hudi?**
Delta añade transaccionalidad ACID, *time travel* y evolución de esquema sobre Parquet, y unifica batch y streaming; es nativo en Databricks. Iceberg y Hudi son formatos abiertos de tabla con objetivos similares; los conocemos como alternativas válidas.

**¿Por qué Databricks y no EMR/Dataproc, Snowflake o BigQuery?**
Databricks es un lakehouse unificado (Spark + Delta + MLflow + SQL + gobernanza con Unity Catalog) en una sola plataforma, con edición gratuita para uso académico. Las demás son adecuadas para capas específicas: warehouse analítico (Snowflake/BigQuery/Redshift), consulta SQL sobre S3 (Athena), Spark gestionado (EMR/Dataproc).

**¿Por qué Medallion (Bronze/Silver/Gold)?**
Refina los datos por etapas garantizando calidad y linaje: Bronze preserva la fuente cruda (reprocesable), Silver limpia y tipa, Gold construye la matriz de features por sesión. Permite volver atrás sin re-descargar de Kaggle.

**¿ETL o ELT?**
ELT: cargamos crudo en Bronze (Delta) y transformamos dentro del lakehouse con Spark, aprovechando el cómputo distribuido sobre el almacenamiento barato, en lugar de transformar antes de cargar.

---

*Nota: las capacidades y límites de Databricks Free Edition fueron verificados contra la documentación oficial vigente; conviene reconfirmarlos cerca de la fecha de entrega por si cambian.*
