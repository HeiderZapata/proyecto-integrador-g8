# 00 — Estado del Proyecto · PI Grupo 8 (documento maestro)

**Última actualización:** miércoles 3 de junio de 2026
**Entrega de productos:** lunes 8 de junio · **Exposición:** martes 9 de junio (5–9 pm)
**Equipo:** Kelly, Sara, Heider y Yeison
**Repositorio:** `HeiderZapata/proyecto-integrador-g8` · este archivo vive en `docs/`

> **Este es el documento maestro y vivo del proyecto.** Da el contexto general y refleja el estado de avance. Manténganlo actualizado —sobre todo la Parte A (Orientación y estado)— a medida que se completan tareas. Es el primer archivo que cualquier integrante (o su IA) debe leer para entrar en contexto.
>
> **Orden del documento (reestructurado el 3-jun):** está dividido en tres partes para no mezclar planos. **Parte A** = orientación y estado (lo primero que se lee). **Parte B** = el *qué* (contenido estable del proyecto). **Parte C** = el *cómo* (forma de trabajo, repo, plan). Antes, la tabla de estado mezclaba planeación y ejecución sin distinguir fase; ahora se separan explícitamente (Fase 3 vs Fase 4).

---

> ## ⚠️ SEGURIDAD — ACCIÓN PENDIENTE (Heider)
>
> **La credencial de Kaggle ya NO está hardcodeada.** El notebook `notebooks/pipeline/01_sube_datos_kaggle_Databricks.ipynb` ahora lee la llave desde el Volume (`kaggle.json`, sin ningún valor escrito en el código) y el `.gitignore` cubre `kaggle.json` para que no vuelva a subirse al repo.
> **Único pendiente — responsable: Heider:** la llave que estuvo expuesta quedó en el historial de Git, así que se asume comprometida. Hay que **expirar/rotar esa llave desde la cuenta de Kaggle** — eso es lo que la neutraliza (borrarla del archivo no basta). Es uno de los criterios para cerrar Fase 3 (§4). *Este banner se elimina cuando Heider confirme que la llave quedó expirada.*

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
| **Expirar la credencial de Kaggle expuesta** | Heider | **Pendiente** | El notebook ya lee del Volume y el `.gitignore` la cubre (Hecho); falta que Heider **expire la llave en Kaggle** (ver banner). Criterio de cierre de Fase 3 |
| Confirmar roles | Equipo | Pendiente | §11 sigue como PROPUESTO hasta la reunión |
| Congelar los tres contratos | Equipo | Pendiente | §13: esquema Gold · salida modelo · datos dashboard |
| Cronograma mié 3 → mar 9 con fecha y dueño | Equipo | Pendiente | §15 (borrador propuesto, por confirmar) |
| Ordenar el repo (estructura de código + protocolo Git + `.gitignore`) | Heider/Yeison (Claude Code) | Hecho | §12. Estructura creada, notebooks movidos a `pipeline/`, `.gitignore` en la raíz (baseline en `main`) |
| Doc de convenciones Git — `CONTRIBUTING.md` (ramas, PRs, evitar conflictos) | Heider/Yeison | Hecho | §14.1. En la raíz del repo (baseline en `main`) |
| Investigar comunidad del dataset (Kaggle) | Todos (transversal) | Pendiente | Cada dueño contrasta su fase con la comunidad REES46 |

### 2.3 Ejecución — Fase 4 (pendiente; arranca al cerrar Fase 3)

