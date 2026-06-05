# 00 — Estado del Proyecto · PI Grupo 8 (documento maestro)

**Última actualización:** jueves 4 de junio de 2026
**Entrega de productos:** lunes 8 de junio · **Exposición:** martes 9 de junio (5–9 pm)
**Equipo:** Kelly, Sara, Heider y Yeison
**Repositorio:** `HeiderZapata/proyecto-integrador-g8` · este archivo vive en `docs/`

> **Este es el documento maestro y vivo del proyecto.** Da el contexto general y refleja el estado de avance. Manténganlo actualizado —sobre todo la Parte A (Orientación y estado)— a medida que se completan tareas. Es el primer archivo que cualquier integrante (o su IA) debe leer para entrar en contexto.
>
> **Orden del documento (reestructurado el 3-jun):** está dividido en tres partes para no mezclar planos. **Parte A** = orientación y estado (lo primero que se lee). **Parte B** = el *qué* (contenido estable del proyecto). **Parte C** = el *cómo* (forma de trabajo, repo, plan). Antes, la tabla de estado mezclaba planeación y ejecución sin distinguir fase; ahora se separan explícitamente (Fase 3 vs Fase 4).

---

> ## ✔ SEGURIDAD — RESUELTO (4-jun)
>
> La credencial de Kaggle ya NO está hardcodeada (el notebook de ingesta lee la llave desde el Volume; el `.gitignore` cubre `kaggle.json`) y **Heider recibió la instrucción de expirar/rotar la llave expuesta desde la cuenta de Kaggle**, que es lo que la neutraliza (la llave había quedado en el historial de Git y se daba por comprometida). Con eso, el riesgo se cierra. *Pendiente menor de confirmación: que Heider dé el "hecho" explícito de que la llave quedó expirada; mientras tanto, ninguna llave válida vive en el repo ni en su historial.*

---

# PARTE A — Orientación y estado

## 1. El proyecto en una frase

Optimización de la conversión en e-commerce mediante modelado de propensión de compra, sobre el dataset abierto REES46 (clickstream multi-categoría, Oct–Nov 2019, ~14.5 GB).

**Pregunta de negocio (alcance completo, cuatro verbos):** ¿Dónde y por qué se nos escapan las compras, qué tipos de visitante las explican, y dónde —y cómo lo mediríamos— conviene concentrar el esfuerzo para recuperar conversión?

**Pregunta de Oro (afilada para el tablero):** ¿Dónde se concentra la fuga de conversión y qué segmento de visitantes representa la mayor oportunidad de recuperarla?

**Frase de ascensor:** La tienda pierde 98 de cada 100 visitas y 4 de cada 10 carritos. No construimos un modelo para adivinar quién compra; construimos un sistema para entender dónde y por qué se fuga la conversión, qué visitantes la explican, y dónde —y con qué experimento— conviene intervenir. La predicción es el instrumento; la decisión de negocio es el producto.

---

## 2. Estado de avance

Estados: **Hecho · Revisión (v1 existe, requiere ajustes) · En curso · Pendiente · URGENTE**.
*La tabla se separa en tres bloques: fundamentos ya cerrados, planeación (Fase 3, en cierre) y ejecución (Fase 4, pendiente). Así no se confunde lo decidido con lo que falta decidir, ni la planeación con la construcción.*

### 2.1 Fundamentos (cerrados)

| Frente / Tarea | Responsable | Estado | Notas |
|---|---|---|---|
| Diseño conceptual (modelo, uplift, alcance) | Equipo | Hecho | Ver §5 |
| Pregunta de negocio + Pregunta de Oro | Equipo | Hecho | Ver §1 |
| Arquitectura Big Data (2 capas) | Heider/Yeison | Hecho | Doc 02 |
| Sincronizar particionamiento por capa (doc 02) | Heider/Yeison | Hecho | doc 02 §3; cierra el GAP del feedback §5 |
| Corregir propuesta del proyecto | Yeison | Hecho | `03_propuesta_corregida.md` (+ `.docx`). Ver §5.4 |
| Crear Proyecto de Claude | Yeison | Hecho | Operando en él |
| Adjuntar feedback de exposiciones pregrado | Yeison | Hecho | `08_feedback_exposiciones_pregrado.md` |
| Pack contexto — Grandes Datos | — | Hecho | `06_contexto_grandes_datos.md` (≠ doc 02) |
| Pack contexto — Visualización | — | Hecho | `07_contexto_visualizacion.md` |
| Pack contexto — Aprendizaje Automático | Yeison (Claude Code) | Hecho | `05_contexto_aprendizaje_automatico.md` |

### 2.2 Planeación — Fase 3 (EN CIERRE)

| Frente / Tarea | Responsable | Estado | Notas |
|---|---|---|---|
| **Expirar la credencial de Kaggle expuesta** | Heider | **Hecho** | Notebook lee del Volume + `.gitignore` (Hecho); Heider con instrucción de expirar la llave en Kaggle → riesgo cerrado (ver banner). Falta solo su "hecho" explícito |
| Confirmar roles | Equipo | **Hecho** | **Aceptados en la reunión del 3-jun.** §11 pasó de PROPUESTO a firme |
| Congelar los tres contratos | Yeison (Gold) / Equipo | En curso | §13. **El esquema Gold se congela HOY** al cerrar la secuencia §2.3.1 (no hay consumidor en paralelo hoy); salida del modelo y datos del dashboard se enuncian para que la Gold ya nazca sirviéndolos |
| Cronograma mié 3 → mar 9 con fecha y dueño | Equipo | Ajustado (4-jun) | §15. La reunión movió el arranque: jue 4 = onboarding del equipo + Yeison propone plan a Heider |
| Ordenar el repo (estructura de código + protocolo Git + `.gitignore`) | Heider/Yeison (Claude Code) | Hecho | §12. Estructura creada, notebooks movidos a `pipeline/`, `.gitignore` en la raíz (baseline en `main`) |
| Doc de convenciones Git — `CONTRIBUTING.md` (ramas, PRs, evitar conflictos) | Heider/Yeison | Hecho | §14.1. En la raíz del repo (baseline en `main`) |
| Investigar comunidad del dataset (Kaggle) | Todos (transversal) | Pendiente | Cada dueño contrasta su fase con la comunidad REES46 |

### 2.3 Ejecución — Fase 4 (pendiente; arranca al cerrar Fase 3)

