# Pack de contexto — Grandes Datos

> Destilación propia (no copia de las diapositivas) del material del curso de **Almacenamiento y Procesamiento de Grandes Volúmenes de Datos / Ingeniería de Datos** (SI7006 · SI6003, Universidad EAFIT, 2026), profesores del área (la sesión de Kafka la dicta **Edwin Montoya**). El curso son 5 sesiones que recorren el ciclo de vida del dato: arquitecturas modernas (DWH→Lakehouse), almacenamiento y formatos columnares, procesamiento batch con Spark, y streaming (Kafka/Kinesis/Flink/Structured Streaming). Este pack prioriza lo que el profe enfatizó y lo que se evalúa, y lo ata a nuestro pipeline (Medallion + Delta sobre Databricks, replay estilo Kappa, Spark SQL, Power BI).
>
> **Relación con el doc 02:** este pack es un documento propio y **distinto** de `02_arquitectura_bigdata_y_databricks.md`. El doc 02 = nuestra arquitectura concreta + reglas de cuota de Databricks Free + Q&A de defensa. Este pack = **qué se enseñó en clase**, el vocabulario/énfasis del profe y el mapeo a nuestro proyecto. Si hay dato de plataforma/cuota, vive en el doc 02, no aquí.

---

## 1. Técnicas y conceptos enseñados

El hilo conductor del curso es **"elegir y justificar la arquitectura/herramienta correcta según el problema"** (SLA, volumen, costo, gobernanza), no "meter toda la tecnología posible". Cada salto tecnológico se enseña como respuesta a una limitación concreta del paradigma anterior.

**1.1 Arquitecturas modernas de datos (Unidad 1)**
- **Las 3 V que rompen el modelo tradicional:** Volumen, Velocidad, Variedad (los RDBMS no escalan horizontalmente sin costo prohibitivo). En Spark se amplían a 5 V (+ Veracidad, Valor).
- **Evolución histórica:** DBMS/OLTP (schema-on-write) → DWH/OLAP (modelado dimensional, cubos) → Data Lake (HDFS→S3/GCS, schema-on-read, barato) → **Lakehouse** (Delta/Iceberg/Hudi: flexibilidad del lake + ACID del warehouse).
- **DWH vs Data Lake vs Lakehouse:** el Lakehouse unifica BI + ML + streaming en un solo sistema con gobernanza (catálogo + ACID), evitando el "data swamp" del lake puro.
- **Ciclo de vida del dato:** Fuentes → Ingesta (batch/streaming) → Almacenamiento (lake/lakehouse) → Procesamiento (Spark, SQL, ETL/ELT) → Consumo (dashboards, ML, APIs).
- **Batch vs Streaming:** batch = volúmenes acumulados, latencia tolerable (reportes, entrenamiento ML); streaming = eventos en tiempo real, baja latencia (fraude, IoT). "La elección depende del SLA del negocio."
- **Lambda vs Kappa:** Lambda = capa batch + capa speed (streaming) + servicio en paralelo (tolerante a fallos pero doble lógica y compleja). **Kappa = una sola tubería de streaming; el log (Kafka) es la fuente de verdad y el reprocesamiento histórico se hace por replay.** Kappa es ideal cuando el streaming puede reprocesar el histórico.
- **Arquitectura Medallion (Bronze/Silver/Gold):** Bronze = crudo inmutable tal como llega (fuente de verdad reprocesable); Silver = limpio, deduplicado, esquema validado, joins; Gold = agregaciones de negocio, KPIs y features para ML/BI. *Ejemplo retail (de la propia clase): Bronze = clics crudos → Silver = sesiones limpias → Gold = tasa de conversión por campaña/día.*
- **Data Platform moderna:** storage desacoplado (object storage) + compute elástico + catálogo/linaje + gobierno/seguridad + observabilidad + motores múltiples sobre el mismo storage.
- **Multi-cloud:** se justifica solo cuando el costo de dependencia (vendor lock-in) supera el costo de complejidad operacional; ojo con egress entre nubes.

