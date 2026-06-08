# 00 — Estado del Proyecto · PI Grupo 8 (documento maestro)

**Última actualización:** domingo 7 de junio de 2026
**Entrega de productos:** lunes 8 de junio · **Exposición:** martes 9 de junio (5–9 pm)
**Equipo:** Kelly, Sara, Heider y Yeison
**Repositorio:** `HeiderZapata/proyecto-integrador-g8` · este archivo vive en `docs/`

> **Este es el documento maestro y vivo del proyecto.** Da el contexto general y refleja el estado de avance. Manténganlo actualizado —sobre todo la Parte A (Orientación y estado)— a medida que se completan tareas. Es el primer archivo que cualquier integrante (o su IA) debe leer para entrar en contexto.
>
> **Orden del documento.** Arriba del todo va **🔥 AHORA** (qué hace cada quien hoy y el mapa de la semana) y el **Índice**: si solo tienes un minuto, lee eso. Luego, el cuerpo en tres partes para no mezclar planos: **Parte A** = orientación y estado · **Parte B** = el *qué* (contenido estable) · **Parte C** = el *cómo* (forma de trabajo, repo, plan). Al final, **§17–§18** son insumos densos de ejecución (hallazgos del EDA y plan de Sara), referenciados desde el cuerpo. *La numeración §1–§18 se mantiene estable a propósito (la referencian `README.md`, `CONTRIBUTING.md` y doc 02); por eso §17–§18 quedan al final aunque conceptualmente pertenezcan a las Partes B y C.*

---

> ## ✔ SEGURIDAD — CERRADO (5-jun)
>
> La credencial de Kaggle ya NO está hardcodeada (el notebook de ingesta lee la llave desde el Volume; el `.gitignore` cubre `kaggle.json`) y **Heider expiró/rotó la llave expuesta desde la cuenta de Kaggle (confirmado 5-jun)**, que es lo que la neutraliza (la llave había quedado en el historial de Git y se daba por comprometida). Con eso el riesgo queda **cerrado por completo**: ninguna llave válida vive en el repo ni en su historial, y la que estuvo expuesta ya no sirve.

---

> ## ✅ CAPA GOLD — COMPLETA Y CONGELADA (4-jun)
>
> La secuencia §2.3.1 (tratamiento de datos) **se cerró**: **Gold de 22 columnas CONGELADA** (§13); EDA oficial re-fuenteado a **base limpia** (cuarentena transversal de la ventana corrupta 14–17 nov) con hallazgos y números **refrescados**; **particionamiento layer-aware** con evidencia medida (doc 02 §3); y **7 CSV agregados** para Power BI en `reports/data/` (verificados vs EDA). Esto **desbloquea a Sara (modelado) y Kelly (tablero)**.
> - **Decisiones y hallazgos:** §17 (calidad, cuarentena, features, números limpios) · **§13** (contrato Gold) · **§6.1·bis del EDA** (split) · doc 02 §3 (particionamiento).
> - **Abierto a propósito (se cierra en el empalme con el equipo, vie 5):** decisión de **split** train/test con Sara; flag `sin_navegacion_previa` (§13); diagramas de arquitectura de Heider (Ilustración 2 — ver §2.3).

---

## ✅ FRENTE DE MODELADO — AVANCE (7-jun)

**Split cerrado: Opción C** (train = oct + nov ≤ 23 / test = 24–30 nov), justificado con evidencia de deriva (`notebooks/modeling/03_drift_split_diagnostico`): conversión estable oct↔nov fuera de la corrupción, precio sin deriva (PSI≈0); brecha de tasa train↔test 5.8% (sano). Invariantes: cuarentena 15–17 nov, `StratifiedKFold` solo en CV interna, `is_black_friday` solo para estratificar la evaluación.

**Propensión** (`02_modelado_propension`): baseline trivial (PR-AUC 0.056) → logística (0.096) → GBM+Optuna calibrado: CV 0.123, test PR-AUC 0.118, Brier 0.052 (con/sin Black Friday reportado). Sin fuga (test ≈ CV). Features top: `max_price_viewed`, `electronics_view_share` → coincide con el EDA.

**Clustering** (`04_clustering`): k=4 (defendido por accionabilidad, no por silhouette). Segmentos: C0 electrónica alto valor (42.6%, conv 7.5%) → objetivo del A/B, C2 general bajo valor, C1 explorador que no cierra, C3 anómalo.

**Pendiente:** `06_scoring_mlflow_databricks` — MLflow + scoring batch + Contrato 2 (`user_session`, prob. calibrada, segmento) en Delta.

**Abierto (con Yeison):** confirmar corte exacto del split y posible muestreo Híbrido (C + A).

---

# 🔥 AHORA — Hoy y esta semana

> **Faltan 3 días para la entrega (lun 8) y 4 para la exposición (mar 9).** Esta es la sección de "qué hago ahora". El plan completo día a día está en **§15**; los frentes en **§11**; los contratos en **§13**.

## Hoy · viernes 5-jun — por persona

La capa de datos se declaró cerrada ayer (**Gold congelada**, §13). Hoy se **audita** antes de soltar modelado y tablero, y se hace el **empalme 1-a-1** con Sara y Kelly. Orden del día: **auditoría → empalmes → arranque**.

- **Yeison — orquestación + datos + A/B + documento.** (1) ✅ **Auditoría de la capa de datos HECHA** (Medallion + EDA, Claude Code, rama `feat/auditoria-datos`): **capa sin bloqueantes**, 9 checks en verde — reporte en `auditoria_capa_datos_2026-06-05.md`. **Confirmada EN VIVO contra la Gold real (5-jun, Databricks serverless):** checks 1/2/3/4/7/8 verificados (esquema 22 cols · grano 22.99M sin dups · tasa 0.0610 full / 0.0589 limpia · particionamiento físico medido · flags) + **MLflow tracking operativo (smoke test)** → **riesgo 3 cerrado; capa lista para modelado/tablero sin pendientes de corrida** (ver addendum "Verificación en vivo" en `auditoria_capa_datos_2026-06-05.md`). (2) §13/§17 actualizados (cross-ref `agg_funnel_global`); pendientes menores cerrados (`_tmp_eda_units` borrado). (3) Armar los **planes de empalme** para Sara y Kelly (en el chat de Claude). Luego: avanzar el diseño del **A/B test** y el documento.
- **Sara — modelado.** Estudiar su arranque: **§18** (plan de modelado), **§13** (contrato Gold + set de features) y **§17** (hallazgos del EDA) + pack ML (`05_…`). En el **empalme** con Yeison se cierran sus tres inputs: **split** (recom. C), **flag `sin_navegacion_previa`** y el **contrato 2** (salida del modelo). Tras el empalme: baseline **logística → GBM**, **PR-AUC + calibración**, sobre snapshot local o muestra en Databricks (§18.3).
- **Kelly — visualización.** ✅ **Tablero Power BI v1 COMPLETO (7-jun) — 4 páginas.** Portada con navegación entre páginas + (1) **Análisis global:** 4 KPIs (conv 2.24%, abandono 43.14%, 994K carritos, $283.6M revenue), funnel por categoría con treemap, tipología de visitante y segmentos de comprador con filtro Top N dinámico (5/10/15). (2) **Detalle Electronics:** 4 KPIs ($409 ticket, 41.5% abandono, 511K carritos, $211.1M revenue), Top 10 marcas por carritos abandonados, scatter ticket vs abandono. (3) **Contexto temporal:** tráfico vs conversión diaria, compras diarias, anotaciones Black Friday y ventana corrupta 15–17 nov. Paleta púrpura coherente. Archivo `.pbix` en `reports/powerbi/` (144 KB). **Publicado en Power BI Service** (área de trabajo personal Kelly). **Pendiente:** conectar scores del modelo cuando Sara entregue (Contrato 2).
- **Heider — ingeniería.** Llave Kaggle **expirada ✅**. Aplicar ya los ajustes 1 y 2 de la Ilustración 2 (§2.3); el ajuste 3 (rótulo del split) queda en espera de la decisión de split con Sara (hoy, en el empalme)** (§2.3). Track paralelo (§2.3.1 paso 6): Bronze → **`readStream`** con checkpoint, **Spark SQL**. **MLflow tracking ya VERIFICADO operativo** (smoke test 5-jun, serverless con `set_registry_uri`); el logging del modelo (params, PR-AUC/Brier, modelo calibrado, signature) lo implementa **Sara** en su pipeline (§18.4). **No tocar la Gold** (está congelada).