| Frente / Tarea | Responsable | Estado | Notas |
|---|---|---|---|
| Pipeline Medallion (Bronze/Silver/Gold) — **todos los ajustes de la capa** | **Heider/Yeison** | En curso | v1 corrió sobre 14 GB. Secuencia §2.3.1: **pasos 1–3 Hechos (4-jun)** — Gold v1 corregida y validada; faltan pasos 4–5. **Ruta crítica — ver §3** |
| Entrenamiento + evaluación del modelo | Sara | Pendiente | PR-AUC, calibración (no accuracy). Arranca con la Gold v1 (paso 3) |
| Clustering de visitantes | Sara | Pendiente | Con metodología propia; el **cruce clasificador × clustering** es el insight |
| Tablero Power BI ejecutivo | Kelly | Pendiente | Sobre la Gold **agregada** (no 69M filas). Arranca con la Gold v1 (paso 3) |
| Diseño del A/B test | Yeison | Pendiente | Cierre: medir un incentivo sobre el segmento de mayor intención |
| Rehacer Ilustración 2 (diagramas de arquitectura de datos) | Heider construye · Yeison revisa (co-responsables) | Pendiente | Reflejar frontera train/test + dos arquitecturas (doc 02 §3): **S3 + Auto Loader = referencia; Volume = implementada** (decisión firme, §7). Yeison deja el insumo listo; Heider arma los diagramas |
| Q&A de defensa por profesor | Equipo | Pendiente | Insumo: `08_feedback_exposiciones_pregrado.md` §5 |
| Documento consolidado del PI | Equipo | Pendiente | |
| Presentación (PPTX) | Equipo | Pendiente | Incluir narrativa del recorrido |

#### 2.3.1 Tratamiento de datos — secuencia ordenada (Heider/Yeison · hoy en adelante)

**Por qué este orden:** primero el *esqueleto* correcto de la Gold (grano + `join` + anti-fuga), después la *carne* (limpieza y variables informadas por el EDA). Limpiar o crear variables sobre un `join` con sesgo es retrabajo. Dos pasadas sobre la Gold, con el EDA en medio; la pasada 2 es **solo aditiva** para no romper el contrato (ver ⚠️). Es el bloque que **tú y Yeison hacen hoy** con Claude Code; Heider apoya el track paralelo (paso 6) o la Ilustración 2.

| # | Paso | Responsable | Estado | Notas |
|---|---|---|---|---|
| 1 | **Separar Medallion ↔ EDA** | Heider/Yeison (Claude Code) | **Hecho (4-jun)** | §12.3. `02_Medallion_Ecommerce.ipynb` queda solo Medallion (funnel-EDA retirado); `EDA.ipynb` movido a `analysis/`, retitulado a "EDA del PI" y con TODO de re-fuente a Silver/Gold. Commit `9f09a8b` |
| 2 | **Validar la estructura de la Gold** (`join`/sesgo, reconstrucción de sesión, grano, corte anti-fuga) | Heider/Yeison (Sara consultada por el sesgo) | **Hecho (4-jun)** | Diagnóstico (`02b_diagnostico_gold_join.ipynb`) confirmó: sesgo de selección real pero pequeño (descartes 9× positivos, ~0.1%), grano roto (`user_session` con >1 `user_id`) y gotcha del frame RANGE. Commit `27b66b6` |
| 3 | **Complementar Medallion — pasada 1** (con el EDA *actual*): limpieza completa, tipado, dedup, nulos/outliers de precio, sesiones-bot; construir/eliminar/transformar variables que el EDA actual ya justifique | Heider/Yeison | **Hecho (4-jun)** | Gold corregida (corte determinista, LEFT join sin sesgo, grano 1 fila = 1 sesión + flag `sin_navegacion_previa`) + Silver con `dropDuplicates` y `price>0`. **Gold v1 validada: 22.99M sesiones, tasa etiqueta 0.0610, 0 duplicados.** Pendiente (sin umbral acordado): outliers de precio y sesiones-bot. Commit `3f66e6a`. *El contrato se congela tras la pasada 2 (paso 5)* |
| 4 | **Organizar el `EDA.ipynb` oficial** sobre la Gold v1 (reubicar en `analysis/`, conectar a Silver/Gold, retitular, revisar completitud vs. Pregunta de Oro) | Heider/Yeison (Claude Code) → alinear con Kelly | **Hecho (4-jun)** | §12.3. **EDA re-fuenteado a Silver/Gold v1 (capa de agregados Spark→pandas; ningún gráfico re-escanea Silver), corriendo entero en Databricks.** Se añadieron **§5 (perfilado Gold v1)** y **§6 (temporal/Black Friday, curva de intención, tipología de visitantes)** + §7 diagnóstico. Insumo de Kelly (tablero) y Sara (features). **Ver §17 para los hallazgos y decisiones que salieron.** Andamiaje local en `notebooks/analysis/_build/` (gitignored). |
| 5 | **Pasada 2 sobre la Gold** (según el EDA): correlación/multicolinealidad, variables para ML, métricas/tablas agregadas para el tablero, **calidad de datos** | Heider/Yeison | Pendiente | Cierra "Gold completa y robusta". **Informado por el EDA (§17):** (a) **cuarentenar la ventana corrupta 15–17 nov** (etiqueta de noviembre rota); (b) auditar categóricas/`"Unknown"`, outliers de precio y sesiones-bot; (c) **enriquecer features** (señal lineal débil/negativa; multicolinealidad `total_views`≈`distinct_products` 0.92). **Al terminar se congela el contrato del esquema Gold COMPLETO (§13).** |
| 6 | **Track paralelo/posterior** (no gatea la correctitud de la Gold): Bronze→`readStream` (Kappa) · Spark SQL · MLflow setup · scoring batch | Heider (readStream/SQL) · Heider/Sara (MLflow/scoring) | Pendiente | `readStream` con checkpoint, no en bucle (doc 02 §4). **Scoring batch es de los últimos pasos: requiere modelo entrenado** |

> **Nota de congelación (4-jun, actualizada):** el **paso 4 (EDA oficial) quedó Hecho** y destapó hallazgos que **deben entrar a la pasada 2 antes de congelar** (calidad de datos de noviembre, features con señal débil — ver §17). Por eso el esquema Gold **NO se congeló hoy**: la pasada 2 (paso 5) se hace mañana temprano, junto con la decisión de split con Sara (§17), y **ahí** se congela el contrato (§13). La regla "aditivo, nunca renombra ni elimina" sigue como red de seguridad para no romper a quien ya consuma la Gold v1.

---

## 3. Ruta crítica (qué desbloquea qué)

La mayor parte del trabajo de Fase 4 depende de unas pocas compuertas. Atacarlas en orden es lo que permite que cuatro personas trabajen en paralelo:

```
Credencial rotada + 3 contratos congelados (§13)
            │  (desbloquea el trabajo en paralelo)
            ▼
Gold VALIDADA  ──►  esquema Gold CONGELADO   ← COMPUERTA PRINCIPAL
 (join sin sesgo +        │
  particionamiento)       ├──► Features + entrenamiento (Sara)
            │             ├──► Tablero Power BI (Kelly)
            │             └──► Selección de segmento para el A/B (Yeison)
            ▼
Gold AGREGADA exportada (doc 02 §4.7: una sola persona materializa las capas
pesadas; la Gold pequeña se exporta para que el resto trabaje sin re-correr 14 GB)
            ▼
Salida del modelo  ──►  enriquece tablero + define el segmento del A/B
```

**Lectura:** hasta que la Gold no esté validada y su esquema congelado, lo de aguas abajo (features, tablero, A/B) se construye sobre arena. Por eso los ajustes del pipeline (`join` de Gold + `readStream` en Bronze) **no son detalle: son ruta crítica.** La tabla Gold —que alimenta tanto el EDA-entregable como los modelos— es **el deliverable-compuerta del proyecto**: **Heider y Yeison son co-responsables** de dejarla completa y robusta antes de abrir los frentes de aguas abajo.

---

## 4. Fase actual y Definición de Hecho de Fase 3

**Estamos cerrando la Fase 3** (pulir la estrategia y el paso a paso definitivo). La reunión de equipo no es el inicio de la Fase 4: es el acto que **cierra la Fase 3**. La Fase 4 (abrir chats de ejecución por frente) arranca recién después.

**Definición de Hecho de Fase 3** — cruzamos a Fase 4 cuando se cumplan los cinco:

1. ~~**Roles confirmados**~~ — **HECHO** (aceptados en la reunión del 3-jun; §11 firme).
2. **Cronograma** mié 3 → mar 9 con fecha y dueño por tarea (§15) — ajustado el 4-jun (la reunión movió el arranque).
3. **Tres contratos congelados** (§13): esquema de la Gold (lo congela Yeison), salida del modelo, datos que consume el dashboard. *(En curso — compuerta para abrir los frentes de aguas abajo.)*
4. **Repo ordenado** (§12): estructura de código definida (Hecho), protocolo Git operativo (Hecho), `.gitignore` (Hecho), **credencial de Kaggle neutralizada** — **HECHO** (llave en proceso de expiración por Heider; ninguna llave válida vive en el repo ni en su historial).
5. **Kickoff por frente listo** — un prompt/encuadre por frente, para que los chats de Fase 4 abran limpios y heredando el knowledge.

**Estado:** quedan vivos los puntos 3 (contratos, sobre todo el esquema Gold) y 5 (kickoff). El proyecto ya está pisando Fase 4 en la práctica: Yeison y Heider arrancan la capa Gold y el EDA en cuanto el equipo termine el onboarding de hoy.

---

# PARTE B — El qué (contenido estable)

## 5. Decisión conceptual central: qué hacemos y qué NO

### 5.1 El giro respecto a la propuesta original (uplift)
La propuesta original prometía un modelo de **uplift** para targeting de incentivos. **No es estimable con estos datos**: el uplift mide el efecto causal de un tratamiento (cupón) y requiere experimento con grupo tratado y control. El dataset es observacional: **no existe variable de tratamiento**. No es un problema de modelo, sino de diseño de los datos. Coincide con el comentario del revisor.

### 5.2 Lo que SÍ hacemos
1. **Clasificador de propensión de compra (supervisado).** Unidad = sesión; etiqueta = ¿contiene `purchase`? Predice *quién* compra.
2. **Clustering de visitantes (no supervisado).** Tipos de visitante; línea paralela con metodología propia (escalado, PCA opcional, k-means, k por codo/silhouette, perfilado).
3. **Diagnóstico.** Funnel, abandono e importancia de variables: la base es `feature_importances_` / importancia por permutación (lo que enseña el curso); un **gráfico tipo SHAP** se usa solo como **capa complementaria** para leer dirección y magnitud del efecto, nunca como "lo visto en clase". El **cruce** clasificador × clustering es donde nace el insight.
4. **Cierre: diseño de un A/B test.** No estimamos causalidad; entregamos el diseño del experimento que mediría un incentivo sobre el segmento de mayor intención. Responde al revisor.

### 5.3 Lo que NO hacemos (y por qué) — es también la narrativa de la presentación
- **Uplift desde la propensión / motor prescriptivo de incentivos:** propensión ≠ uplift; seleccionaría compradores seguros; afirma causalidad no verificable.
- **Simulación bajo supuestos:** el resultado queda dentro del supuesto; atacable.
- **Recomendador:** otro problema, otros modelos; revienta el alcance.
- **Pronóstico de demanda:** otro target; la compra se decide en <2 min.

### 5.4 La corrección de la propuesta (qué cambió) — HECHO
Materializada en `03_propuesta_corregida.md` (+ `03_propuesta_corregida.docx`); el original quedó en `material_entrega/propuesta_original.docx`. Resumen:
- **Uplift reenfocado:** se reconoce que no es estimable con datos observacionales → propensión + clustering + diagnóstico + **diseño de A/B test**. Se eliminó el lenguaje de uplift causal / CATE / Qini / "conversión incremental" / "tiempo real".
- **Métodos por materia ajustados:** ensambles (logística baseline → GBM), métricas de desbalanceo (PR-AUC/F1, no accuracy), split temporal + anti-fuga, Big Data en dos capas (referencia vs. implementada, Kappa), Power BI + visuales analíticos.
- **Promesas no sostenibles reubicadas** (uplift causal, Kafka/Kinesis implementado) como arquitectura de referencia.
- **Pendientes que arrastra:** curar Problema/Impacto contra el informe final; rehacer la Ilustración 2 (§2.3).

### 5.5 Umbral con costos
Opcional, como una diapositiva. Demuestra que 0.5 es erróneo bajo desbalanceo, pero aporta poco (trivial, depende de costos inventados, redundante con la matriz deslizable del tablero). No es pieza central. *(El umbral como **política operativa** —definido por el costo del error, no por 0.5— sí es central y va en el componente de ML; lo opcional es la diapositiva de la matriz de costos.)*

---

## 6. Datos clave (EDA sobre los 14 GB)

