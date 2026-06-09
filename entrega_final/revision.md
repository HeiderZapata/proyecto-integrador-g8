# Revisión de entrega final — PI Grupo 8

**Fecha:** 2026-06-08
**Alcance:** `entrega_final/Informe_Final_PI_Grupo8_plantilla.docx` y `entrega_final/Arquitectura_Utilizada.jpeg`
**Estado:** hallazgos registrados — **nada editado todavía**.

> Nota general: tanto el informe como el diagrama están **bien hechos y son coherentes** con los docs del repo (`docs/00_estado_del_proyecto.md`, `docs/02_arquitectura_bigdata_y_databricks.md`, `reports/data/README.md`). Los hallazgos de abajo son afinamientos y correcciones puntuales, no reescrituras.

---

## A. Diagrama de arquitectura (`Arquitectura_Utilizada.jpeg`)

### A.1 Qué representa
Diagrama end-to-end del pipeline (la "Ilustración 2 / arquitectura implementada"), en 5 carriles:
1. **Ingesta** — Kaggle (CSVs Oct & Nov) → descarga manual/script → Volume `ecommerce_raw` (Unity Catalog).
2. **Procesamiento (replay Kappa)** — Spark Structured Streaming + Auto Loader (`cloudFiles`).
3. **Almacenamiento · Medallion** — Bronze → Silver → Gold (Delta) con sus reglas de partición/ZORDER.
4. **Consumo & Modelado (ML)** — Gold Agregada (Spark SQL), MLflow, Modelos de árbol (LightGBM elegido vs XGBoost/RF), split Train/Test, Predicciones (Scoring Batch en Delta).
5. **Visualización** — Power BI Desktop → Power BI Service (Tablero Ejecutivo).

### A.2 Coherencia con el repo
Muy coherente. Coincide al detalle en lo más delicado:
- Silver: **6.41 GB · 61 particiones · partición fecha + ZORDER categoría** → idéntico a doc 02 §3.
- Gold: **sin partición · `ZORDER(session_date, user_id)` · data-skipping** → exacto (doc 02 §3, doc 00 §7).
- Bronze partición por fecha, Auto Loader `cloudFiles`, replay Kappa, Volume Unity Catalog → correctos.
- LightGBM elegido vs XGBoost/RF, MLflow, Predicciones en Delta (scoring batch) → consistentes.
- Test = 24–30 nov (holdout final), cuarentena 14–17 nov → correctos.

### A.3 Inconsistencias

- **🔴 PRINCIPAL — el límite del Train está mal.** El diagrama dice **Train = "oct + nov ≤ 13"**; los docs fijan la **Opción C: Train = oct + nov ≤ 23** (doc 00 líneas 34/36: *"train = oct + nov ≤ 23 / test = 24–30 nov"*; train 16.97M/0.0603). Con "≤ 13" + cuarentena 14–17 + test 24–30, **los días 18–23 nov (6 días) desaparecen** del diagrama (en realidad esos días SÍ están en train). → Corregir a **"oct + nov ≤ 23 (excl. cuarentena 14–17)"**.
- **🟡 Baselines omitidos.** Salta directo a "Modelos de árbol"; el frente va Dummy → logística → familias de árbol. Simplificación, no error.
- **🟡 Falta `Trigger.AvailableNow`** en el carril de Procesamiento (es la pieza que concreta el replay Kappa en serverless). Solo aparecen Structured Streaming + cloudFiles.
- **🟡 Gold Agregada sin el paso CSV.** El flujo real es Gold agregada → **12 CSV en `reports/data/`** → Power BI; el diagrama dibuja "Gold Agregada → Importar → Power BI" sin ese paso. Además la flecha punteada hacia Gold Agregada arranca visualmente a la altura de **Bronze** (se deriva conceptualmente de la **Gold de sesión**).

---

## B. Informe final (`Informe_Final_PI_Grupo8_plantilla.docx`)

> Documento bien escrito y bien estructurado. Ortografía correcta (sin typos ni errores de acentuación evidentes). Los hallazgos son de estilo/formato, estructura, claridad y cifras.

### B.1 Ortografía, gramática y redacción

- **Frase incompleta/agramatical (tabla §3.3.4, fila C2):** *"106 sesiones de días: hallazgo de calidad de datos"* — "sesiones de días" está truncado o mal construido. Completar la idea.
- **Formato de números inconsistente (ES vs EN):** coma inglesa de miles (`$1,460`, `$1,024`, `$257`) junto a punto decimal (`$250.4 M`, `3.86 %`, `2.2 min`). Unificar un solo criterio (el `reports/data/README.md` ya lo pedía).
- **"revenue" / "ingreso" indistintos** (76.9 %, 73.7 % "revenue" vs "ingreso por visitante"). Elegir uno.
- **Anglicismos sin cursiva uniforme:** *funnel, clickstream, boosting, bagging, leakage, scoring, drill, toggle, slicers, field parameters, scatter, treemap, heatmap, replay, checkpoint*. Aceptables en un informe técnico, pero (a) sin cursiva consistente y (b) **"targetear"** y **"guardarraíl"** (calco de *guardrail*) suenan informales → preferir "focalizar/segmentar" y "métrica de salvaguarda".
- **Redundancia menor:** título de §4 *"Tecnología: ingeniería de datos y uso de tecnología"* repite "tecnología".
- **Registro coloquial:** "vara mínima" (rol del Dummy), "pitch de negocio".

### B.2 Estructura y secciones faltantes