## Mapa de la semana (detalle en §15)

| Día | Estado | Foco |
|---|---|---|
| mié 3 | ✅ | Reunión — Fase 3 cerrada, roles firmes |
| jue 4 | ✅ | Onboarding + **Gold congelada** + EDA oficial |
| **vie 5 (hoy)** | 🔄 | **✅ Auditoría capa de datos HECHA** (sin bloqueantes, `auditoria_capa_datos_2026-06-05.md`) → empalmes con Sara/Kelly → arranque de modelo y tablero; Heider: Ilustración 2 + `readStream`/MLflow |
| sáb 6 | — | **Tablero v1 ✅ Kelly ·** **Insight** (cruce clasificador × clustering) + scoring batch + tablero v2 + documento |
| dom 7 | — | Consolidar documento + PPTX + **ensayo de defensa** (filtrado en vivo) |
| lun 8 | 🎯 | **ENTREGA** — buffer y revisión final; **sin cargas pesadas** (regla de cuota) |
| mar 9 (5–9 pm) | 🎤 | **EXPOSICIÓN** ante los tres profesores |

---

## Índice

**Parte A — Orientación y estado**
§1 El proyecto en una frase · §2 Estado de avance (fundamentos / Fase 3 / Fase 4 / secuencia de datos §2.3.1) · §3 Ruta crítica · §4 Fase actual y Definición de Hecho

**Parte B — El qué (contenido estable)**
§5 Decisión conceptual: qué sí / qué no · §6 Datos clave · §7 Arquitectura de Grandes Datos · §8 Visualización · §9 Plataforma y herramientas

**Parte C — El cómo (forma de trabajo)**
§10 Flujo de trabajo y fases · §11 Reparto de frentes · §12 Estructura del repo · §13 Contratos congelados (Gold) · §14 Operación: Claude + Git · §15 Plan con fechas y dueños · §16 Qué subirle a tu IA

**Insumos de ejecución** (al final, por densidad)
§17 Hallazgos y decisiones del EDA — *insumo de modelado y tablero* · §18 Plan de arranque para Sara — *modelado*

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
| **Expirar la credencial de Kaggle expuesta** | Heider | **Hecho** | Notebook lee del Volume + `.gitignore` (Hecho); **Heider expiró la llave en Kaggle (confirmado 5-jun)** → riesgo cerrado por completo (ver banner) |
| Confirmar roles | Equipo | **Hecho** | **Aceptados en la reunión del 3-jun.** §11 pasó de PROPUESTO a firme |
| Congelar los tres contratos | Yeison (Gold) / Equipo | **Hecho (Gold)** | §13. **Esquema Gold CONGELADO (4-jun)** —22 columnas— al cerrar la pasada 2. Salida del modelo (la cierra Sara) y datos del dashboard (Gold agregada, ya exportada a `reports/data/`) enunciados |
| Cronograma mié 3 → mar 9 con fecha y dueño | Equipo | Ajustado (4-jun) | §15. La reunión movió el arranque: jue 4 = onboarding del equipo + Yeison propone plan a Heider |
| Ordenar el repo (estructura de código + protocolo Git + `.gitignore`) | Heider/Yeison (Claude Code) | Hecho | §12. Estructura creada, notebooks movidos a `pipeline/`, `.gitignore` en la raíz (baseline en `main`) |
| Doc de convenciones Git — `CONTRIBUTING.md` (ramas, PRs, evitar conflictos) | Heider/Yeison | Hecho | §14.1. En la raíz del repo (baseline en `main`) |
| Investigar comunidad del dataset (Kaggle) | Todos (transversal) | Pendiente | Cada dueño contrasta su fase con la comunidad REES46 |

### 2.3 Ejecución — Fase 4 (pendiente; arranca al cerrar Fase 3)

| Frente / Tarea | Responsable | Estado | Notas |
|---|---|---|---|
| Pipeline Medallion (Bronze/Silver/Gold) — **todos los ajustes de la capa** | **Heider/Yeison** | **Hecho (4-jun)** | Secuencia §2.3.1 **completa (pasos 1–5)**: Gold v1 → pasada 2 (calidad + cuarentena 14–17 nov + 22 features + multicolinealidad resuelta) → **particionamiento layer-aware** → **contrato §13 CONGELADO** → Gold agregada BI exportada. EDA oficial a base limpia. **Detalle en §17 y §13.** *(Track paralelo §2.3.1 paso 6 —`readStream`/MLflow/scoring— sigue pendiente.)* |
| Entrenamiento + evaluación del modelo | Sara | Pendiente | PR-AUC, calibración (no accuracy). Arranca con la Gold congelada (§13). **Plan de arranque detallado en §18** |
| Clustering de visitantes | Sara | Pendiente | Con metodología propia; el **cruce clasificador × clustering** es el insight. Encuadre en §18 |
| Tablero Power BI ejecutivo | Kelly | Pendiente | Sobre la Gold **agregada** (no 69M filas). Arranca con la Gold v1 (paso 3) |
| Diseño del A/B test | Yeison | Pendiente | Cierre: medir un incentivo sobre el segmento de mayor intención |
| Rehacer Ilustración 2 (diagramas de arquitectura de datos) | Heider construye · Yeison revisa (co-responsables) | **En revisión** | Heider añadió **Ilustración 2 (implementada, Kappa) e Ilustración 3 (referencia)** en doc 03 (Mermaid). ⚠️ **3 ajustes pendientes para Heider** antes de la entrega — ver el callout debajo de esta tabla |
| Q&A de defensa por profesor | Equipo | Pendiente | Insumo: `08_feedback_exposiciones_pregrado.md` §5 |
| Documento consolidado del PI | Equipo | Pendiente | |
| Presentación (PPTX) | Equipo | Pendiente | Incluir narrativa del recorrido |

> **📐 Para Heider — Ilustración 2: 3 ajustes + verificación final antes de la entrega (revisión de Yeison).** Heider integró en `docs/03_propuesta_corregida.md` dos diagramas Mermaid: **Ilustración 2** (arquitectura implementada, estilo Kappa) e **Ilustración 3** (de referencia). Muy buena base; **la Ilustración 3 está correcta**. La **Ilustración 2** tiene 3 detalles que **no coinciden con lo congelado el 4-jun** y conviene alinear para que el diagrama defienda lo que de verdad hicimos (el jurado de HPC pregunta esto):
>
> 1. **Silver — partición.** El diagrama dice *"Partición: fecha+categoría"*. Lo **implementado** es **partición por `date` + `ZORDER (category_id)`** (la categoría es *clustering*, no partición). → Rotular: *"Silver · partición por fecha · ZORDER categoría"*. (Evidencia medida: Silver 6.41 GB en 61 particiones — doc 02 §3.)
> 2. **Gold — NO se particiona.** El diagrama no debe mostrar la Gold particionada: a **~1.33 GB (≪ 1 TB)** se decidió **`OPTIMIZE` + `ZORDER (session_date, user_id)`**, no partición (particionarla por fecha daría micro-archivos de ~22 MB, el anti-patrón). → Mostrar la Gold con *data-skipping por `session_date`*, sin partición. (doc 02 §3, regla "decidir por tamaño".)
> 3. **Split train/test**. ⏸ BLOQUEADO — esperar el empalme con Sara (hoy 5-jun). El diagrama fija "Train Octubre / Test Noviembre", pero el split está ABIERTO con Sara (recomendado C = train Oct+Nov→23 / test Nov 24–30; ver EDA §6.1·bis). Heider NO debe rotular este punto hasta que el split quede cerrado en el empalme de hoy. Cerrado el split: rotular con el corte acordado, o genérico "Split temporal (corte por fecha)" si se prefiere no fijar el mes en el diagrama.
>
> 4. **⏳ TAREA PENDIENTE (7-jun) — verificar que la Ilustración 2 refleje la versión FINAL de la capa (tras sellar ingesta/medallion).** Hubo cambios que el diagrama debe recoger: **(a) Bronze por Auto Loader (`cloudFiles`) + `Trigger.AvailableNow` + checkpoint = replay de streaming REAL (Kappa), ya NO batch** (el código dejó de ser batch); **(b) cuarentena 14–17 nov** (antes 15–17); **(c) `user_id` determinista en la Gold**; **(d) la capa de CONSUMO**: Gold agregada → **11 CSV** → Power BI. → **Contrastar el diagrama de Heider end-to-end (ingesta `readStream` → Medallion → consumo BI) contra estos ajustes antes de la entrega.** *(Hacerlo cuando la corrida de sellado esté verificada.)*
>
> *No se editó el diagrama de Heider (es su deliverable). Los ajustes 1 y 2 (partición de Silver, Gold sin partición) NO dependen de nada y Heider puede aplicarlos ya; solo el ajuste 3 (rótulo del split) espera la decisión con Sara. La cuarentena 14–17 nov puede mencionarse como nota de calidad en el diagrama si se quiere.*