- **Esquema:** `event_time`, `event_type` (view/cart/purchase), `product_id`, `category_id`, `category_code`, `brand`, `price`, `user_id`, `user_session`.
- **Volumen:** Oct (5.5 GB) + Nov (9 GB); 109.5M eventos → ~69.6M unidades full / **~58.6M en base limpia** (sin 15–17 nov).
- **Métricas (base limpia · EDA oficial, cuarentena 15–17 nov):** cart **3.93%**, **conversión 2.24%**, **cierre 56.9%**, **abandono 43.1%** (unidad = producto-en-sesión). *Full-data (con la ventana corrupta): abandono 51.7% / cierre 48.3% — el 15-nov inflaba el abandono +8.6pp; ver §7/§17.*
- **Modelado:** split temporal (Oct entrena, Nov prueba), corte anti-fuga (solo comportamiento previo al primer cart/purchase), métrica PR-AUC + calibración (no accuracy).

---

## 7. Arquitectura de Grandes Datos (detalle en doc 02)

- **De referencia (producción, solo enunciada):** Kafka/Kinesis → Structured Streaming/Flink → Delta sobre S3/GCS → warehouse/Athena → serving en tiempo real.
- **Implementada (Databricks Free):** ingesta batch → **replay de streaming** (Auto Loader + `Trigger.AvailableNow()` + checkpoint, estilo Kappa) → Medallion en Delta → Spark SQL → MLflow + scoring batch → Power BI.
- **Ingesta: Volume de Databricks, NO bucket S3 externo (DECISIÓN FIRME — 4-jun).** El crudo se queda en el Volume `ecommerce_raw` y Databricks ingesta desde ahí. Por qué: (1) un Volume **ya está respaldado por object storage** —leer del Volume *es* ingestar desde un almacén de objetos—, y Auto Loader (`cloudFiles`) puede apuntar al path del Volume, así que el replay de streaming/Kappa se demuestra **sin** bucket externo; (2) S3 agregaría una cuenta AWS y credenciales que gestionar (choca con *no exponer secretos* y con la reproducibilidad del repo) y las *external locations / storage credentials* son limitadas en Free Edition; (3) re-subir y re-ingestar 14 GB quema tiempo y cuota a pocos días de entregar, sin resolver ningún problema actual. S3 solo valdría la pena si herramientas **fuera** de Databricks tuvieran que leer el crudo, si hubiera un *landing zone* multi-fuente real, o si el Volume no aguantara el tamaño —nada de eso aplica aquí. **En la Ilustración 2:** S3 + Auto Loader van dibujados como la arquitectura **de referencia** productiva; el **Volume respaldado por object storage** es la **implementada**. Esta es también la respuesta de Q&A a «¿por qué no S3?».
- **Streaming liviano aprobado:** convertir Bronze a `readStream`. Cubre la Unidad 4, da sustancia a la narrativa Kappa. Correr con checkpoint, no en bucle. Nada de broker/productor/AWS.
- **Particionamiento por capa (doc 02 §3):** Bronze por fecha de evento; Silver/Gold por fecha (+ categoría); `ZORDER` por columnas selectivas; tamaños 128 MB–1 GB; nunca por alta cardinalidad. Cuida la cuota y deja visible la frontera train/test.

---

## 8. Visualización (B3)

- **Ejecutivo (Power BI, desplegado en Power BI Service, conectado a la Gold agregada):** se detalla en Fase 4, con el material de clase.
- **Analítico (notebook, Plotly/Matplotlib):** se detalla en Fase 4, con el material de clase.
- **Jurado:** Edison Valencia (BA/HPC) y Mauricio Árias (Diseño) → pesa narrativa y diseño. Ensayar defensa **filtrando en vivo**.
- **Q&A de defensa:** preparar por profesor; el feedback de pregrado ya está en `08_feedback_exposiciones_pregrado.md` (ver su §5 para acciones).

---

## 9. Plataforma y herramientas

- **Databricks Free Edition:** alcanza para el núcleo; serverless con cuota de uso justo. Reglas de cuota y limitaciones en el doc 02.
- **Power BI:** Desktop solo Windows; conectar a la Gold agregada (no a 69M filas crudas).
- **GitHub:** mejorar protocolo (ramas por frente, PRs, `.gitignore`, secretos fuera del código). Ver §12 y §14.
- **AWS ($100):** opcional (no necesario para el tablero al ir con Power BI).

---

# PARTE C — El cómo (forma de trabajo)

## 10. Flujo de trabajo y fases

**Tres lugares, tres funciones.** No confundirlos es lo que mantiene el trabajo ordenado:

- **Proyecto de Claude (chats) = pensar y decidir.** Estrategia y desarrollo *conversacional* por frente: diseñar el A/B test, decidir features, interpretar resultados, redactar el documento, afinar la narrativa del tablero, preparar el Q&A. Un chat por frente; todos comparten el mismo knowledge.
- **Claude Code (terminal/escritorio) = construir.** Lo que toca código y repo: pipeline, refactor de notebooks, entrenamiento, limpieza del repo, destilación de packs. Lee y escribe archivos reales.
- **Repo `docs/` = memoria compartida.** El contexto vive en archivos `.md`. El knowledge del Proyecto de Claude y Claude Code **no se sincronizan automáticamente**; `docs/` es el puente entre ambos (y entre los compañeros).

### Fases
- **Fase 0 — Crear el Proyecto de Claude — HECHO.**
- **Fase 1 — Destilar los packs de contexto — HECHO.** Los tres packs están en `docs/`.
- **Fase 2 — Cargar todo al Proyecto — HECHO.** Packs + feedback + docs; el Proyecto tiene contexto completo.
- **Fase 3 — Pulir la estrategia y el paso a paso definitivo — EN CURSO (cerrando).** Plan, tiempos, tareas por persona, prompts por frente, configuración del entorno (estructura del repo, contratos, protocolo Git). Cierra con la Definición de Hecho (§4).
- **Fase 4 — Abrir chats de ejecución por frente — PENDIENTE.** Pipeline, modelado, visualización, documento. Heredan el knowledge.

---

## 11. Reparto de frentes (FIRME — aceptado en la reunión del 3-jun)

- **Heider (con Yeison en la capa Medallion) — Ingeniería de Datos:** pipeline, streaming, Delta, persistencia, salud del repo.
- **Sara (Yeison acompaña) — ML/Modelado:** features, entrenamiento, evaluación, clustering.
- **Kelly (Yeison acompaña) — Visualización + narrativa:** EDA-funnel pulido, tablero, Pregunta de Oro, storytelling.
- **Yeison — Co-responsable de la tabla Gold (con Heider) + Integración + Decision Science + orquestación con Claude:** garantizar que la Gold quede completa y robusta (alimenta EDA y modelos), capa de decisión / A-B, documento, coordinación.