**1.2 Almacenamiento y formatos columnares (Unidad 2)**
- **SQL vs NoSQL en analítica:** SQL relacional (schema-on-write, ACID, DWH/BI) vs NoSQL (documentos/clave-valor/grafos, schema-on-read, escala horizontal, ingesta de logs/IoT). Coexisten con roles distintos. *Error común: usar Mongo para analítica pesada — los motores columnares SQL ganan en agregaciones masivas.*
- **OLTP vs OLAP:** transaccional (operaciones cortas, normalizado, ACID) vs analítico (consultas complejas, desnormalizado, throughput de escaneo). El diseño físico se optimiza para el destino analítico.
- **Formatos: CSV vs Parquet vs ORC.** Row-based vs **columnar**. Por qué columnar gana en analítica: **column pushdown** (solo lee columnas del SELECT), **mayor compresión** (valores similares en bloque), **vectorized execution**, **predicate pushdown** (estadísticas min/max evitan leer bloques). *Un CSV de 10 GB → 1-2 GB en Parquet+Snappy, consultas 5-10× más rápidas.*
- **Optimización física:** **particionamiento** (subdirectorios `year=2024/month=03/...` → *partition pruning*); **compresión** (Snappy=velocidad, GZIP=ratio, ZSTD=balance); **ordenamiento/clustering** (`ZORDER`); **estadísticas de columna** (min/max/nulls/distinct por row group); **caché** (Delta Cache, result cache). Regla de tamaño de archivo: **128 MB – 1 GB por partición**. Anti-patrón crítico: **particionar por alta cardinalidad** (UUID, timestamp exacto) → millones de micro-archivos, metadata overhead.
- **Delta Lake:** capa de tabla open-source sobre Parquet con `_delta_log/`. ACID, **time travel** (`VERSION/TIMESTAMP AS OF`), **schema enforcement** + **schema evolution** (`mergeSchema`), **MERGE/upsert**, **OPTIMIZE + ZORDER**. Unifica batch y streaming.
- **Apache Iceberg:** formato de tabla abierto (origen Netflix). **Particionamiento oculto**, evolución de partición sin reescribir, time travel, y **verdadera interoperabilidad multi-motor** (Spark, Flink, Trino, Athena, Snowflake) — ventaja clave frente a Delta en entornos multi-nube.
- **Delta vs Iceberg:** Delta brilla en ecosistema Databricks, CDC y streaming unificado; Iceberg en multi-motor/multi-nube/gobernanza abierta (a costa de catálogo más complejo).

**1.3 Sistemas distribuidos y procesamiento batch con Spark (Unidades 3 y fundamentos)**
- **Fundamentos distribuidos:** un sistema distribuido se presenta como uno solo coherente. **5 pilares:** consistencia/consenso, particionamiento, replicación, transacciones, tolerancia a fallos. Servicios base de coordinación (elección de líder, sincronización) → **ZooKeeper**. **Teorema CAP (Brewer):** con tolerancia a particiones asumida, se elige entre Consistencia y Disponibilidad.
- **Apache Spark — motor in-memory** (hasta ~100× MapReduce), propósito general (SQL, streaming, ML, grafos), APIs multilenguaje.
- **Arquitectura:** Driver (coordina) + Cluster Manager (YARN/Mesos/K8s) + Worker Nodes + Executors. `SparkContext` como punto de entrada.
- **Modelo de ejecución:** **DAG** + **lazy evaluation**. **Transformaciones (lazy)** vs **acciones (eager)**: nada se ejecuta hasta una acción → Catalyst optimiza el plan completo.
- **Narrow vs Wide transformations:** narrow (sin shuffle, ej. `map`/`filter`) vs wide (con **shuffle** por red, ej. `groupByKey`/`join`). El **shuffle es la operación más costosa** (disco+red); minimizarlo es clave. *Preferir `reduceByKey` sobre `groupByKey` (pre-agrega antes del shuffle).*
- **Particionamiento y serialización:** HashPartitioner (default) / RangePartitioner; nº óptimo de particiones ≈ 2-4× cores; serialización Kryo (más rápida/compacta que Java).
- **RDD vs DataFrame:** DataFrame (esquema tipado + **Catalyst** + **Tungsten**) es la opción recomendada; RDD para control fino o datos no estructurados.
- **Catalyst Optimizer:** Análisis → Optimización lógica (**predicate pushdown**, **column pruning**, constant folding) → Planificación física (CBO, elección de join) → **Code generation** (Tungsten, whole-stage codegen a bytecode JVM).
- **Estrategias de join:** **Broadcast Hash Join** (un lado pequeño <10 MB, sin shuffle, el más rápido) / Shuffle Hash Join (medianos) / **Sort Merge Join** (ambos grandes, default).
- **Módulos:** Spark SQL (consultas + ETL con Catalyst, compatible Hive), **Structured Streaming** (flujo como tabla infinita, misma API que batch), MLlib (ML distribuido), GraphX.
- **ETL vs ELT:** ELT carga crudo y transforma dentro del lakehouse (aprovecha compute distribuido sobre storage barato).