#### 2.3.1 Tratamiento de datos — secuencia ordenada (Heider/Yeison · hoy en adelante)

**Por qué este orden:** primero el *esqueleto* correcto de la Gold (grano + `join` + anti-fuga), después la *carne* (limpieza y variables informadas por el EDA). Limpiar o crear variables sobre un `join` con sesgo es retrabajo. Dos pasadas sobre la Gold, con el EDA en medio; la pasada 2 es **solo aditiva** para no romper el contrato (ver ⚠️). Es el bloque que **tú y Yeison hacen hoy** con Claude Code; Heider apoya el track paralelo (paso 6) o la Ilustración 2.

| # | Paso | Responsable | Estado | Notas |
|---|---|---|---|---|
| 1 | **Separar Medallion ↔ EDA** | Heider/Yeison (Claude Code) | **Hecho (4-jun)** | §12.3. `02_medallion.ipynb` queda solo Medallion (funnel-EDA retirado); `eda_ecommerce.ipynb` movido a `analysis/`, retitulado a "EDA del PI" y con TODO de re-fuente a Silver/Gold. Commit `9f09a8b` |
| 2 | **Validar la estructura de la Gold** (`join`/sesgo, reconstrucción de sesión, grano, corte anti-fuga) | Heider/Yeison (Sara consultada por el sesgo) | **Hecho (4-jun)** | Diagnóstico (`02b_diagnostico_gold_join.ipynb`) confirmó: sesgo de selección real pero pequeño (descartes 9× positivos, ~0.1%), grano roto (`user_session` con >1 `user_id`) y gotcha del frame RANGE. Commit `27b66b6` |
| 3 | **Complementar Medallion — pasada 1** (con el EDA *actual*): limpieza completa, tipado, dedup, nulos/outliers de precio, sesiones-bot; construir/eliminar/transformar variables que el EDA actual ya justifique | Heider/Yeison | **Hecho (4-jun)** | Gold corregida (corte determinista, LEFT join sin sesgo, grano 1 fila = 1 sesión + flag `sin_navegacion_previa`) + Silver con `dropDuplicates` y `price>0`. **Gold v1 validada: 22.99M sesiones, tasa etiqueta 0.0610, 0 duplicados.** Pendiente (sin umbral acordado): outliers de precio y sesiones-bot. Commit `3f66e6a`. *El contrato se congela tras la pasada 2 (paso 5)* |
| 4 | **Organizar el `eda_ecommerce.ipynb` oficial** sobre la Gold v1 (reubicar en `analysis/`, conectar a Silver/Gold, retitular, revisar completitud vs. Pregunta de Oro) | Heider/Yeison (Claude Code) → alinear con Kelly | **Hecho (4-jun)** | §12.3. **EDA re-fuenteado a Silver/Gold v1 (capa de agregados Spark→pandas; ningún gráfico re-escanea Silver), corriendo entero en Databricks.** Se añadieron **§5 (perfilado Gold v1)** y **§6 (temporal/Black Friday, curva de intención, tipología de visitantes)** + §7 diagnóstico. Insumo de Kelly (tablero) y Sara (features). **Ver §17 para los hallazgos y decisiones que salieron.** Andamiaje local en `notebooks/analysis/_build/` (gitignored). |
| 5 | **Pasada 2 sobre la Gold** (según el EDA): correlación/multicolinealidad, variables para ML, métricas/tablas agregadas para el tablero, **calidad de datos** | Heider/Yeison | **Casi (§13 ✅)** | Cierra "Gold completa y robusta". ✅ **Hecho (4-jun):** calidad (cuarentena 14–17 nov + categóricas/precio/bots auditados), **8 features nuevas (22 cols)**, multicolinealidad resuelta (`revisit_intensity`), particionamiento layer-aware, **contrato §13 CONGELADO**, EDA re-fuenteado a base limpia con hallazgos refrescados. **Pendiente menor cerrado (5-jun):** `_tmp_eda_units` borrado. *(Block 3 — **7 CSVs agregados** en `reports/data/` —los 6 por-categoría/marca/segmento + `agg_funnel_global` (KPI global incl. `Unknown`)—, verificados vs EDA — **Hecho**.)* **Auditoría 5-jun: capa sin bloqueantes** (`auditoria_capa_datos_2026-06-05.md`). |
| 6 | **Track paralelo/posterior** (no gatea la correctitud de la Gold): Bronze→`readStream` (Kappa) · Spark SQL · MLflow tracking (✅ **verificado operativo 5-jun**) · scoring batch | Heider (readStream/SQL) · Heider/Sara (MLflow/scoring) | Parcial→✅ | **Bronze→`readStream` ✅ IMPLEMENTADO y verificado (8-jun): Auto Loader (`cloudFiles`) + `Trigger.AvailableNow` + checkpoint; 109.95M filas sin duplicar** (doc 02 §4). **MLflow tracking VERIFICADO operativo** (smoke test 5-jun en serverless; requiere `set_registry_uri("databricks-uc")` antes de `set_experiment` — gotcha Free Edition, doc 02 §4); el **logging del modelo** (params, PR-AUC/Brier, calibrado, signature) lo implementa Sara (§18.4). **Scoring batch es de los últimos pasos: requiere modelo entrenado** |

> **Nota de congelación (4-jun · ACTUALIZADA):** la pasada 2 (paso 5) **se ejecutó hoy** —calidad, cuarentena 14–17 nov, features enriquecidas (22 cols), multicolinealidad, particionamiento layer-aware— y el **esquema Gold quedó CONGELADO (§13)**. El EDA oficial se re-fuenteó a la **base limpia** (cuarentena transversal) y sus hallazgos/números se refrescaron. La regla "aditivo, nunca renombra ni elimina" sigue como red de seguridad. *Pendiente menor cerrado (5-jun): `_tmp_eda_units` borrado. Block 3 (**7 CSVs**: los 6 originales + `agg_funnel_global`) ya en `reports/data/`.* **La auditoría del 5-jun (Medallion + EDA) confirmó la capa sin bloqueantes** (`auditoria_capa_datos_2026-06-05.md`; única recomendación opcional: `user_id` determinista, no bloqueante). La **decisión de split** queda **abierta con Sara** (§6.1·bis).

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
4. **Repo ordenado** (§12): estructura de código definida (Hecho), protocolo Git operativo (Hecho), `.gitignore` (Hecho), **credencial de Kaggle neutralizada** — **HECHO** (llave expirada por Heider, confirmado 5-jun; ninguna llave válida vive en el repo ni en su historial).
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
- **Volumen:** Oct (5.5 GB) + Nov (9 GB); 109.5M eventos → ~69.6M unidades full / **~58.6M en base limpia** (sin 14–17 nov). *(⏳ conteos de base limpia a refrescar: antes excluían solo 15–17; ahora también el 14 — §17 callout 7-jun.)*
- **Métricas (base limpia · EDA oficial, cuarentena 14–17 nov · ⏳ a refrescar tras re-correr con el 14 excluido):** cart **3.93%**, **conversión 2.24%**, **cierre 56.9%**, **abandono 43.1%** (unidad = producto-en-sesión). *Full-data (con la ventana corrupta): abandono 51.7% / cierre 48.3% — el 15-nov inflaba el abandono +8.6pp; ver §7/§17.*
- **Modelado:** split temporal *out-of-time* (corte por fecha; **recom. Opción C**, abierto con Sara — §17.3), corte anti-fuga (solo comportamiento previo al primer cart/purchase), métrica PR-AUC + calibración (no accuracy).
- → **Hallazgos completos y decisiones de calidad de datos** (funnel sobre base limpia, dos palancas de negocio, cuarentena 14–17 nov, perfilado para Sara): **§17**.