---

## 12. Estructura del repo y separación de código

### 12.1 Qué vive en `docs/`
Para que a los compañeros (y sus IAs) les baste con apuntar a `docs/`:

- `docs/00_estado_del_proyecto.md` — este documento maestro.
- `docs/01_configuracion_proyecto_ia.md` — instrucciones para las conversaciones de Claude dentro del Proyecto.
- `docs/02_arquitectura_bigdata_y_databricks.md` — arquitecturas, particionamiento por capa, cuota, Q&A.
- `docs/03_propuesta_corregida.md` (+ `.docx`) — propuesta corregida (la `.docx` es la versión de entrega).
- `docs/04_propuesta_texto_acordado.md` — texto fuente acordado en el chat (insumo de la corrección).
- `docs/05_contexto_aprendizaje_automatico.md` · `06_contexto_grandes_datos.md` (**≠ doc 02**) · `07_contexto_visualizacion.md` — packs de curso.
- `docs/08_feedback_exposiciones_pregrado.md` — feedback de pregrado para la defensa.
- `docs/material_entrega/` — materiales **dados** del curso: `criterios_evaluacion_pi.docx`, `formatos_y_reglas_entrega_pi.docx`, `propuesta_original.docx`.

### 12.2 Estructura de código (NUEVO — llena el hueco de Fase 3)
La estructura se define por el principio **exploración ≠ producción**:

```
notebooks/
├── exploration/   # EDA de DESCUBRIMIENTO (profiling): informa el diseño. Corre en muestra. Casi desechable.
├── pipeline/      # Ingesta + Medallion (Bronze/Silver/Gold). Determinista e idempotente.
├── analysis/      # EDA-entregable (funnel/abandono) + exploración de clustering. LEE de Silver/Gold.
└── modeling/      # features · train · eval (MLflow).
reports/
├── powerbi/       # tablero (.pbix)
└── data/          # Gold AGREGADA pequeña para Power BI (excepción del .gitignore)
```

**Archivos de gobierno en la raíz del repo:** `README.md` (puesta en marcha del entorno), `CONTRIBUTING.md` (cómo colaborar en Git, §14.1) y `.gitignore`. *(El `material_cursos/` se eliminó: los packs ya están destilados en `docs/` y el material crudo no hace falta en el repo.)*

**Regla mental:** si el código **produce** una tabla Delta que otros consumen → `pipeline/`. Si **lee** capas ya hechas para aprender o comunicar → `exploration/` o `analysis/`. El funnel "reciclado del taller 2" pertenece a `analysis/`, no al pipeline.

**Por qué separar (no es estética):** (1) **cuota** — si los gráficos vivieran dentro del pipeline, cada reconstrucción de una capa quemaría cómputo renderizando visualizaciones (doc 02 §4); (2) **anti-fuga** — el EDA de features debe respetar el corte temporal Oct/Nov; enredarlo con la ingesta facilita "espiar" datos de test.

### 12.3 Separar Medallion del EDA + dejar el EDA oficial (Heider/Yeison — tarea de §2.3)
El EDA de descubrimiento *informa* las transformaciones, pero estas se **codifican en Silver/Gold**, no se quedan en el notebook de exploración. **Co-responsables: Heider y Yeison** (es parte de dejar la tabla Gold completa y robusta). La dirección de dependencia es de una sola vía: EDA-descubrimiento → transformaciones (pipeline) → EDA-entregable (consume). Esta tabla Gold alimenta **tanto el EDA-entregable como los modelos**, y el EDA es **insumo directo de las decisiones de diseño del tablero (Kelly) y de features (Sara)** — el conocimiento del negocio que sale del EDA es crucial, no decorativo. Por eso es el paso más importante y se valida con cuidado.

**Situación a 4-jun — hay dos notebooks en juego:**
- `02_Medallion_Y_EDA_Ecommerce` (en `pipeline/`): **combina Medallion + un EDA-funnel delgado** (reciclado, superficial) — justo la conflación que evitamos.
- `EDA.ipynb` (recién subido a `pipeline/`, ubicación provisional): **es el EDA *real*** que el equipo desarrolló hace semanas para el taller de Visualización, sobre una **muestra**. Este será el **archivo oficial del EDA** del PI.

**Tarea con Claude Code (abrir ambos notebooks reales; co-responsables Heider/Yeison):**
1. **Entender** qué hay en `EDA.ipynb` (qué análisis trae, sobre qué muestra, qué supuestos).
2. **Separar bien Medallion ↔ EDA en ambas direcciones:** qué del EDA debe promoverse a la lógica determinista de Silver/Gold (limpieza, tipado, dedup, reconstrucción de sesión, features) y qué del `02_Medallion_Y_EDA` es realmente análisis y debe salir del pipeline. `EDA.ipynb` queda como el EDA oficial y **debe alimentarse de Silver/Gold**.
3. **Reubicar** `EDA.ipynb` en `notebooks/analysis/` (no en `pipeline/`).
4. **Cambiar la fuente de datos:** que lea de **Silver/Gold** (ya no del archivo de muestra del taller) y **corra en Databricks**.
5. **Retitular** el notebook: el título y la descripción aún aluden al *taller 2 de Visualización*; pasar a "EDA del PI".
6. **Revisar completitud (no es la versión definitiva):** ver si al EDA le falta algún análisis relevante para la Pregunta de Oro; y al **Medallion**, verificar que haga una **limpieza completa** (aquí es útil contrastar aportes de la comunidad REES46 en Kaggle — §2.2 transversal).

**Estrategia de cuota para este refactor (doc 02 §4):** el EDA oficial **no debe re-escanear los 14 GB**. Mientras se itera el refactor, trabajar sobre **muestra o sobre la Gold/Silver ya materializada**; una sola persona materializa las capas pesadas y exporta la **Gold agregada pequeña**, y el EDA-entregable lee de ahí. Nada de re-correr el Medallion completo solo para refrescar un gráfico; correr con checkpoint y `AvailableNow`, no en bucle. Así el nuevo `EDA.ipynb` añade valor sin quemar la cuota Free.

### 12.4 Convención de packs de contexto
- `material_cursos/<curso>/` — material crudo. **Eliminado del repo** (los packs ya están destilados); si alguien lo necesita en local, mantenerlo fuera de Git.
- `docs/0X_contexto_<curso>.md` — pack destilado (síntesis propia, no copia del material). Sí va al repo.
- Plantilla del pack: (1) técnicas/conceptos, (2) vocabulario y énfasis del profe, (3) herramientas vistas, (4) expectativas de evaluación, (5) mapeo "qué se enseñó → dónde aparece en el proyecto", (6) conceptos/citas para la defensa.

