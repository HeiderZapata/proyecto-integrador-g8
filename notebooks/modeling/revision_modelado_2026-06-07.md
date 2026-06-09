# Revisión del frente de modelado — para Sara

**Fecha:** domingo 7-jun-2026 · **Revisó:** Yeison (+ Claude Code) · **Contra:** pack del curso de ML (`docs/05_contexto_aprendizaje_automatico.md`, estándar del profe Terán).
**Alcance:** `01_extraccion_gold_a_parquet`, `02_modelado_propension`, `03_drift_split_diagnostico`, `04_clustering` (rama `feat/modelo`).

> **Lee esto primero:** el trabajo está **fuerte y bien alineado con lo que el profe premia**. No hay errores que invaliden el experimento. Esto es una lista para **blindarlo** antes de congelar resultados y montar el scoring/Contrato 2. Ninguna recomendación cambia las conclusiones; las hacen a prueba de balas. Las recomendaciones se entregan ordenadas por prioridad; tú decides cuáles aplicar (varias son de criterio).

---

## ✅ Lo que ya está sólido (no tocar — es justo lo que la rúbrica valora)

- **Cadena defendible completa:** Dummy (trampa del accuracy) → logística (baseline serio) → comparación de familias (RF/XGB/LGBM bajo misma CV, métrica y muestra) → LightGBM+Optuna → calibración → umbral. Es la secuencia de `docs/05` §4.
- **PR-AUC + Brier, nunca accuracy**, con la explicación del porqué.
- **Anti-fuga correcto:** `StratifiedKFold` solo en la CV interna; split temporal en la frontera; `is_black_friday` solo para estratificar la evaluación (bien resuelto que el flag solo marca el 29 → usas la fecha para 29–30).
- **Desbalanceo bien hecho:** `scale_pos_weight`/`class_weight` calculado **solo del train**, dentro del fit. Sin SMOTE-antes-del-split.
- **test ≈ CV (0.118 vs 0.123):** evidencia limpia de no-fuga / no-overfit.
- **Importancia por permutación** (lo que el profe reconoce; no depende de SHAP).
- **Clustering:** escalado obligatorio, k por accionabilidad con "cluster≠segmento" explícito, perfilado/nombrado/acción, cruce con propensión → C0.
- **Drift (`03`):** PSI de precio + conversión por ventana/diaria como evidencia objetiva de la Opción C.

---

## 🔴 Pendiente ya conversado (datos OK, falta el texto)

**P0. Cuarentena 14–17 nov — datos ya corregidos; queda alinear el markdown.**
- **Los datos ya están actualizados:** Yeison re-corrió `01_extraccion_gold_a_parquet` sobre la Gold sellada (cuarentena **14–17**, `user_id` determinista) y te pasó el nuevo snapshot. La tasa "honesta" de la base limpia pasa de **0.0589 (15–17)** a **0.0597 (14–17)**.
- **Pendiente al re-correr:** que el **markdown/narrativa** de los notebooks quede en **14–17**, no en "15–17". Hoy lo dicen en `15–17`:
  - `02_modelado_propension`: encabezado de decisiones, sección **§2 "Cuarentena anti-fuga (15–17 nov)"** y §3 (texto e invariantes).
  - `03_drift_split_diagnostico`: encabezado, banda roja `axvspan("2019-11-15","2019-11-17")` y comentarios.
  - `04_clustering`: §1 ("excluida la ventana corrupta 15–17 nov").
- Si no se alinea, el informe/defensa dirá "cuarentena 14–17" mientras el código narra "15–17" → contradicción fácil de detectar. (Los números se regeneran solos al re-ejecutar con el snapshot nuevo.)

---

## 🟠 Recomendaciones de defensa (el jurado puede picar aquí)

**R1. Corregir la afirmación de "no fuga a nivel de usuario" (`02`, §3 markdown).**
Dice que el split temporal *"impide que sesiones del mismo cliente se repartan entre train y test"*. **No es exacto:** un usuario activo en, p. ej., Nov 10 (train) y Nov 25 (test) cae en ambos conjuntos. → **No cambies el split; corrige el texto.** La justificación honesta y suficiente es que las features son **intra-sesión y pre-corte** (no hay fuga entre sesiones) y que ver usuarios recurrentes en el test es **realista** (producción también los ve). Evita reclamar una disjunción de usuarios que el split temporal no garantiza.