---

## 7. Arquitectura de Grandes Datos (detalle en doc 02)

- **De referencia (producción, solo enunciada):** Kafka/Kinesis → Structured Streaming/Flink → Delta sobre S3/GCS → warehouse/Athena → serving en tiempo real.
- **Implementada (Databricks Free):** ingesta batch → **replay de streaming** (Auto Loader + `Trigger.AvailableNow()` + checkpoint, estilo Kappa) → Medallion en Delta → Spark SQL → MLflow + scoring batch → Power BI.
- **Ingesta: Volume de Databricks, NO bucket S3 externo (DECISIÓN FIRME — 4-jun).** El crudo se queda en el Volume `ecommerce_raw` y Databricks ingesta desde ahí. Por qué: (1) un Volume **ya está respaldado por object storage** —leer del Volume *es* ingestar desde un almacén de objetos—, y Auto Loader (`cloudFiles`) puede apuntar al path del Volume, así que el replay de streaming/Kappa se demuestra **sin** bucket externo; (2) S3 agregaría una cuenta AWS y credenciales que gestionar (choca con *no exponer secretos* y con la reproducibilidad del repo) y las *external locations / storage credentials* son limitadas en Free Edition; (3) re-subir y re-ingestar 14 GB quema tiempo y cuota a pocos días de entregar, sin resolver ningún problema actual. S3 solo valdría la pena si herramientas **fuera** de Databricks tuvieran que leer el crudo, si hubiera un *landing zone* multi-fuente real, o si el Volume no aguantara el tamaño —nada de eso aplica aquí. **En la Ilustración 2:** S3 + Auto Loader van dibujados como la arquitectura **de referencia** productiva; el **Volume respaldado por object storage** es la **implementada**. Esta es también la respuesta de Q&A a «¿por qué no S3?».
- **Streaming liviano — ✅ IMPLEMENTADO (8-jun):** Bronze convertido a `readStream` con **Auto Loader (`cloudFiles`) + `Trigger.AvailableNow` + checkpoint** (replay Kappa, no en bucle; verificado 109.95M sin duplicar). Cubre la Unidad 4 y da sustancia a la narrativa Kappa. Nada de broker/productor/AWS.
- **Particionamiento por capa (doc 02 §3 · decisión por TAMAÑO):** Bronze/Silver por **fecha** (capas grandes → particiones de ~128 MB+; Silver además `ZORDER (category_id)`). La **Gold de sesión NO se particiona** (~1.33 GB ≪ 1 TB → `OPTIMIZE` + `ZORDER (session_date, user_id)`; particionarla por fecha daría micro-archivos de ~22 MB, el anti-patrón). **Nunca** por alta cardinalidad. La **frontera train/test** = filtro sobre `session_date` con *data-skipping*. Evidencia medida (`DESCRIBE DETAIL`) en doc 02 §3. *(Es también respuesta de Q&A: "¿por qué no particionan la Gold?" — conocer la regla y su excepción.)*

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

**Convención de nombres (4-jun).** Notebooks en `snake_case` con prefijo numérico por orden de ejecución en `pipeline/`: `01_ingesta_kaggle` → `02_medallion` → `03_gold_agregada_bi` (+ `02b_diagnostico_gold_join`, diagnóstico puntual del `join`). EDA entregable: `analysis/eda_ecommerce`. *(Renombrados desde nombres mixtos —`01_sube_datos_kaggle_Databricks`, `02_medallion`, `EDA`— el 4-jun, con referencias cruzadas actualizadas.)* **Excepción del `.gitignore`:** los CSV de la Gold agregada en `reports/data/` **sí** se versionan (`!reports/data/*.csv`); el resto de `*.csv` sigue ignorado.

**Regla mental:** si el código **produce** una tabla Delta que otros consumen → `pipeline/`. Si **lee** capas ya hechas para aprender o comunicar → `exploration/` o `analysis/`. El funnel "reciclado del taller 2" pertenece a `analysis/`, no al pipeline.

**Por qué separar (no es estética):** (1) **cuota** — si los gráficos vivieran dentro del pipeline, cada reconstrucción de una capa quemaría cómputo renderizando visualizaciones (doc 02 §4); (2) **anti-fuga** — el EDA de features debe respetar el corte temporal Oct/Nov; enredarlo con la ingesta facilita "espiar" datos de test.

### 12.3 Separar Medallion del EDA + dejar el EDA oficial (Heider/Yeison — tarea de §2.3)
El EDA de descubrimiento *informa* las transformaciones, pero estas se **codifican en Silver/Gold**, no se quedan en el notebook de exploración. **Co-responsables: Heider y Yeison** (es parte de dejar la tabla Gold completa y robusta). La dirección de dependencia es de una sola vía: EDA-descubrimiento → transformaciones (pipeline) → EDA-entregable (consume). Esta tabla Gold alimenta **tanto el EDA-entregable como los modelos**, y el EDA es **insumo directo de las decisiones de diseño del tablero (Kelly) y de features (Sara)** — el conocimiento del negocio que sale del EDA es crucial, no decorativo. Por eso es el paso más importante y se valida con cuidado.

**Estado — HECHO (4-jun).** La separación Medallion ↔ EDA se completó:
- **`02_medallion.ipynb`** (en `pipeline/`) quedó **solo Medallion** (Bronze/Silver/Gold, determinista e idempotente); el EDA-funnel delgado que traía se retiró.
- **`eda_ecommerce.ipynb`** (el EDA real del taller, originalmente sobre muestra) se **reubicó en `analysis/`**, se **re-fuenteó a Silver/Gold** (capa de agregados Spark→pandas; corre entero en Databricks **sin re-escanear los 14 GB**), se retituló a "EDA del PI" y se llevó a **base limpia** (cuarentena 14–17 nov). Sus hallazgos alimentan a **Kelly** (tablero) y **Sara** (features) — **ver §17**.
- La limpieza/tipado/dedup/reconstrucción de sesión/features se **codificaron en Silver/Gold** (no quedaron en el notebook de análisis). La dirección de dependencia se respetó: EDA-descubrimiento → transformaciones (pipeline) → EDA-entregable (consume).

*(Estrategia de cuota respetada, doc 02 §4: el cómputo pesado se concentra en la "capa de agregados" sobre Silver/Gold y baja tablas pequeñas a pandas; ningún gráfico re-escanea Silver.)*

### 12.4 Convención de packs de contexto
- `material_cursos/<curso>/` — material crudo. **Eliminado del repo** (los packs ya están destilados); si alguien lo necesita en local, mantenerlo fuera de Git.
- `docs/0X_contexto_<curso>.md` — pack destilado (síntesis propia, no copia del material). Sí va al repo.
- Plantilla del pack: (1) técnicas/conceptos, (2) vocabulario y énfasis del profe, (3) herramientas vistas, (4) expectativas de evaluación, (5) mapeo "qué se enseñó → dónde aparece en el proyecto", (6) conceptos/citas para la defensa.

---

## 13. Contratos a congelar (compuerta de Fase 3)

Para trabajar en paralelo sin pisarse, congelar las tres interfaces. Estado: **el esquema Gold quedó CONGELADO (4-jun) al cierre de la pasada 2 (§2.3.1 paso 5)**; los otros dos se enuncian para que la Gold ya nazca sirviéndolos.

1. **Esquema de la tabla Gold** — columnas, tipos, grano (por sesión), particionamiento. ✅ **CONGELADO** (pasada 2 cerrada; `join` validado §3). 22 columnas; regla de evolución: **aditivo, nunca renombra ni elimina** (Sara/Kelly pueden añadir features sin romper el contrato).
2. **Salida del modelo** — qué entrega el scoring (id de sesión, score/probabilidad, segmento), formato y dónde se persiste. *Lo cierra Sara al definir el modelo; se enuncia hoy.*
3. **Datos que consume el dashboard** — la Gold **agregada** que alimenta Power BI (no las 69M filas crudas). *Las tablas/métricas agregadas salen de la pasada 2 (§2.3.1 paso 5).*