---

## 13. Contratos a congelar (compuerta de Fase 3)

Para trabajar en paralelo sin pisarse, congelar las tres interfaces. Estado: **el esquema Gold se congela HOY (4-jun) al cierre de la secuencia §2.3.1**; los otros dos se enuncian para que la Gold ya nazca sirviéndolos.

1. **Esquema de la tabla Gold** — columnas, tipos, grano (por sesión), particionamiento. *Se congela completo tras la pasada 2 (§2.3.1 paso 5), una vez validado el `join` (§3). Como Sara/Kelly no consumen hoy, se hacen las dos pasadas y se congela una sola vez.*
2. **Salida del modelo** — qué entrega el scoring (id de sesión, score/probabilidad, segmento), formato y dónde se persiste. *Lo cierra Sara al definir el modelo; se enuncia hoy.*
3. **Datos que consume el dashboard** — la Gold **agregada** que alimenta Power BI (no las 69M filas crudas). *Las tablas/métricas agregadas salen de la pasada 2 (§2.3.1 paso 5).*

> **Gold v1 — esquema actual (estado 4-jun; aún NO congelado, se cierra en el paso 5).**
> Grano: **1 fila = 1 `user_session`** · **22.99M** sesiones · tasa de etiqueta (purchase) **0.0610**.
> Columnas (núcleo v1): `user_session`, `target_purchase` (Y, 0/1), `user_id`, `total_views`, `distinct_products_viewed`, `brands_compared`, `categories_explored`, `browsing_duration_sec`, `sin_navegacion_previa` (bool).
> **Añadido en la pasada 2 (aditivo · 4-jun · commit `15dc44d`):** `session_date` (date), `session_hour` (int), `day_of_week` (int), `is_black_friday` (0/1, **solo para estratificar evaluación — no es feature**, §6.1·bis inv. 4), `label_window_corrupt` (0/1, marca la ventana 15–17 nov; **cuarentena = filtro de modelado/agregación, no se borra**). Derivadas del primer evento de la sesión (inicio, previo a cart/purchase → anti-fuga safe). Habilita cuarentena, split temporal, particionamiento por fecha y estratificación BF.
> Particionamiento: pendiente de aplicar (doc 02 §3: por fecha/categoría + `ZORDER`).
>
> **📌 Para Sara (features · §11) — set inicial de la pasada 2, A REVISAR por modelo.** La pasada 2 añade a la Gold (aditivo, anti-fuga, commit en `feat/pipeline`) un set informado por el EDA (§17.1): `categories_explored_cid` (usa `category_id` —limpio— en vez del conteo macro contaminado por `"Unknown"`), `avg_price_viewed`, `max_price_viewed`, `electronics_view_share` (señal de negocio: electrónica = 75% revenue), `is_weekend`, y tres de **decisividad/ritmo**: `revisit_intensity` (**decorrelaciona el par `total_views`≈`distinct_products_viewed`, corr 0.92**), `views_per_minute`, `avg_inter_event_sec`. **Es un punto de partida, no la palabra final:** Sara decide qué raw column del par colineal soltar y si cada modelo necesita **FE adicional** (interacciones, encoding de `category_id` —target/frequency—, codificación cíclica de `session_hour`, escalado). Todo es **anti-fuga** (solo comportamiento pre-corte) y **aditivo**: no rompe el contrato; seleccionar/ignorar es libre aguas abajo.
>
> **⚠️ Decisión pendiente para Sara (modelado) — flag `sin_navegacion_previa`.** Marca **34,144 sesiones (~0.15%)** que "abren" con `cart`/`purchase`: por el corte anti-fuga tienen **todas las features de navegación en 0** y son **mayormente positivas** (en la muestra, ~55% vs 6% global). El `inner join` viejo las descartaba en silencio (sesgo de selección); la Gold v1 las conserva con el flag. Sara debe decidir cómo tratarlas: **(A)** mantenerlas usando `sin_navegacion_previa` como variable; **(B)** excluirlas del entrenamiento (el clasificador predice desde el comportamiento *previo*, que aquí no existe); o **(C)** tratarlas como segmento aparte. Impacto chico (0.15%) pero a definir antes de entrenar. Cierra el *"Sara consultada por el sesgo"* de §2.3.1 paso 2.

---

## 14. Operación: Claude + Git

- **Proyecto de Claude** con knowledge = packs de cursos + propuesta corregida + rúbricas + esquema del dataset + convenciones del repo.
- **Claude Code** para todo lo que toque el repo (refactor, limpieza, README, **ordenar el repo según §12**, eliminar el secreto, reproducibilidad).
- Cada integrante usa su IA por frente; convergen vía los contratos (§13).
- **Git:** ramas por frente + PRs; Pull antes / Push después; `.gitignore`; secretos en variables de entorno / Databricks secrets (nunca en el código).

### 14.1 Convenciones de Git (`CONTRIBUTING.md`)
Para que cuatro personas trabajen en paralelo sin conflictos de versiones, el repo lleva un `CONTRIBUTING.md` **en la raíz** (GitHub lo muestra al abrir PRs). Contenido mínimo:
- **Una rama por frente** (p. ej. `feat/pipeline`, `feat/modelo`, `feat/viz`, `feat/doc`); nunca trabajar directo sobre `main`.
- **Convención de nombres** de ramas y de commits (mensajes claros, en presente).
- **Flujo de PR:** `pull` antes de empezar / `push` al terminar; PR a `main` con al menos una revisión antes de fusionar.
- **Evitar conflictos:** respetar la separación de carpetas (§12) —cada frente toca su carpeta—; coordinar los cambios a archivos compartidos (este doc 00, los contratos).
- **Nunca** commitear secretos ni datos pesados (`.gitignore`: `material_cursos/`, credenciales, datos crudos).

*Alternativa de nombre, si prefieren mantener el esquema numerado de `docs/`: `docs/09_convenciones_git.md`. Recomiendo `CONTRIBUTING.md` por el soporte nativo de GitHub.*

---

## 15. Plan con fechas y dueños (mié 3 → mar 9) — AJUSTADO el 4-jun