**1.4 Streaming en tiempo real (Unidad 4)**
- **Mensajería / MOM** (Message-Oriented Middleware): desacopla productor de consumidor (store-and-forward, débilmente acoplado). Modelos: **colas punto-a-punto (FIFO, 1:1)** vs **pub/sub (tópicos, 1:N)**. Estándares: JMS, AMQP (RabbitMQ), MQTT (IoT), STOMP/XMPP.
- **Semánticas de entrega:** **at-most-once** (rápido, puede perder), **at-least-once** (garantiza entrega, puede duplicar → exige idempotencia), **exactly-once** (sin pérdida ni duplicado; esencial en finanzas/ML/auditoría).
- **Apache Kafka:** Producer / Topic / Consumer / Broker / coordinación (**ZooKeeper o KRaft/Raft**). **Tópicos particionados:** orden estricto (FIFO) **por partición**, no global; **offset** secuencial; escritura append-only. **Consumer groups:** cada grupo recibe el tópico completo; dentro del grupo, **N particiones = N consumidores activos máximos**. Asignación por round-robin / **key (hash → misma partición = orden por entidad)** / explícita. **Replicación líder + ISR** (factor N tolera N-1 caídas). **acks** = 0/1/all (velocidad vs garantía). Broker discovery vía bootstrap server.
- **AWS Kinesis** (alternativa gestionada a Kafka): **Kinesis Data Streams** (shards, retención 24 h–7 días, replay, registros ordenados por shard, escalado manual reshard/merge) vs **Kinesis Data Firehose** (totalmente gestionado, near-real-time ~60 s, entrega a S3/Redshift/ES/Splunk, conversión JSON→Parquet, sin almacenamiento). Productores: SDK (`PutRecord/PutRecords`), KPL, Agent. Consumidores: KCL (checkpoints + coordinación en DynamoDB), Lambda. Réplica en 3 AZ.
- **Procesamiento de flujos:** **ventanas** (tumbling = fija sin solape; sliding = solapada; session = por inactividad; global) con Window Assigner/Function/Trigger/Evictor/**Allowed Lateness**; **estado** (memoria entre eventos, backend RocksDB en Flink, CEP, perfiles en vivo); **watermarks** (marcan el progreso del tiempo de evento → cuándo cerrar una ventana y qué hacer con datos tardíos).
- **Flink vs Spark Structured Streaming:** **Flink** = record-at-a-time, latencia <100 ms (sub-ms), exactly-once nativo. **Spark Structured Streaming** = micro-batches (~100 ms), exactly-once vía **checkpointing + WAL**, reutiliza el motor Spark SQL (curva de adopción menor). El modo continuo de Spark da ~1 ms pero solo at-least-once.
- **Tríada de pipeline moderno:** **Kafka (ingesta) + Flink/Spark (procesamiento, estado, exactly-once) + Delta Lake (almacenamiento ACID)**, con observabilidad (Prometheus/Grafana) y orquestación (Airflow).

---

## 2. Vocabulario y énfasis del profe (qué valora, qué penaliza)

**Frases-ancla del material:**
- *"La evolución fue necesaria, no opcional"* / *"No existe una arquitectura universal: la elección depende del SLA, el volumen y la madurez del equipo."*
- *"El diseño de arquitectura consiste en elegir las herramientas correctas según latencia, volumen, costo y gobernanza."* (criterio rector — ver doc 02).
- *"Diseñar el particionamiento según los patrones de consulta reales, no los de ingesta."*
- *"Medir siempre: tamaño en disco, particiones escaneadas, tiempo de ejecución."*
- *"Multi-cloud se justifica cuando el costo de dependencia supera el costo de complejidad operacional."*
- Ejercicio de clase: replicar la arquitectura de una plataforma real y **"vender la idea"** como equipo de ingeniería a un cliente/directivo → el profe valora **comunicar y defender decisiones**, no solo dibujar diagramas.

**Qué VALORA (sube nota):**
- **Justificar cada decisión de arquitectura** con el *por qué*: ¿por qué object storage?, ¿por qué Lakehouse y no DWH+Lake?, ¿batch/streaming/híbrida según SLA?, ¿Delta o Iceberg?, ¿Databricks o Snowflake?
- **Diseño por capas con responsabilidades claras** (ingesta → almacenamiento → procesamiento → consumo) y el patrón Medallion como organización.
- **Optimización medida** (no asumida): formato columnar, particionamiento por patrón de consulta, tamaño de archivo, pruning, ZORDER, estadísticas.
- **Gobernanza desde el día uno** (catálogo, linaje, schema enforcement) para evitar el "data swamp".
- **Reconocer alternativas del ecosistema** aunque no se implementen (amplitud + criterio).
- **Pipeline end-to-end reproducible** (de la ingesta al consumo).

**Qué PENALIZA:**
- **CSV para analítica pesada** (sin compresión, sin column pushdown, sin estadísticas → imposible escalar).
- **Particionar por alta cardinalidad** (device_id, UUID, timestamp exacto) → millones de micro-archivos, metadata overhead.
- **Confundir Data Lake con Lakehouse** (un lake sin formato de tabla, catálogo y gobierno es un data swamp).
- **No gestionar metadatos/catálogo** → el motor no optimiza, el pruning falla.
- **Shuffles innecesarios** en Spark (`groupByKey` donde cabe `reduceByKey`; joins sin broadcast cuando un lado es pequeño).
- **"Meter toda la tecnología posible"** sin justificar por qué cada herramienta corresponde a cada capa.
- En streaming: ignorar la **semántica de entrega** y los **datos tardíos/watermarks** cuando la decisión los exige.

---

## 3. Herramientas / librerías vistas en clase

- **Plataformas:** **Databricks** (Lakehouse: Spark + Delta + Unity Catalog + SQL; usada en el laboratorio guiado), **Snowflake** (DWH cloud SQL, external stages, Query Profile), Google Colab.
- **Cloud:** **AWS** (S3, Redshift, EMR, Glue, Athena, Kinesis, SQS, Lambda, DynamoDB, CloudWatch) y **GCP** (GCS, BigQuery, Dataproc, Dataflow/Beam, Pub/Sub, Dataplex). Crédito académico AWS Academy / GCP Education.
- **Procesamiento:** **Apache Spark / PySpark** (`spark.read`, `df.write.format("delta")`, `partitionBy`, `OPTIMIZE ... ZORDER`, `broadcast()`), Spark SQL, Structured Streaming; conceptualmente MapReduce/Hadoop, Trino/Presto, Hive.
- **Formatos / tablas:** Parquet, ORC; **Delta Lake**, Apache Iceberg, Apache Hudi.
- **Streaming:** **Apache Kafka** (KRaft/ZooKeeper, consumer groups, acks), **AWS Kinesis** (Data Streams, Firehose, KPL/KCL/Agent), **Apache Flink** (Managed Apache Flink en AWS), Kafka Streams.
- **Orquestación / observabilidad / catálogo:** Apache Airflow, Prometheus + Grafana; Unity Catalog, AWS Glue Data Catalog, Hive Metastore.
- **Comandos/optimizaciones clave de Databricks:** conversión CSV→Parquet→Delta, `OPTIMIZE`, `ZORDER BY`, time travel (`VERSION AS OF`), `MERGE`, checkpoints + `Trigger.AvailableNow()` (detalle de uso/cuota en doc 02).

---

## 4. Expectativas de evaluación (qué espera ver en el PI)

El **Proyecto Integrador vale 35 %** de la materia (resto del curso: 3 Trabajos de 15 % c/u y 2 Exámenes 20 %). El aporte de Grandes Datos al PI es la **arquitectura, el pipeline y el despliegue**.

Lo que el material deja claro que se espera demostrar:
1. **Arquitectura justificada por capas** — cada componente con su *por qué* (latencia/volumen/costo/gobernanza), encuadrada en batch/streaming/Lambda/Kappa.
2. **Pipeline end-to-end funcional** — de la ingesta al consumo, con el patrón Medallion (Bronze/Silver/Gold) sobre formato de tabla (Delta/Iceberg).
3. **Optimización medida** — formato columnar, particionamiento según consulta, tamaño de archivo, pruning/ZORDER, estadísticas; reportar métricas (tamaño en disco, particiones escaneadas, tiempo).
4. **Gobernanza y calidad** — catálogo, linaje, schema enforcement, separación raw/curado/consumo.
5. **Capacidad de defender decisiones** — saber responder el *por qué* de cada elección y reconocer alternativas (el ejercicio "vender la arquitectura" es señal del estilo de evaluación). Las preguntas de defensa probables y las reglas de cuota de Databricks Free están en el **doc 02**.

---

## 5. Mapeo "qué se enseñó → dónde aparece en nuestro proyecto"

| Lo que se enseñó | Dónde aparece en nuestro PI |
|---|---|
| **Lakehouse + Medallion (Bronze/Silver/Gold)** | Nuestro pipeline implementado: Bronze (clickstream REES46 crudo, inmutable), Silver (eventos limpios/tipados, sesiones), **Gold (matriz de features por sesión + agregados para BI)**. Es exactamente el ejemplo retail de la clase (clics → sesiones → conversión). |
| **Delta Lake (ACID, time travel, schema, OPTIMIZE/ZORDER)** | Las tres capas son tablas **Delta**; permite reprocesar sin re-descargar de Kaggle, versionar y unificar batch+streaming. |
| **Formatos columnares + particionamiento por patrón de consulta** | Gold/Silver en Parquet por debajo de Delta; particionar por fecha/categoría (no por `user_id`/`session` de alta cardinalidad — anti-patrón explícito de la clase); tamaño de archivo 128 MB–1 GB; ZORDER por columnas de filtro. |
| **Lambda vs Kappa + el log como fuente de verdad** | Adoptamos **diseño estilo Kappa**: tratamos los CSV históricos como un log reproducible y los procesamos con la *misma* ruta de Structured Streaming que consumiría Kafka/Kinesis en producción. Detalle y defensa en doc 02. |
| **Structured Streaming + exactly-once (checkpointing/WAL) + `Trigger.AvailableNow()`** | El "replay de streaming" sobre Bronze con Auto Loader + checkpoint, estilo Kappa, da sustancia a la Unidad 4 sin inventar un broker. (Reglas de cuota/trigger en doc 02.) |
| **Kafka / Kinesis / Flink** | Arquitectura de referencia (no implementada); citarlos con su rol demuestra amplitud sin reventar el alcance. Encuadre y tabla por capa en el doc 02. |
| **Spark / PySpark / Catalyst / shuffle / joins** | Toda la transformación Bronze→Silver→Gold corre en **PySpark**; cuidamos shuffles (preferir `reduceByKey`/agregaciones, broadcast del lado pequeño). El **join en Gold** que el equipo marcó por posible sesgo se revisa con criterio de estrategia de join. |
| **Spark SQL** | Consultas analíticas sobre Gold (cierra el ciclo batch antes del scoring y del tablero). |
| **ETL vs ELT** | **ELT:** cargamos crudo en Bronze (Delta) y transformamos dentro del lakehouse con Spark, aprovechando compute distribuido sobre storage barato. |
| **Optimización medida (pruning, estadísticas, métricas)** | Sobre 14 GB: desarrollar en muestra, leer capas materializadas, medir particiones escaneadas/tiempo (alineado con las reglas de cuota del doc 02). |
| **Gobernanza / catálogo / linaje** | Unity Catalog + Volumes; separación raw (Bronze) / curado (Silver) / consumo (Gold); schema enforcement evita corrupción. |
| **Consumo: warehouse / BI** | **Power BI** conectado a la **Gold agregada** (no a 69M filas crudas) — la capa de consumo del ciclo de vida enseñado. |
| **Teorema CAP / semánticas de entrega / watermarks** | Conceptos para la defensa de la arquitectura de referencia (por qué streaming real exige exactly-once y manejo de datos tardíos); no se implementan en el alcance académico. |

---

## 6. Conceptos/citas que conviene nombrar en la defensa

Conceptos del curso que, dichos en la sustentación, demuestran que aplicamos el estándar del profe:

1. **"No existe una arquitectura universal; se elige por SLA, volumen, costo, gobernanza y madurez del equipo."** — Marco para justificar por qué, con un dataset **histórico**, lo correcto es batch + replay estilo Kappa, no streaming real.
2. **Kappa: "el log es la fuente de verdad; el reprocesamiento histórico se hace por replay."** — Defiende tratar los CSV como un log reproducido con la misma ruta de Structured Streaming.
3. **Lakehouse = "un solo sistema para BI y ML con ACID sobre object storage"** + **Medallion (Bronze/Silver/Gold)** como organización de las transformaciones. — Núcleo de nuestra arquitectura.
4. **"Por qué columnar gana en analítica": column pushdown + compresión + predicate pushdown.** — Justifica Parquet/Delta sobre CSV.
5. **"Particionar según los patrones de consulta reales, no los de ingesta; nunca por alta cardinalidad."** — Defiende nuestra estrategia de particionamiento y evita el anti-patrón que el profe penaliza.
6. **Delta: ACID + time travel + schema enforcement/evolution + OPTIMIZE/ZORDER.** — Concepto a nombrar; la justificación completa (Delta vs Parquet plano / Iceberg / Hudi) vive en el doc 02 (Q&A).
7. **Spark: lazy evaluation + Catalyst + "el shuffle es la operación más costosa".** — Muestra que entendemos el motor (preferir `reduceByKey`, broadcast joins, minimizar shuffle).
8. **ELT sobre el lakehouse** (transformar dentro, no antes de cargar). — Justifica el flujo Bronze→Silver→Gold.
9. **Tríada de streaming Kafka + Flink/Spark + Delta y semánticas exactly-once / at-least-once.** — Para describir la arquitectura de referencia de producción y por qué exactly-once importa en decisiones críticas.
10. **CAP (Brewer): con particiones, se elige entre consistencia y disponibilidad.** — Cultura distribuida para responder preguntas de fondo.
11. **"Medir siempre: tamaño en disco, particiones escaneadas, tiempo de ejecución."** — Cierra cualquier afirmación de optimización con evidencia, no con intuición.

> **Cierre defendible (estilo del profe):** "No elegimos tecnología por moda: para un dataset histórico de 14 GB, la arquitectura correcta es un Lakehouse con Medallion en Delta, ELT en Spark y un replay estilo Kappa que comparte la ruta de código del streaming de producción (Kafka/Kinesis→Flink); cada decisión —object storage, formato columnar, particionamiento, Delta— está justificada por SLA, costo y patrón de consulta, y medida, no asumida."