> **🔒 Esquema Gold — CONGELADO (4-jun · `feat/pipeline`).**
> Grano: **1 fila = 1 `user_session`** · **22.99M sesiones** (full) · tasa etiqueta (purchase) **0.0610** full / **0.0597** base limpia (cuarentena **14–17 nov**; era 0.0589 con 15–17) · **22 columnas**. Evolución permitida: **aditiva** (nunca renombrar/eliminar).
>
> *Pipeline sellado (8-jun, `feat/pipeline`): Bronze por **Auto Loader + `Trigger.AvailableNow` + checkpoint** (replay Kappa, ya no batch); **`user_id` determinista** (user_id del primer evento por `event_time`); `label_window_corrupt` = **14–17 nov**. Verificado en vivo: Bronze 109.95M sin duplicar · grano 22.99M intacto · limpia 0.0597 / corrupta 0.0691.*
>
> | Grupo | Columnas (tipo) |
> |---|---|
> | Clave / etiqueta | `user_session` (string, PK) · `target_purchase` (int 0/1 = Y) · `user_id` (string) |
> | Conductuales pre-corte (anti-fuga) | `total_views` (int) · `distinct_products_viewed` (int) · `brands_compared` (int) · `categories_explored` (int, macro) · `categories_explored_cid` (int, `category_id` — preferida para ML) · `browsing_duration_sec` (int) |
> | Precio / categoría (pre-corte) | `avg_price_viewed` (float) · `max_price_viewed` (float) · `electronics_view_share` (float 0–1) |
> | Decisividad / ritmo (derivadas) | `revisit_intensity` (float, decorrelaciona el par 0.92) · `views_per_minute` (float) · `avg_inter_event_sec` (float) |
> | Temporal / calendario | `session_date` (date) · `session_hour` (int) · `day_of_week` (int) · `is_weekend` (int 0/1) |
> | Banderas | `sin_navegacion_previa` (bool) · `is_black_friday` (int 0/1, **solo estratificar evaluación, no feature**) · `label_window_corrupt` (int 0/1, **cuarentena 14–17 nov = filtro, no se borra**) |
>
> **Particionamiento (doc 02 §3 · decisión por tamaño):** la Gold de sesión (~1–2 GB ≪ 1 TB) **NO se particiona** (evita micro-archivos) → `OPTIMIZE` + `ZORDER BY (session_date, user_id)`; el *data-skipping* poda el split train/test. Bronze/Silver sí particionadas por fecha. *(Físico: se materializa al re-correr el pipeline; el contrato —esquema + estrategia— ya es definitivo.)*
>
> **📌 Para Sara (features · §11) — set inicial de la pasada 2, A REVISAR por modelo.** La pasada 2 añade a la Gold (aditivo, anti-fuga, commit en `feat/pipeline`) un set informado por el EDA (§17.1): `categories_explored_cid` (usa `category_id` —limpio— en vez del conteo macro contaminado por `"Unknown"`), `avg_price_viewed`, `max_price_viewed`, `electronics_view_share` (señal de negocio: electrónica = 75% revenue), `is_weekend`, y tres de **decisividad/ritmo**: `revisit_intensity` (**decorrelaciona el par `total_views`≈`distinct_products_viewed`, corr 0.92**), `views_per_minute`, `avg_inter_event_sec`. **Es un punto de partida, no la palabra final:** Sara decide qué raw column del par colineal soltar y si cada modelo necesita **FE adicional** (interacciones, encoding de `category_id` —target/frequency—, codificación cíclica de `session_hour`, escalado). Todo es **anti-fuga** (solo comportamiento pre-corte) y **aditivo**: no rompe el contrato; seleccionar/ignorar es libre aguas abajo. → El **plan de arranque de modelado** (estrategia híbrida, dónde iterar, qué entregar y qué decisiones cerrar) está en **§18**.
>
> **⚠️ Decisión pendiente para Sara (modelado) — flag `sin_navegacion_previa`.** Marca **34,144 sesiones (~0.15%)** que "abren" con `cart`/`purchase`: por el corte anti-fuga tienen **todas las features de navegación en 0** y son **mayormente positivas** (en la muestra, ~55% vs 6% global). El `inner join` viejo las descartaba en silencio (sesgo de selección); la Gold v1 las conserva con el flag. Sara debe decidir cómo tratarlas: **(A)** mantenerlas usando `sin_navegacion_previa` como variable; **(B)** excluirlas del entrenamiento (el clasificador predice desde el comportamiento *previo*, que aquí no existe); o **(C)** tratarlas como segmento aparte. Impacto chico (0.15%) pero a definir antes de entrenar. Cierra el *"Sara consultada por el sesgo"* de §2.3.1 paso 2.

---

## 14. Operación: Claude + Git

- **Proyecto de Claude** con knowledge = packs de cursos + propuesta corregida + rúbricas + esquema del dataset + convenciones del repo.
- **Claude Code** para todo lo que toque el repo (refactor, limpieza, README, **ordenar el repo según §12**, eliminar el secreto, reproducibilidad).
- Cada integrante usa su IA por frente; convergen vía los contratos (§13).
- **Git:** ramas por frente + PRs; Pull antes / Push después; `.gitignore`; secretos en variables de entorno / Databricks secrets (nunca en el código).
- **Merge a `main` centralizado:** una sola persona fusiona los PRs —**Yeison** (suplente: Heider)—; nadie más toca el botón de merge. Esto evita que cuatro personas fusionen a la vez y mantiene `main` siempre funcional (detalle en `CONTRIBUTING.md` §5–§6).
- **Credenciales de Kaggle:** cada integrante usa **su propia** API key *Legacy*, subida a su Volume `ecommerce_raw` —**nunca al repo**— y el notebook de ingesta la lee de ahí (README Paso 6). La llave que estuvo expuesta (Heider) ya fue **expirada** (ver banner de seguridad).

### 14.1 Convenciones de Git (`CONTRIBUTING.md`)
Para que cuatro personas trabajen en paralelo sin conflictos de versiones, el repo lleva un `CONTRIBUTING.md` **en la raíz** (GitHub lo muestra al abrir PRs). Contenido mínimo:
- **Una rama por frente** (`feat/pipeline`, `feat/modelo`, `feat/viz`, `feat/ab-test`, `feat/doc`); nunca trabajar directo sobre `main`.
- **Convención de nombres** de ramas y de commits (mensajes claros, en presente).
- **Flujo de PR:** `pull` antes de empezar / `push` al terminar; PR a `main` con al menos una revisión antes de fusionar.
- **Evitar conflictos:** respetar la separación de carpetas (§12) —cada frente toca su carpeta—; coordinar los cambios a archivos compartidos (este doc 00, los contratos).
- **Nunca** commitear secretos ni datos pesados (`.gitignore`: `material_cursos/`, credenciales, datos crudos).

*Alternativa de nombre, si prefieren mantener el esquema numerado de `docs/`: `docs/09_convenciones_git.md`. Recomiendo `CONTRIBUTING.md` por el soporte nativo de GitHub.*

---

## 15. Plan con fechas y dueños (mié 3 → mar 9) — AJUSTADO el 5-jun

Ajustado tras el avance del 4-jun (**Gold congelada**) y la decisión de **auditar la capa de datos antes de soltar aguas abajo**. Respeta la ruta crítica (§3) y la regla de cuota (no correr cargas pesadas la víspera, doc 02 §4). **Hoy (vie 5) el orden es: auditoría de Medallion+EDA → empalmes 1-a-1 con Sara y Kelly → arranque de modelo y tablero.** La Gold quedó lista el jueves, así que Sara y Kelly **no esperan datos**; el único gate de hoy es confirmar que la capa está sin errores y cerrarles sus inputs.