Ajustado tras la reunión del 3-jun. Respeta la ruta crítica (§3) y la regla de cuota de no correr cargas pesadas la víspera (doc 02 §4). **Cambio clave:** la reunión decidió que **jue 4 es día de onboarding** para Kelly, Sara y Heider (entienden el repo y estudian su tema); **no trabajan en sus frentes hoy**. Como nadie consume la Gold hoy, **Yeison + Claude Code dejan la Gold completa y el EDA oficial listos hoy mismo** (secuencia §2.3.1), de modo que Sara y Kelly arrancan el viernes sobre una Gold ya congelada.

| Día | Foco | Quién / qué |
|---|---|---|
| **mié 3** | Reunión — Fase 3 cerrada | **HECHO.** Roles aceptados (§11 firme), avances y pasos a seguir presentados. Heider con instrucción de expirar la credencial Kaggle. |
| **jue 4** (hoy) | Onboarding del equipo + **Gold completa y EDA oficial (HITO, hoy)** | **Kelly, Sara, Heider:** entender repo + docs y estudiar su tema (ML, Viz, ingeniería) — **no trabajan en sus frentes hoy**. **Yeison + Claude Code:** ejecutar la secuencia §2.3.1 completa — separar Medallion/EDA, validar `join`, pasada 1, organizar `EDA.ipynb`, pasada 2 — y **congelar el esquema Gold completo + exportar la Gold agregada** al cierre del día. **Heider:** al terminar onboarding, apoya con la **Ilustración 2** (autónoma, no toca la Gold). |
| **vie 5** | Modelo + tablero arrancan sobre Gold congelada | **Sara:** features (anti-fuga, split temporal) + entrenar (logística → GBM), PR-AUC + calibración; iniciar clustering. **Kelly:** Power BI v1 sobre la Gold agregada + narrativa, usando el EDA oficial como guía de diseño. **Heider:** Bronze→`readStream` (Kappa) + Spark SQL + MLflow setup. **Yeison:** diseño del A/B test + documento. |
| **sáb 6** | Insight + integración | **Sara:** cerrar clustering + perfilado; **cruce clasificador × clustering** (el insight). **Heider:** scoring batch + métricas de optimización. **Kelly:** tablero v2. **Yeison:** documento consolidado integrando resultados. |
| **dom 7** | Consolidar + ensayar | Documento consolidado + PPTX. Ensayo de defensa (filtrado en vivo). Q&A por profesor (doc 08 §5). |
| **lun 8** | **ENTREGA** | Buffer, revisión final, subir productos. **No correr cargas pesadas hoy** (regla de cuota). |
| **mar 9** (5–9 pm) | **EXPOSICIÓN** | Ante los tres profesores. |

> **Nota de cronograma (4-jun):** dejar la Gold lista **hoy** (Yeison + Claude Code, en paralelo al onboarding del equipo) **resuelve** la compresión que preocupaba: Sara y Kelly arrancan el viernes sobre una Gold ya congelada, no esperando. Riesgo residual: la secuencia §2.3.1 es larga para un día; si la pasada 2 (paso 5) no alcanza a cerrar hoy, priorizar **`join` validado + limpieza (pasada 1) + esquema congelado + Gold agregada exportada** —eso ya desbloquea aguas abajo— y dejar la pasada 2 (variables ML/agregados del tablero) como ajuste aditivo del viernes temprano, avisando al equipo.

---

## 16. Qué subirle a tu IA para tener contexto

Cada integrante usa su propia IA (Claude o Gemini). Súbele estos documentos de `docs/`. **No subas la transcripción de chats ni el código** —eso vive en el repo y cambia seguido—; sube documentos de referencia estables. Si tu IA necesita ver un notebook o archivo puntual, adjúntalo solo en ese chat; no lo cargues como contexto permanente.

> **Puesta en marcha del entorno:** los pasos para configurarte (crear cuenta en Databricks, vincular GitHub, clonar el Git folder, crear el Volume, subir tu `kaggle.json` y correr el notebook de ingesta) **no se duplican aquí**: viven en el `README.md` de la raíz, que es la guía de configuración del entorno. Este doc 00 da el *qué* y el estado; el `README.md` da el *cómo* del setup.

**Mínimo, para cualquiera del equipo:**
- `00_estado_del_proyecto.md` (este maestro): siempre, el primero que debe leer.
- `02_arquitectura_bigdata_y_databricks.md`: referencia técnica común.

**Según tu frente, además:**
- **Heider (Ingeniería de datos):** doc 02 + `06_contexto_grandes_datos.md`.
- **Sara (Modelado):** `05_contexto_aprendizaje_automatico.md` + el esquema de la Gold / contratos (§13).
- **Kelly (Visualización):** `07_contexto_visualizacion.md` + la Pregunta de Oro y la estructura de la Gold agregada.
- **Yeison (Integración/documento):** todos los anteriores + la propuesta (`03_propuesta_corregida.md`) y las rúbricas (`material_entrega/`).

**Referencia común útil:** rúbricas/criterios (`material_entrega/criterios_evaluacion_pi.docx`) y la propuesta corregida.

---

*Recordatorio: reconfirmar límites de Databricks Free Edition cerca de la entrega (la plataforma cambia rápido).*

---

## 17. Hallazgos y decisiones del EDA oficial (4-jun) — insumo para Sara, Kelly y la pasada 2

El EDA (`notebooks/analysis/EDA.ipynb`) quedó re-fuenteado a **Silver/Gold v1** y corriendo en Databricks (§2.3.1 paso 4 Hecho). Esto resume lo que salió y lo que hay que decidir/hacer. **Es el puente para abrir los siguientes frentes.**

### 17.1 Hallazgos de negocio (base limpia · cuarentena 15–17 nov · 90.6M eventos / 58.6M unidades)
- **Funnel por unidad (base LIMPIA):** cart **3.93%**, conv **2.24%**, cierre **56.9%**, **abandono 43.1%** (58.6M unidades; 994k carritos abandonados). *(La ventana corrupta inflaba el abandono a 51.7% y deprimía el cierre a 48.3%; full-data en §7.)* El negocio es **electrónica**: conv **3.52%**, mayor volumen (20.7M unidades), y el mayor pool de carritos abandonados ≈ **$211M en juego** (de $283.6M totales; full-data inflaba electronics a $349.5M). **Concentración de revenue:** electronics **76.9%**, top-3 **87.3%**; el 50% del revenue lo hacen **50 productos** (0.1%).
- **Dos palancas (síntesis §4.9):** (A) recuperar carritos de alta intención que no cierran (electronics; **Samsung+Apple = 68.6%** de carritos electronics, abandono ~39–40%); (B) retener al núcleo recurrente (**35.8%** de compradores = **73.9%** del revenue; ticket recurrente **$1,464** vs one-time **$289**; 2ª compra mediana **2.9 días** —37% ≤1 día—, **84.4%** a la misma categoría → cross-sell 15.6%).
- **El precio no es el freno**; la decisión es casi inmediata (mediana **2.2 min**).
- **Perfilado Gold (§5) — contraintuitivo y clave para Sara:** las features tienen correlación **débil y NEGATIVA** con la compra; los compradores **navegan menos** (deciden rápido). Son features **pre-carrito** (anti-fuga). **Multicolinealidad** alta (`total_views`≈`distinct_products_viewed` 0.92). → el poder predictivo está en no-linealidades/interacciones; **enriquecer features** en la pasada 2.
- **Flag `sin_navegacion_previa`:** 34.144 sesiones (0.15%), **tasa 55.2%** vs 6.0% global → decisión de Sara (§13): mantener con flag / excluir / segmento aparte.