**R2. La validación con DBSCAN es sobre la proyección PCA 2D, no sobre el espacio completo (`04`, §8).**
DBSCAN corre sobre `P` (2 componentes) mientras KMeans se ajustó sobre las 12 features escaladas → no es un cross-check entre espacios distintos, y `eps=0.1` es sensible. → Opción A: correr DBSCAN sobre `Xs` (espacio completo) para un contraste real. Opción B: **suavizar la afirmación** de "robustez" a "consistente en la proyección de inspección" (el profe: *"una visualización abre una hipótesis, no la cierra"*).

**R3. Outliers de duración no tratados.**
El clustering destapó C3 = sesiones con `browsing_duration_sec` ≈ **15 días** (~1.3M s). Esas mismas sesiones rotas alimentan `browsing_duration_sec`, `views_per_minute` y `avg_inter_event_sec` como features en `02`. Los árboles lo toleran, pero la auditoría de calidad (doc 00 §17.5) solo cubrió bots por conteo y precio, **no duración**. → Acotar/winsorizar la duración (o excluir sesiones rotas con un umbral) **o** documentarlo explícitamente como decisión consciente.

**R4. "Selección por promedio Y variabilidad" se enuncia pero no se ejerce.**
El objetivo de Optuna devuelve la media de PR-AUC y `study.best_value` toma el **máximo de la media** — la variabilidad entre folds no entra en la selección, aunque el markdown (y el profe) la piden. → Reportar el **std de los folds del best trial** (o un top-N de trials con media ± std) para respaldar la frase. Es barato y es exactamente lo que Terán exige ("menor variabilidad puede ser mejor evidencia que un máximo aislado").

---

## 🟡 Pulido (bajo impacto)

- **R5. Features redundantes.** `categories_explored` está contaminada con `"Unknown"` y el doc 00 §17.5 pidió preferir `categories_explored_cid`; hoy van las dos. Igual `total_views`≈`distinct_products_viewed` (corr 0.92) y `avg_price_viewed`/`max_price_viewed`. Los árboles lo manejan, pero por limpieza/defensa conviene al menos soltar la macro contaminada. (En `04`, además, esa redundancia infla la dimensión "amplitud de navegación" de la PCA.)
- **R6. Optuna a 20 trials** (preliminar) → subir a **≥50** para el fit final (ya anotado en el propio notebook).
- **R7. `sin_navegacion_previa` aporta −0.0000** en permutación → decidir explícitamente: mantener / soltar / tratar como segmento (doc 00 §13) y dejarlo escrito.
- **R8. Higiene de rutas y etiquetas.** `DATA_DIR` relativo en `04` es frágil (usa el `_repo_root()` de `02`); y la numeración C0–C3 de KMeans depende de la semilla → **nombrar los segmentos por perfil**, no por índice (al re-correr con el snapshot nuevo pueden reordenarse).
- **R9. Black Friday.** Ya reportas con/sin (bien). Sugerencia: titular el **ex-BF ≈0.113** como el número "régimen normal honesto" y el global como ~0.118.

---

## 🛡️ Para la defensa (trade-offs correctos, ten la respuesta lista)

- **k=4 con silhouette 0.28** (k=2 daba 0.50): es deliberado y correcto según el criterio del profe (*"el mejor k no es el que maximiza una métrica, es el que sostiene una partición defendible"*). Refuérzalo mostrando **estabilidad** (re-correr con varias semillas o bootstrap), no solo con DBSCAN.
- **StratifiedKFold con shuffle dentro del train** (que abarca Oct–Nov): correcto — la honestidad temporal la da el **test out-of-time**, y el profe avala CV estratificada en el tuning. No es contradicción.

---

## Checklist sugerido antes de congelar el modelo

- [ ] Re-ejecutar `02`/`03`/`04` con el snapshot 14–17 y **alinear el markdown a 14–17** (P0).
- [ ] Corregir el texto de "fuga a nivel de usuario" (R1).
- [ ] Decidir DBSCAN sobre espacio completo o suavizar la afirmación (R2).
- [ ] Resolver outliers de duración o documentarlos (R3).
- [ ] Reportar media ± std del best trial de Optuna (R4) y subir a ≥50 trials (R6).
- [ ] Decidir `sin_navegacion_previa` (R7) y soltar features redundantes (R5).
- [ ] Cerrar `06_scoring_mlflow_databricks`: MLflow (params, PR-AUC/Brier, modelo calibrado, signature) + scoring batch + **Contrato 2** (`user_session`, prob. calibrada, segmento) en Delta.

> Cualquier duda sobre el porqué de una recomendación, está atada al estándar del profe en `docs/05` (frases-ancla en su §2 y §6). — Yeison