| Día | Foco | Quién / qué |
|---|---|---|
| **mié 3** | Reunión — Fase 3 cerrada | **HECHO.** Roles aceptados (§11 firme), avances y pasos a seguir presentados. Heider con instrucción de expirar la credencial Kaggle. |
| **jue 4** | ✅ HECHO | **Onboarding del equipo + Gold congelada + EDA oficial (HITO).** **Kelly, Sara, Heider:** entendieron repo + docs y estudiaron su tema (ML, Viz, ingeniería). **Yeison + Claude Code:** ejecutaron la secuencia §2.3.1 completa (separar Medallion/EDA, validar `join`, pasada 1, organizar `eda_ecommerce.ipynb`, pasada 2) → **esquema Gold CONGELADO** (§13) + **Gold agregada exportada** a `reports/data/`. **Heider:** llave Kaggle expirada + apoyo a la Ilustración 2. |
| **vie 5** (hoy) | **Auditoría → empalmes → arranque** | **Yeison:** auditoría de la capa de datos (Claude Code, rama `feat/auditoria-datos`) → actualizar §13/§17 si algo cambia → **planes de empalme** de Sara y Kelly; luego A/B + documento. **Sara:** estudiar §18/§13/§17 → **empalme** (cierra split recom. C, flag `sin_navegacion_previa`, contrato 2) → baseline logística → GBM, PR-AUC + calibración; iniciar clustering (§18.3). **Kelly:** estudiar §17/`07_…`/Pregunta de Oro → **empalme** (mapa pregunta→gráfico) → Power BI v1 sobre la Gold agregada. **Heider:** los 3 ajustes de la Ilustración 2 (§2.3) + Bronze→`readStream` (Kappa) + Spark SQL (**MLflow tracking ya verificado operativo 5-jun**; el logging del modelo lo hace Sara, §18.4). |
| **sáb 6** | Insight + integración | **Sara:** cerrar clustering + perfilado; **cruce clasificador × clustering** (el insight). **Heider:** scoring batch + métricas de optimización. **Kelly:** tablero v2. **Yeison:** documento consolidado integrando resultados. |
| **dom 7** | Consolidar + ensayar | Documento consolidado + PPTX. Ensayo de defensa (filtrado en vivo). Q&A por profesor (doc 08 §5). |
| **lun 8** | **ENTREGA** | Buffer, revisión final, subir productos. **No correr cargas pesadas hoy** (regla de cuota). |
| **mar 9** (5–9 pm) | **EXPOSICIÓN** | Ante los tres profesores. |

> **Nota de cronograma (5-jun):** la Gold quedó congelada el 4-jun, así que Sara y Kelly **no esperan datos**. El único gate de hoy es la **auditoría** (confirmar que la capa de datos está sin errores ni descuidos que comprometan modelado/tablero) y el **empalme** (cerrarles inputs y entregarles el arranque). Riesgo residual: si la auditoría destapa un fix de la Gold, se aplica como cambio **aditivo** (§13) en `feat/pipeline` y se reexporta el agregado; el empalme puede arrancar en paralelo con lo que no dependa del fix. Mantener la regla de cuota: nada de cargas pesadas el lun 8 (víspera).

---

## 16. Qué subirle a tu IA para tener contexto

Cada integrante usa su propia IA (Claude o Gemini). Súbele estos documentos de `docs/`. **No subas la transcripción de chats ni el código** —eso vive en el repo y cambia seguido—; sube documentos de referencia estables. Si tu IA necesita ver un notebook o archivo puntual, adjúntalo solo en ese chat; no lo cargues como contexto permanente.

> **Puesta en marcha del entorno:** los pasos para configurarte (crear cuenta en Databricks, vincular GitHub, clonar el Git folder, crear el Volume, subir tu `kaggle.json` y correr el notebook de ingesta) **no se duplican aquí**: viven en el `README.md` de la raíz, que es la guía de configuración del entorno. Este doc 00 da el *qué* y el estado; el `README.md` da el *cómo* del setup.

**Mínimo, para cualquiera del equipo:**
- `00_estado_del_proyecto.md` (este maestro): siempre, el primero que debe leer.
- `02_arquitectura_bigdata_y_databricks.md`: referencia técnica común.

**Según tu frente, además:**
- **Heider (Ingeniería de datos):** doc 02 + `06_contexto_grandes_datos.md`.
- **Sara (Modelado):** `05_contexto_aprendizaje_automatico.md` + el esquema de la Gold / contratos (§13) + **el plan de arranque de modelado (§18)**.
- **Kelly (Visualización):** `07_contexto_visualizacion.md` + la Pregunta de Oro y la estructura de la Gold agregada.
- **Yeison (Integración/documento):** todos los anteriores + la propuesta (`03_propuesta_corregida.md`) y las rúbricas (`material_entrega/`).

**Referencia común útil:** rúbricas/criterios (`material_entrega/criterios_evaluacion_pi.docx`) y la propuesta corregida.

---

*Recordatorio: reconfirmar límites de Databricks Free Edition cerca de la entrega (la plataforma cambia rápido).*

---

## 17. Hallazgos y decisiones del EDA oficial (4-jun) — insumo para Sara, Kelly y la pasada 2

El EDA (`notebooks/analysis/eda_ecommerce.ipynb`) quedó re-fuenteado a **Silver/Gold v1** y corriendo en Databricks (§2.3.1 paso 4 Hecho). Esto resume lo que salió y lo que hay que decidir/hacer. **Es el puente para abrir los siguientes frentes.**

> **🔧 Actualización 7-jun — ventana de cuarentena ampliada a 14–17 nov (antes 15–17).** El criterio original definía la ventana **solo por la etiqueta `purchase` rota** (15-nov = 0 compras, 16–17 volcado del backlog) → por eso el **14-nov se escapó**: tiene compras normales (22k) pero un **volcado de eventos** (2.86M views / 165k carts, *por encima del propio Black Friday*; conversión a la mitad, 0.77%). **Criterio nuevo: etiqueta rota _o_ volumen de eventos anómalo → 14–17 nov.** El **13-nov NO** entra (volumen normal: views 1.17× / carts 1.00×; su tasa algo baja es el régimen de inicios de noviembre). Impacto preliminar (serie diaria) sobre la base que veían Kelly/EDA: **−3.3% views, −6.5% carts** → la conversión sube algo y el abandono baja. **⏳ Los números marcados en §17.1, §6 y §13 (y los 6 CSV agregados de Kelly) quedan pendientes de refresco**: requieren re-correr `02_medallion` (repobla `label_window_corrupt` en la Gold de Sara) + `03_gold_agregada_bi_pyspark` (CSV de Kelly) + el EDA (nuevo HTML).

### 17.1 Hallazgos de negocio (base limpia · cuarentena 14–17 nov · 90.6M eventos / 58.6M unidades ⏳ a refrescar)
- **Funnel por unidad (base LIMPIA):** cart **3.93%**, conv **2.24%**, cierre **56.9%**, **abandono 43.1%** (58.6M unidades; 994k carritos abandonados). *(La ventana corrupta inflaba el abandono a 51.7% y deprimía el cierre a 48.3%; full-data en §7.)* El negocio es **electrónica**: conv **3.52%**, mayor volumen (20.7M unidades), y el mayor pool de carritos abandonados ≈ **$211M en juego** (de $283.6M totales; full-data inflaba electronics a $349.5M). **Concentración de revenue:** electronics **76.9%**, top-3 **87.3%**; el 50% del revenue lo hacen **50 productos** (0.1%). *(Tablero: los CSV por-categoría `agg_funnel_categoria`/`agg_revenue_en_juego` **excluyen `Unknown`** (~32%); el funnel y totales **globales** del titular —incl. `Unknown`, 58.6M unidades, 994k carritos, $283.6M— viven en `agg_funnel_global.csv`. Detalle en `reports/data/README.md`.)*
- **Dos palancas (síntesis §4.9):** (A) recuperar carritos de alta intención que no cierran (electronics; **Samsung+Apple = 68.6%** de carritos electronics, abandono ~39–40%); (B) retener al núcleo recurrente (**35.8%** de compradores = **73.9%** del revenue; ticket recurrente **$1,464** vs one-time **$289**; 2ª compra mediana **2.9 días** —37% ≤1 día—, **84.4%** a la misma categoría → cross-sell 15.6%).
- **El precio no es el freno**; la decisión es casi inmediata (mediana **2.2 min**).
- **Perfilado Gold (§5) — contraintuitivo y clave para Sara:** las features tienen correlación **débil y NEGATIVA** con la compra; los compradores **navegan menos** (deciden rápido). Son features **pre-carrito** (anti-fuga). **Multicolinealidad** alta (`total_views`≈`distinct_products_viewed` 0.92). → el poder predictivo está en no-linealidades/interacciones; **enriquecer features** en la pasada 2.
- **Flag `sin_navegacion_previa`:** 34.144 sesiones (0.15%), **tasa 55.2%** vs 6.0% global → decisión de Sara (§13): mantener con flag / excluir / segmento aparte.