### 17.2 Calidad de datos de noviembre (DIAGNOSTICADO — §6.1/§7.1) ⚠️
- **La ventana 15–17 nov está corrupta en `purchase`** (15-nov = **0 compras** pese a 467k carritos; volcado del backlog el 16–17; **no** es duplicación). Coincide con un problema reportado por la comunidad del dataset (REES46). → **Cuarentenar 15–17 nov en la pasada 2** (corrompe la etiqueta de noviembre).
- **La divergencia Oct→Nov es mayormente REAL:** excluir la ventana baja el abandono de Nov de 60.9% → 52.0%, pero **no** lo reconcilia con octubre (31.9%). Es estacionalidad pre-fiestas + posible cambio en la captura de `cart` (a confirmar).
- **Black Friday (29-nov)** es pico **real pero modesto** (~+30%); el pico ×6 del 17-nov era el artefacto, no BF.
- **Contexto dataset (consenso comunidad, no oficial — REES46 bajo NDA):** tienda Rusia/CIS, precios en USD, `event_time` en UTC.

### 17.3 Decisión de split train/test — A CERRAR con Sara/equipo (documento en EDA §6.1·bis)
- **Aclaración:** la estratificación va en la **CV interna** (`StratifiedKFold`), **no** en la frontera temporal; la diferencia de tasa Oct↔Nov es **drift**, se maneja con calibración (doc 05 §1.4).
- **Recomendado: Opción C** — train = octubre + noviembre hasta ~23 / test = nov 24–30 (mismo régimen, *out-of-time*, alto volumen) → aísla *skill* del *drift* sin renunciar a noviembre. **Opción A** (train Oct / test Nov-sin-15–17) como **sensibilidad**. **Octubre-solo descartado** (renuncia al out-of-time). Invariante en todas: cuarentena 15–17, `StratifiedKFold`, calibración+Brier, `is_black_friday` solo para estratificar evaluación.
- **Preguntas abiertas para mañana:** ¿el shift es conductual o de medición (captura de `cart`)?; punto de corte exacto de C; ¿C sola o híbrida C+A?; tasa de etiqueta a nivel sesión por mes.

### 17.4 Pendientes para el paso 5 (también listados en la última celda del EDA)
1. **Pasada 2 + calidad:** cuarentenar 15–17 nov; auditar categóricas/`"Unknown"`/outliers/bots; enriquecer features; resolver multicolinealidad.
2. **Cerrar el split con Sara** (doc §6.1·bis) y la decisión del flag `sin_navegacion_previa`.
3. **Congelar contrato Gold (§13)** + particionamiento (doc 02 §3) + exportar la Gold agregada para Power BI.
4. **Limpiar** el Delta temporal de trabajo del EDA (`_tmp_eda_units`).
5. **Alinear con Kelly** (las dos palancas y el mapa pregunta→gráfico, doc 07) y **coordinar la actualización de §6** (los números full-data difieren de los allí citados).

### 17.5 Pasada 2 — calidad de datos (RESUELTO · 4-jun · §2.3.1 paso 5 en curso)
Auditoría ejecutada sobre Silver/Gold completas. Hallazgo transversal: **los "problemas" de limpieza resultaron pequeños o inexistentes**; se documentan para no re-abrirlos.

- **Cuarentena 15–17 nov — CONFIRMADA y materializada** (flag `label_window_corrupt` + `session_date` en Gold · commit `15dc44d`). Tasa de `purchase` por día: **15-nov = 0.0000** (suprimido), **16-nov = 0.0573**, **17-nov = 0.1555** (volcado del backlog, ×3). La etiqueta está *descuadrada en el tiempo*, no simplemente baja; sobre los 3 días queda **inflada** (0.0764 vs 0.0589 fuera de ventana) por el volcado. Cuarentena = excluir de train **y** test (§6.1·bis inv. 1). *Watch-item para Sara (frontera de split): 13–14 nov salen algo blandos (0.042 / 0.033) — posible rampa de degradación del registro; no se expandió la ventana unilateralmente.*
- **Categóricas / `"Unknown"` — NO backfilleable.** `category_id` es limpio (690 valores, **0 mapean a >1 macro**) y completo, pero **0 de 413** ids con macro `"Unknown"` tienen macro conocida en otra fila → el string no se puede re-derivar. **Decisión:** mantener `"Unknown"` como categoría legítima; para **ML usar `category_id`** (mejor señal que el string macro: 14 valores con 32% Unknown); `macro_category` queda para el **tablero** (Kelly), etiquetando "Unknown" como *"Sin taxonomía"* (~32% de eventos · `brand` Unknown ~14%). *Block 2: `categories_explored` debería pasar a `countDistinct(category_id)` (hoy cuenta "Unknown" como un valor más).*
  - **Consistencia de valores verificada (4-jun):** `macro_category`(14) / `sub_category`(59) / `item_type`(86) / `brand`(4304) **sin colisiones case/espacio (0)** ni sinónimos en las listas de baja cardinalidad → **no requiere normalización en Silver** (la taxonomía proviene de un único `category_code` controlado). Único typo de fuente: `sub_category='cartrige'` (consistente; se respeta el origen). **Con esto la limpieza queda cerrada.**
- **Outliers de precio — no hay.** `p999 = 2562`, `max = 2574`, todas las macro topan en ~2574 → precio **acotado en la fuente** (probable cap de REES46); con `price>0` ya filtrado en Silver, **no se winsoriza** (confirma "el precio no es el freno", §17.1).
- **Sesiones-bot — cola despreciable.** `p999 = 76` ev/sesión; **>200 ev = 506 sesiones (0.002%)**, >500 = 16. Impacto ~nulo → **no se añade flag** (revisado, no amerita acción).