| Frente / Tarea | Responsable | Estado | Notas |
|---|---|---|---|
| Pipeline Medallion (Bronze/Silver/Gold) — **todos los ajustes de la capa** | **Heider/Yeison** | Revisión | v1 corrió sobre 14 GB; **validar `join` Gold (posible sesgo)** + particionamiento por capa + **reconstruir Bronze como `readStream`** (replay Kappa). **Ruta crítica — ver §3** |
| Revisar detalle del `join` en Gold | Heider/Yeison (Sara consultada por el sesgo) | Pendiente | Posible sesgo de selección; compuerta del esquema Gold |
| **Definir qué del EDA-descubrimiento se promueve a Silver/Gold** | **Heider/Yeison** | Pendiente | §12.3. Es *el* paso más importante: dejar la tabla Gold **completa y robusta** (alimenta EDA-entregable y modelos) |
| Streaming replay (`readStream` + `AvailableNow`) | Heider | Pendiente | Liviano · con checkpoint, no en bucle (doc 02 §4) |
| Spark SQL + MLflow + scoring batch | Heider/Sara | Pendiente | Cierra el ciclo batch |
| EDA / funnel (entregable analítico) | Yeison → alinear con Kelly (Viz) | Revisión | v1 reciclado del taller 2; alinear a la Pregunta de Oro y a nombres de variables claros. **Vive en `analysis/`, no en el pipeline (ver §12)** |
| Entrenamiento + evaluación del modelo | Sara | Pendiente | PR-AUC, calibración (no accuracy) |
| Clustering de visitantes | Sara | Pendiente | Con metodología propia; el **cruce clasificador × clustering** es el insight |
| Tablero Power BI ejecutivo | Kelly | Pendiente | Sobre la Gold **agregada** (no 69M filas) |
| Diseño del A/B test | Yeison | Pendiente | Cierre: medir un incentivo sobre el segmento de mayor intención |
| Rehacer Ilustración 2 (diagrama de arquitectura) | Heider/Yeison | Pendiente | Reflejar frontera train/test + dos arquitecturas (doc 02 §3) |
| Q&A de defensa por profesor | Equipo | Pendiente | Insumo: `08_feedback_exposiciones_pregrado.md` §5 |
| Documento consolidado del PI | Equipo | Pendiente | |
| Presentación (PPTX) | Equipo | Pendiente | Incluir narrativa del recorrido |

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

1. **Roles confirmados** — §11 pasa de PROPUESTO a firme.
2. **Cronograma** mié 3 → mar 9 con fecha y dueño por tarea (§15), confirmado por el equipo.
3. **Tres contratos congelados** (§13): esquema de la Gold, salida del modelo, datos que consume el dashboard.
4. **Repo ordenado** (§12): estructura de código definida, protocolo Git operativo, `.gitignore`, **credencial de Kaggle rotada y limpia del historial**.
5. **Kickoff por frente listo** — un prompt/encuadre por frente, para que los chats de Fase 4 abran limpios y heredando el knowledge.

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
- **Volumen:** Oct (5.5 GB) + Nov (9 GB); ~69M unidades en el funnel.
- **Métricas:** cart rate 3.85%, **conversión 2.22%**, **abandono 42.27%**.
- **Modelado:** split temporal (Oct entrena, Nov prueba), corte anti-fuga (solo comportamiento previo al primer cart/purchase), métrica PR-AUC + calibración (no accuracy).

---

## 7. Arquitectura de Grandes Datos (detalle en doc 02)

- **De referencia (producción, solo enunciada):** Kafka/Kinesis → Structured Streaming/Flink → Delta sobre S3/GCS → warehouse/Athena → serving en tiempo real.
- **Implementada (Databricks Free):** ingesta batch → **replay de streaming** (Auto Loader + `Trigger.AvailableNow()` + checkpoint, estilo Kappa) → Medallion en Delta → Spark SQL → MLflow + scoring batch → Power BI.
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

## 11. Reparto de frentes (PROPUESTO — por confirmar en la reunión que cierra Fase 3)

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

### 12.3 Qué del EDA va a Medallion (Heider/Yeison — tarea de §2.3)
El EDA de descubrimiento *informa* las transformaciones, pero estas se **codifican en Silver/Gold**, no se quedan en el notebook de exploración. **Responsables: Heider y Yeison** (es parte de dejar la tabla Gold completa y robusta). A resolver abriendo los notebooks reales con Claude Code: **decidir qué transformaciones discoveradas en el EDA se promueven a la lógica determinista de Silver/Gold** (limpieza, tipado, dedup, reconstrucción de sesión, features) y qué se queda como análisis. La dirección de dependencia es de una sola vía: EDA-descubrimiento → transformaciones (pipeline) → EDA-entregable (consume). Esta tabla Gold alimenta **tanto el EDA-entregable como los modelos**: por eso es el paso más importante y se valida con cuidado.