### 17.2 Calidad de datos de noviembre (DIAGNOSTICADO — §6.1/§7.1) ⚠️
- **La ventana 14–17 nov está corrupta — criterio doble (etiqueta _o_ volumen de eventos).** El `purchase` está roto el **15–17** (15-nov = **0 compras** pese a 467k carritos; volcado del backlog el 16–17; **no** es duplicación). El **14-nov** se añadió (7-jun) por **anomalía de volumen de eventos**, no de etiqueta: 2.86M views y 165k carts —*por encima del propio Black Friday*— con compras planas (22k) y conversión a la mitad (0.77%); es el borde de entrada del mismo volcado de ETL. El criterio inicial miraba solo la etiqueta, por eso el 14 se había escapado. Coincide con un problema reportado por la comunidad del dataset (REES46). → **Cuarentenar 14–17 nov en la pasada 2** (etiqueta y/o eventos corruptos).
- **La divergencia Oct→Nov es mayormente REAL:** excluir la ventana baja el abandono de Nov de 60.9% → 52.0%, pero **no** lo reconcilia con octubre (31.9%). Es estacionalidad pre-fiestas + posible cambio en la captura de `cart` (a confirmar).
- **Black Friday (29-nov)** es pico **real pero modesto** (~+30%); el pico ×6 del 17-nov era el artefacto, no BF.
- **Contexto dataset (consenso comunidad, no oficial — REES46 bajo NDA):** tienda Rusia/CIS, precios en USD, `event_time` en UTC.

### 17.3 Decisión de split train/test — A CERRAR con Sara/equipo (documento en EDA §6.1·bis)
- **Aclaración:** la estratificación va en la **CV interna** (`StratifiedKFold`), **no** en la frontera temporal; la diferencia de tasa Oct↔Nov es **drift**, se maneja con calibración (doc 05 §1.4).
- **Recomendado: Opción C** — train = octubre + noviembre hasta ~23 / test = nov 24–30 (mismo régimen, *out-of-time*, alto volumen) → aísla *skill* del *drift* sin renunciar a noviembre. **Opción A** (train Oct / test Nov-sin-14–17) como **sensibilidad**. **Octubre-solo descartado** (renuncia al out-of-time). Invariante en todas: cuarentena 14–17, `StratifiedKFold`, calibración+Brier, `is_black_friday` solo para estratificar evaluación.
- **Preguntas abiertas para mañana:** ¿el shift es conductual o de medición (captura de `cart`)?; punto de corte exacto de C; ¿C sola o híbrida C+A?; tasa de etiqueta a nivel sesión por mes.
- **🔬 Hallazgo de régimen temporal de noviembre (VERIFICADO EN VIVO 5-jun · insumo del empalme con Sara, no es decisión tomada).** La serie diaria completa de noviembre (tasa de etiqueta a nivel sesión) muestra un **valle con recuperación**, no una degradación monótona: baja de ~0.060 (1-nov) al **fondo en 13–14 nov (0.042 / 0.033)**; sigue la **cuarentena 14–17** (excluida; 15≈0, 16=0.057, 17=0.155); y **desde el 18-nov se recupera** y estabiliza en ~0.048–0.055, con pico en **Black Friday 29 (0.061) y 30 (0.061)**. Implicación para la **Opción C** (test = nov 24–30): **test ≈0.056 vs train ≈0.059 → brecha ~5% relativa** (mismo régimen, *out-of-time* sano). El test **incluye Black Friday (29) y el día después (30)** → Sara debe **reportar métricas del test CON y SIN esos dos días** (estratificar la evaluación con `is_black_friday`), o el agregado mezcla dos regímenes. **Hipótesis de negocio** (registrada **explícitamente como hipótesis, NO como hecho**): *diferimiento de compra pre-Black-Friday* explicaría el valle 13–14. Evidencia defensiva y refinamiento del watch-item 13–14 nov (§17.5, §18.5).

### 17.4 Pendientes para el paso 5 (también listados en la última celda del EDA)
1. **Pasada 2 + calidad:** cuarentenar 14–17 nov; auditar categóricas/`"Unknown"`/outliers/bots; enriquecer features; resolver multicolinealidad.
2. **Cerrar el split con Sara** (doc §6.1·bis) y la decisión del flag `sin_navegacion_previa`.
3. **Congelar contrato Gold (§13)** + particionamiento (doc 02 §3) + exportar la Gold agregada para Power BI.
4. ~~**Limpiar** el Delta temporal de trabajo del EDA (`_tmp_eda_units`).~~ **Hecho (5-jun).**
5. **Alinear con Kelly** (las dos palancas y el mapa pregunta→gráfico, doc 07) y **coordinar la actualización de §6** (los números full-data difieren de los allí citados).

### 17.5 Pasada 2 — calidad de datos (RESUELTO · 4-jun · §2.3.1 paso 5 en curso)
Auditoría ejecutada sobre Silver/Gold completas. Hallazgo transversal: **los "problemas" de limpieza resultaron pequeños o inexistentes**; se documentan para no re-abrirlos.

- **Cuarentena 14–17 nov — materializada** (flag `label_window_corrupt` + `session_date` en Gold). Tasa de `purchase` por día: **15-nov = 0.0000** (suprimido), **16-nov = 0.0573**, **17-nov = 0.1555** (volcado del backlog, ×3); la etiqueta está *descuadrada en el tiempo*. **Ampliación 7-jun:** se añadió el **14-nov** por **anomalía de volumen de eventos** (no de etiqueta): 2.86M views / 165k carts —por encima de Black Friday— con compras planas → el *watch-item 13–14 nov* queda **resuelto para el 14** (se cuarentena). El **13-nov NO** se cuarentena: su volumen es normal (views 1.17× / carts 1.00×); su tasa algo blanda (0.042) es el régimen de inicios de noviembre, no corrupción. Cuarentena = excluir de train **y** test (§6.1·bis inv. 1). *(El flag se materializó para 15–17 en commit `15dc44d`; la extensión a 14–17 exige **re-correr `02_medallion`** para repoblar `label_window_corrupt`.)*
- **Categóricas / `"Unknown"` — NO backfilleable.** `category_id` es limpio (690 valores, **0 mapean a >1 macro**) y completo, pero **0 de 413** ids con macro `"Unknown"` tienen macro conocida en otra fila → el string no se puede re-derivar. **Decisión:** mantener `"Unknown"` como categoría legítima; para **ML usar `category_id`** (mejor señal que el string macro: 14 valores con 32% Unknown); `macro_category` queda para el **tablero** (Kelly), etiquetando "Unknown" como *"Sin taxonomía"* (~32% de eventos · `brand` Unknown ~14%). *Block 2: `categories_explored` debería pasar a `countDistinct(category_id)` (hoy cuenta "Unknown" como un valor más).*
  - **Consistencia de valores verificada (4-jun):** `macro_category`(14) / `sub_category`(59) / `item_type`(86) / `brand`(4304) **sin colisiones case/espacio (0)** ni sinónimos en las listas de baja cardinalidad → **no requiere normalización en Silver** (la taxonomía proviene de un único `category_code` controlado). Único typo de fuente: `sub_category='cartrige'` (consistente; se respeta el origen). **Con esto la limpieza queda cerrada.**
- **Outliers de precio — no hay.** `p999 = 2562`, `max = 2574`, todas las macro topan en ~2574 → precio **acotado en la fuente** (probable cap de REES46); con `price>0` ya filtrado en Silver, **no se winsoriza** (confirma "el precio no es el freno", §17.1).
- **Sesiones-bot — cola despreciable.** `p999 = 76` ev/sesión; **>200 ev = 506 sesiones (0.002%)**, >500 = 16. Impacto ~nulo → **no se añade flag** (revisado, no amerita acción).

---

## 18. Plan de arranque para Sara (Modelado) — recomendación