- **Tabla de contenido vacía:** el `Heading1` "Contenido" no tiene TOC generado (campo de Word sin actualizar).
- **Placeholders sin rellenar:**
  - Integrantes: *"Kelly [apellido], Sara [apellido], … Yeison [apellido]"* (solo Heider Zapata completo).
  - **3 marcas `[ POR COMPLETAR ]`:** enlace público del tablero (§5.4), confirmación v2 con matriz de oportunidad (§5.4), y referencias APA (§7).
  - **§7 Referencias incompleta:** falta URL/fecha de acceso del dataset, versiones de librerías y citas APA (PR-AUC, calibración, K-Means, diseño de experimentos).
- **Falta el diagrama de arquitectura embebido:** §4 describe la arquitectura solo con viñetas; **no incrusta ninguna ilustración**, pese a que `Arquitectura_Utilizada.jpeg` existe en la misma carpeta. *(Si se incrusta, corregir antes el split "≤ 13" — ver A.3.)*
- **§3.1 demasiado delgada:** tres viñetas que remiten a otras secciones.
- **Título de §2 engañoso:** "Marco teórico **y referencias**", pero las referencias son la §7 aparte.
- **Sin resumen/abstract ejecutivo** (verificar si lo exige `criterios_evaluacion_pi.docx`).

### B.3 Claridad y coherencia de ideas

La narrativa central (problema → baseline → métrica → validación → umbral → decisión; propensión ≠ uplift; las dos palancas; el cruce clasificador × clustering) es clara y consistente. A pulir:

- **"Conversión" con denominadores distintos sin avisar siempre:** 2.27 % (producto-en-sesión, funnel), 5.97 % (sesión, tasa de etiqueta), 3.58 % (electrónica, producto-en-sesión), 8.4 % (C3, sesión). En la tabla de clusters dice solo "Conversión 8.4 %" sin recordar la unidad → un lector puede chocar "electrónica 3.58 %" contra "C3 electrónica 8.4 %". Añadir nota que reconcilie.
- **El umbral operativo nunca se cuantifica:** §3.3.4 y §3.5.3 lo declaran "política operativa" / "central", pero no se da el valor ni su efecto (en los docs: F1-óptimo ≈ 0.092 → ~15 % de sesiones marcadas, precision ~12 % vs base ~5.6 %, lift ~2.2×).
- **§3.5.6 — "elegibles" ignora el filtro de propensión:** estima *"~10⁵ visitantes-C3 elegibles/día"* sobre **todo** C3, pero §3.5.3 dice que dentro de C3 el modelo define el subconjunto elegible (menor). Sobreestima el flujo elegible y la duración.
- **§3.5.4 — unidad de aleatorización (visitante) vs unidad del segmento (sesión):** target/modelo/C3 son por sesión; el A/B aleatoriza por visitante. Falta una frase que cierre el salto sesión→visitante (un visitante puede tener sesiones dentro y fuera de C3).

### B.4 Inconsistencias internas (cifras)

- **🔴 "17 features" no cuadra con su descripción** (§3.3.1): *"17 features: las 12 conductuales + día de la semana + fin de semana + hora"*, con **solo la hora** en seno/coseno → 12 + 1 + 1 + 2 = **16**, no 17. Falta una feature o el día de la semana también es cíclico (no se dijo). Reconciliar.
- **🔴 C2: "0.04 % del tráfico" vs "106 sesiones"** (§3.3.4): sobre 22.99 M sesiones, 0.04 % ≈ 9.200 sesiones, no 106 (106 ≈ 0.0005 %). Solo cuadra si el clustering corrió sobre una **muestra** (~265 k), que el documento no menciona. Aclarar la base o corregir.
- **🟡 Conteo de eventos: 109.5 M vs 109.95 M:** §1 y §3.2.1 dicen "109.5 millones de eventos"; §4.1 dice "109.95 M filas ingeridas sin duplicar". Misma magnitud, dos valores. Unificar o explicar (crudo ingerido vs eventos tras limpieza).
- **🟡 Conteo de sesiones limpias implícito:** se da 22.99 M (base completa) pero nunca el total **limpio** (19.71 M); solo se deduce sumando train 16.97 M + test 2.74 M (§3.3.3). Enunciarlo para mayor claridad.

**Verificaciones que SÍ cuadran:** Dummy 0.056 = tasa base del test (0.0560) ✓; tasa limpia 0.0597 = promedio ponderado train 0.0603 / test 0.0560 ✓; "2.1× vs Dummy" = 0.117/0.056 ✓; XGB 0.1185 vs LGBM 0.1184 → "diferencia 0.0001" ✓; tuneo 0.118→0.124 ✓; MDE relativo +10 % → 8.4 %→9.24 % ✓; suma de % de clusters ≈ 100 ✓; $187 M / $250 M ✓.

---

## C. Prioridades de corrección

1. **Cifras que no cuadran:** "17 features" (§3.3.1) y "0.04 % vs 106 sesiones" (§3.3.4).
2. **Diagrama:** corregir Train **"≤ 13" → "≤ 23 (excl. 14–17)"** (error factual del jurado de HPC).
3. **Placeholders/entregables:** apellidos, los 3 `[ POR COMPLETAR ]`, referencias APA, TOC vacía.
4. **Cuantificar el umbral operativo** (lo declaran central y no está el número).
5. **Incrustar el diagrama** en §4 (tras corregir el split).
6. **Pulido:** unificar formato de cifras (coma de miles), "conversión" con unidad explícita en clusters, 109.5 vs 109.95 M.