**Hallazgo concreto (jun):** hoy existe un solo notebook `02_Medallion_Y_EDA_Ecommerce` que **combina Medallion + EDA** en un mismo archivo — justo la conflación que evitamos. Se mueve provisionalmente a `notebooks/pipeline/`; **la primera tarea de Fase 4 es separarlo**: la lógica Medallion se queda en `pipeline/` y el EDA-funnel se extrae a `notebooks/analysis/`.

### 12.4 Convención de packs de contexto
- `material_cursos/<curso>/` — material crudo. **Eliminado del repo** (los packs ya están destilados); si alguien lo necesita en local, mantenerlo fuera de Git.
- `docs/0X_contexto_<curso>.md` — pack destilado (síntesis propia, no copia del material). Sí va al repo.
- Plantilla del pack: (1) técnicas/conceptos, (2) vocabulario y énfasis del profe, (3) herramientas vistas, (4) expectativas de evaluación, (5) mapeo "qué se enseñó → dónde aparece en el proyecto", (6) conceptos/citas para la defensa.

---

## 13. Contratos a congelar (compuerta de Fase 3)

Para trabajar en paralelo sin pisarse, congelar las tres interfaces. Estado: **pendiente** (se congelan en la reunión).

1. **Esquema de la tabla Gold** — columnas, tipos, grano (por sesión), particionamiento. *Depende de validar antes el `join` de Gold (§3).*
2. **Salida del modelo** — qué entrega el scoring (id de sesión, score/probabilidad, segmento), formato y dónde se persiste.
3. **Datos que consume el dashboard** — la Gold **agregada** que alimenta Power BI (no las 69M filas crudas).

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

## 15. Plan con fechas y dueños (mié 3 → mar 9) — PROPUESTO, por confirmar

Borrador para confirmar y rebalancear en la reunión. Respeta la ruta crítica (§3) y la regla de cuota de no correr cargas pesadas la víspera (doc 02 §4). **Cambio clave vs. versión anterior:** Yeison se ancla a la tabla Gold con Heider los dos primeros días (es la compuerta); por eso su trabajo de A/B test y documento se concentra de viernes en adelante.

| Día | Foco | Quién / qué |
|---|---|---|
| **mié 3** (hoy) | Cerrar Fase 3 | **Heider:** rotar credencial Kaggle (revocar → limpiar historial). **Equipo:** reunión — confirmar roles (§11), congelar contratos (§13), validar este doc. **Heider + Yeison:** abrir el trabajo de la Gold — revisar `join` (sesgo; Sara consultada) + decidir **qué del EDA se promueve a Silver/Gold** (§12.3) + particionamiento. **Claude Code:** ordenar repo (§12) + `CONTRIBUTING.md` (§14.1). |
| **jue 4** | **Gold completa y robusta (HITO)** | **Heider + Yeison:** cerrar Gold validada + Bronze→`readStream` (checkpoint, `AvailableNow`); **congelar el esquema Gold** y exportar la Gold agregada. **Sara:** preparar el pipeline de features sobre muestra (anti-fuga, split temporal), listo para enchufar a la Gold. **Kelly:** andamiaje del EDA-funnel sobre Silver. |
| **vie 5** | Modelo + tablero v1 (Gold ya congelada) | **Sara:** features + entrenar (logística → GBM), evaluar PR-AUC + calibración. **Heider:** Spark SQL sobre Gold + setup MLflow + scoring batch. **Kelly:** Power BI v1 sobre la Gold agregada. **Yeison:** liberado de la Gold → diseño del A/B test + Ilustración 2. |
| **sáb 6** | Insight + integración | **Sara:** cerrar clustering + perfilado; **cruce clasificador × clustering**. **Heider:** scoring batch final + métricas de optimización (particiones escaneadas, tiempo). **Kelly:** tablero v2 + narrativa. **Yeison:** iniciar documento consolidado integrando resultados. |
| **dom 7** | Consolidar + ensayar | Documento consolidado completo + PPTX. Ensayo de defensa (filtrado en vivo). Q&A por profesor (doc 08 §5). |
| **lun 8** | **ENTREGA** | Buffer, revisión final, subir productos. **No correr cargas pesadas hoy** (regla de cuota). |
| **mar 9** (5–9 pm) | **EXPOSICIÓN** | Ante los tres profesores. |

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