> **Para quién es esto:** si vas a empezar el frente de modelado y solo lees esta sección, debes quedar con el panorama completo —qué tienes, cómo conviene trabajar, dónde correr cada cosa, qué tienes que entregar y qué decisiones cerrar antes—. Es una **recomendación razonada**, no una orden: donde hay opción, se dice cuál se recomienda y por qué. Complementa el contrato Gold (§13) y los hallazgos del EDA (§17), que conviene leer primero.

### 18.1 De dónde arrancas (lo que ya está hecho por ti)

No tienes que tocar los 14 GB ni reconstruir nada: arrancas sobre la **Gold de sesión congelada** (§13) —22 columnas, grano *1 fila = 1 `user_session`*, **22.99M sesiones**, tasa de etiqueta 0.0610 full / 0.0597 base limpia (cuarentena 14–17)—. El EDA oficial ya dejó los hallazgos que informan tus features (§17), y conviene leerlos antes de modelar porque cambian la intuición:

- Las features conductuales tienen correlación **débil y NEGATIVA** con la compra: los compradores **navegan menos** y deciden rápido (mediana 2.2 min). Son señales **pre-carrito** (anti-fuga), no de "engagement".
- Hay **multicolinealidad** fuerte (`total_views` ≈ `distinct_products_viewed`, corr 0.92); por eso la Gold ya trae `revisit_intensity` para decorrelacionar el par.
- → el poder predictivo **no es lineal**: está en **interacciones y no-linealidades**. Un baseline logístico te da el piso; el GBM es donde esperas ganar.

El **set inicial de features** (anti-fuga, aditivo) está en el callout "📌 Para Sara" de §13. Es un punto de partida, no la palabra final: tú decides qué columna del par colineal soltar y qué *feature engineering* adicional necesita cada modelo.

### 18.2 La estrategia recomendada: modelado híbrido

La regla que organiza todo tu frente es **separar iterar de materializar el entregable**. Son dos modos con costos y lugares distintos:

- **Iterar (barato, muchas veces, fuera de cuota):** selección de modelo (logística → GBM), *sweeps* de hiperparámetros, curvas de calibración, análisis de umbral y la exploración del clustering. Esto se repite decenas de veces y **no debe quemar cuota de Databricks**.
- **Materializar el entregable (UNA sola vez, en Databricks):** el modelo elegido logueado en **MLflow** (params, PR-AUC/Brier, modelo **calibrado**, *signature*) y el **scoring batch** (sesión → score/segmento) que aterriza junto a la Gold y alimenta el tablero (Kelly) y la selección del segmento del A/B (Yeison).

Por qué híbrido y no "todo en Databricks": **(1) cuota** —iterar selección/calibración/clustering sobre serverless medido es justo lo que el proyecto evita (doc 02 §4)—; **(2) los cursos premian MLflow + scoring batch** —ya están en §7 y en el paso 6 del pipeline; renunciar a eso es regalar un punto de defensa—; **(3) reproducibilidad** —el scoring final usa el mismo *transform* que el entrenamiento—.

### 18.3 Dónde iterar — regla de decisión

El destino de la iteración depende de una **medición**, no de un gusto. Antes de decidir, exporta la Gold congelada a **Parquet con compresión `zstd`** —este snapshot vive **fuera del árbol versionado** del repo (no se commitea); los CSV de `reports/data/` son los **agregados para el tablero de Kelly**, no son tus datos de modelado— y mide su tamaño comprimido contra la RAM de tu máquina.

- **Opción A — local (recomendada si tu máquina aguanta).** Iteras sobre el snapshot con pandas / polars / duckdb, **cero Databricks**. Si el archivo entra en RAM, o lo consultas con duckdb sin cargarlo entero, esta es la vía más ágil y barata.
- **Opción B — Databricks sobre muestra estratificada (si hay fricción de setup o no entra en RAM).** Iteras en Databricks sobre una **muestra estratificada** que respete la tasa 0.061, y dejas el *fit* final + MLflow + scoring sobre *full data* para **una única corrida**.

En **ambas** opciones el entregable aterriza en Databricks, así que Databricks se toca **pocas veces**: exportar el snapshot una vez y scorear una vez. Eso es lo que cuida la cuota sin renunciar al entregable que evalúan.

> **Pendiente de empalme:** la forma exacta de exportar el snapshot (script + *loader* local) y, si eliges A, validar que entra en tu entorno, se cierra en el **plan de empalme contigo** (posterior a la auditoría de la capa de datos). No exportes ni recortes datos por tu cuenta hasta ese empalme, para no fijar el split antes de tiempo (§18.5).

### 18.4 El entregable que cierra tu frente

Tu frente cierra cuando existen tres cosas:

1. **Contrato 2 — salida del modelo (§13), que tú defines.** Qué entrega el scoring: `user_session`, **probabilidad calibrada**, **segmento**, formato y dónde se persiste (Delta junto a la Gold). Esto es lo que consume Kelly (tablero) y Yeison (segmento del A/B), así que **enúncialo temprano** aunque lo refines después.
2. **MLflow:** params, métricas (**PR-AUC**, **Brier**/calibración), el **modelo calibrado** y la *signature*. *(El **tracking** ya quedó **verificado operativo** —smoke test 5-jun, riesgo 3 cerrado—; tú implementas el **logging** del modelo. En serverless/Free Edition, llama `mlflow.set_tracking_uri("databricks")` y `mlflow.set_registry_uri("databricks-uc")` **antes** de `set_experiment`, o este falla con `CONFIG_NOT_AVAILABLE` — gotcha documentado en doc 02 §4.)*
3. **Scoring batch:** el modelo aplicado a toda la Gold, materializado.

### 18.5 Decisiones que debes cerrar antes del *fit* final

Tres inputs siguen **abiertos a propósito** y son tuyos —no son errores de la capa de datos, son decisiones de modelado—:

- **Split temporal (recomendado: Opción C).** Train = octubre + noviembre hasta ~23 / test = nov 24–30 (mismo régimen, *out-of-time*, alto volumen). **Opción A** (train Oct / test Nov-sin-14–17) como **sensibilidad**. Invariantes en todas: **cuarentena 14–17 nov** (excluir de train *y* test), `StratifiedKFold` en la **CV interna** (no en la frontera temporal), **calibración + Brier**, e `is_black_friday` solo para **estratificar la evaluación** (no como feature). *Watch-item:* 13–14 nov salen blandos (tasas 0.042 / 0.033) → posible rampa de degradación del registro cerca de la frontera; tenlo en cuenta al fijar el corte exacto. Detalle en EDA §6.1·bis y §17.3.
- **Flag `sin_navegacion_previa` (§13).** Marca ~34.144 sesiones (0.15%) que abren con `cart`/`purchase` y, por el corte anti-fuga, tienen toda la navegación en 0 (y son ~55% positivas). Decide: **(A)** mantenerlas usando el flag como variable, **(B)** excluirlas del entrenamiento, o **(C)** tratarlas como segmento aparte. Impacto chico, pero a fijar antes de entrenar.
- **Métrica y umbral.** **PR-AUC + calibración (Brier), nunca accuracy** (desbalanceo). El umbral es una **política operativa** definida por el costo del error, **no** 0.5 (§5.5).

### 18.6 Guardarraíl de reproducibilidad (no opcional)

Tu *feature engineering* a medida —interacciones, *encoding* de `category_id` (target/frequency), codificación cíclica de `session_hour`, escalado— vive **dentro del pipeline del modelo** (un `Pipeline` de sklearn o una función versionada) y **no muta la Gold**. Solo se promueve a la Gold —y siempre **aditivo** (§13)— una feature si es **estable y compartida** (la usa también el tablero o el clustering). El beneficio es directo: el **mismo *transform*** aplica en el scoring batch sin recalcular features sueltas, y se elimina el riesgo de fuga accidental al no esparcir transformaciones por fuera del pipeline.

### 18.7 Tu alcance (lo que NO es tu tarea)

No estimas *uplift* ni causalidad (§5): el alcance es **propensión + clustering + diagnóstico + diseño de A/B test**. El clasificador predice *quién* compra; el clustering da *tipos* de visitante; y el **cruce clasificador × clustering es el insight** del proyecto (sáb 6, §15). No construyes recomendador ni pronóstico de demanda (revientan el alcance, §5.3).
