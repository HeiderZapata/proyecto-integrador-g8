# Auditoría de la capa de datos (Medallion + EDA) — 2026-06-05

**Auditor:** Yeison (Claude Code) · rama `feat/auditoria-datos`
**Alcance:** verificar que la capa Medallion y el EDA están **cerrados sin errores ni descuidos**
que comprometan (a) el modelado de Sara y (b) el tablero de Kelly.
**Método:** auditoría **estática** del código de los notebooks y de los 6 CSV agregados, contrastada
contra el contrato §13, los hallazgos §17 y el particionamiento doc 02 §3. **No hay conexión a
Databricks** en esta corrida → donde un criterio depende de números en vivo, se traza la lógica y se
marca **«requiere corrida en Databricks para confirmar»**. La regla de cuota se respetó (cero re-escaneo
de los 14 GB).

> **Veredicto global:** la capa está **sólida y defendible**. La lógica anti-fuga, el grano, la
> cuarentena, el esquema y el particionamiento son **correctos en código**. **No hay bloqueantes.**
> Hay **2 hallazgos ámbar** (consistencia de cifras BI ↔ titular, e idempotencia/limpieza pendiente) y
> varios menores, todos con fix concreto. Ninguno frena el arranque de modelado/tablero hoy.

> ## ✅ ACTUALIZACIÓN (5-jun, post-acciones) — los 2 ámbar quedaron RESUELTOS
>
> Tras correr/aplicar los fixes: **(5)** se generó **`agg_funnel_global.csv`** y cuadra **exacto** con el
> titular (58.598.189 unidades · cart 3.93% · conv 2.24% · abandono 43.14% · 994.150 carritos · $283.6M);
> el README documenta la exclusión de `Unknown`; y se corrigió el claim falso de `src/funnel.py` en el EDA.
> **(8)** se borraron los Delta `_tmp*` del catálogo y se puso `FORCE_REBUILD_UNITS=False`.
> **Queda 1 sola recomendación opcional:** hacer determinista el `user_id` de sesiones multi-usuario
> (requiere re-correr la Gold; aditivo-seguro). **Con eso, los 9 checks quedan en verde.**

---

## Verificación en vivo (5-jun, post-auditoría estática)

La auditoría original (§1–§9, abajo) fue **estática**: trazó la lógica del código de los notebooks sin
conexión a Databricks. **Hoy 5-jun se corrió la verificación en vivo** sobre la Gold real
`/Volumes/workspace/default/e_commerce/gold/features_session` (Databricks serverless, Free Edition; solo
agregados sobre la Gold ~1.33 GB y metadata, **cero re-escaneo de los 14 GB**). Notebook usado:
`notebooks/analysis/_verif_post_audit.ipynb`. **Todos los checks que dependían de números en vivo
quedaron confirmados.**

| Check | Esperado (estático) | Medido en vivo (5-jun) | Resultado |
|---|---|---|:---:|
| **6 — esquema** | 22 columnas del contrato §13 | **22 columnas exactas** al contrato | ✅ |
| **2 — grano** | filas == sesiones distintas, 0 dup | **22.995.676 filas = sesiones distintas, 0 duplicados** | ✅ |
| **1+3 — tasa de etiqueta** | full ~0.0610 / limpia ~0.0589 | **0.0610 full · 0.0589 base limpia** (`label_window_corrupt=0`) | ✅ |
| **1 — flag `sin_navegacion_previa`** | ~34.144 sesiones, ~55% positivas | **34.144 sesiones, 55.2% positivas** | ✅ |
| **4 — `categories_explored_cid`** | poblada, distinta del macro, sin nulos | **avg 1.32 vs macro 1.14, 0 nulos** | ✅ |
| **7 — particionamiento físico** | Bronze/Silver por fecha; Gold sin partición, sin micro-archivos | **Bronze [event_date] 61 arch ~4.31 GB · Silver [date] 61 arch ~6.41 GB · Gold partición [] 6 arch ~1.33 GB (~222 MB/arch, cero micro-archivos)** | ✅ |
| **8 — idempotencia** | sin Delta `_tmp*` en gold/ | **gold/ sin `_tmp*`** (catálogo limpio) | ✅ |
| **MLflow tracking** | setup operativo | **smoke test OK** tras `set_tracking_uri("databricks")` + `set_registry_uri("databricks-uc")` (ver doc 02 §4) | ✅ |

**=> El "riesgo 3" (físico de particionamiento pendiente de corrida + MLflow setup) queda CERRADO EN VIVO.
La capa está lista para modelado/tablero sin pendientes de corrida.**

### Hallazgo de régimen temporal de noviembre (insumo del split de Sara)

La serie diaria completa de noviembre (tasa de etiqueta a nivel sesión) muestra un **valle con recuperación**,
no una degradación monótona: baja de ~0.060 (1-nov) al **fondo en 13–14 nov (0.042 / 0.033)**; sigue la
**cuarentena 15–17** (excluida; 15≈0, 16=0.057, 17=0.155); y **desde el 18-nov se recupera** y estabiliza en
~0.048–0.055, con pico en **Black Friday 29 (0.061) y 30 (0.061)**. Implicación para la **Opción C**
(test = nov 24–30): **test ≈0.056 vs train ≈0.059 → brecha ~5% relativa** (mismo régimen, *out-of-time* sano).
El test **incluye Black Friday (29) y el día siguiente (30)** → la evaluación debe **estratificarse CON y SIN**
esos dos días (`is_black_friday`) o el agregado mezcla dos regímenes. **Hipótesis de negocio** (explícitamente
**hipótesis, no hecho**): *diferimiento de compra pre-Black-Friday*. Esto **refina** el watch-item 13–14 nov ya
registrado (doc 00 §17.5/§18.5); el split **sigue abierto con Sara** — es insumo/evidencia, no decisión tomada.

> **Nota:** esta verificación **no** modifica el contrato §13 ni las cifras §17 (la auditoría ya las dio por
> correctas: 0.0589 base limpia, etc.); solo **confirma en vivo** lo que la auditoría estática había trazado en
> código. Los marcadores `†` (físico pendiente) y las frases "requiere corrida" de §1/§3 se actualizan abajo a
> "confirmado en vivo 5-jun".

---

## 1. Resumen ejecutivo — semáforo por check

| # | Check | Semáforo | Una línea |
|---|---|:---:|---|
| 1 | Anti-fuga (corte por primer cart/purchase) | 🟢 | Corte determinista `event_time < t_crit`; las 12 conductuales se agregan solo sobre `df_pre`. Sin gotcha RANGE en producción. |
| 2 | Grano e integridad (1 fila = 1 sesión) | 🟢* | Grano garantizado por `groupBy(user_session)` + `LEFT join`. *Menor:* el `user_id` de sesiones multi-usuario se resuelve con `first()` **no determinista** (sin documentar/ordenar). |
| 3 | Cuarentena 15–17 nov (etiqueta) | 🟢 | `label_window_corrupt` + `session_date` presentes; CSV diario confirma 15-nov=0 compras; watch-item 13–14 nov registrado para Sara. |
| 4 | Categóricas (`_cid` vs macro) | 🟢 | `categories_explored_cid = countDistinct(category_id)` poblada; macro se conserva para tablero; `Unknown` consistente. |
| 5 | Consistencia de cifras (titular ↔ §17 ↔ CSV) | 🟠→🟢 | Los CSV por-categoría **excluyen `Unknown`** → no reproducían el titular. **Resuelto:** `agg_funnel_global.csv` (incl. Unknown) cuadra exacto + README documenta + claim de `src/funnel.py` corregido. |
| 6 | Esquema vs contrato §13 (22 cols) | 🟢 | La Gold materializa **exactamente** las 22 columnas del contrato; evolución aditiva. |
| 7 | Particionamiento (doc 02 §3) | 🟢† | Código correcto: Bronze/Silver por fecha (+ZORDER cat_id), Gold sin partición +ZORDER(session_date,user_id). †Físico **confirmado en vivo 5-jun** (ver §Verificación en vivo): Bronze [event_date] 61 arch · Silver [date] 61 arch · Gold partición [] 6 arch ~222 MB, cero micro-archivos. |
| 8 | Reproducibilidad / idempotencia | 🟠→🟢 | Writes `overwrite` (idempotentes) y sin secretos. **Resuelto:** `_tmp*` borrados, `FORCE_REBUILD_UNITS=False`. Queda solo la recomendación opcional del `user_id` determinista. |
| 9 | Higiene de cuota en el EDA | 🟢 | El EDA lee de Silver/Gold vía "Capa de agregados" (Spark→pandas); ningún gráfico re-escanea Silver; distribuciones sobre muestra 3% (semilla 42). |

`*` verde con nota menor · `†` verde en código; **físico confirmado en vivo 5-jun** (ver §Verificación en vivo).

**Conclusión:** **apto para soltar modelado (Sara) y tablero (Kelly) hoy.** Los ámbar son de
*presentación/consistencia y limpieza*, no de corrección de la Gold; se corrigen sin re-congelar el
contrato.

---

## 2. Tabla de hallazgos

| Sev | Check | Hallazgo | Evidencia (archivo : celda/CSV) | Fix propuesto |
|---|---|---|---|---|
| **Ámbar** | 5 | **Los CSV por-categoría excluyen `Unknown` (~32% de unidades) → no reproducen el titular.** Sumando `agg_funnel_categoria` se obtiene **cart 4.40% / conv 2.54% / abandono 42.35%** sobre **38.6M** unidades, no el titular **3.93/2.24/43.1** sobre **58.6M**. Igual con carritos abandonados (**720k** CSV vs **994k** titular) y revenue en juego (**$254.5M** CSV vs **$283.6M** titular). No existe ningún CSV con el **funnel global** ni con el total en-juego. | `reports/data/agg_funnel_categoria.csv`, `agg_revenue_en_juego.csv`; `eda_ecommerce.ipynb` celda 17 (`global_funnel_spark`, incl. Unknown) y celda 19 (`rev_aband_total` global) vs celda 33 (titular 58.60M) | **(aplicado, pendiente de correr)** añadir `agg_funnel_global.csv` (con fila TOTAL incl. `Unknown`) en `03_gold_agregada_bi.ipynb`; **(aplicado)** documentar la exclusión de `Unknown` en `reports/data/README.md` para que Kelly no intente cuadrar los per-categoría con el titular. |
| **Ámbar** | 5 | **Claim falso de código compartido.** El EDA afirma *"La lógica vive en `src/funnel.py` … compartida con el dashboard para que ambos reporten exactamente lo mismo"*. **No existe `src/` ni `funnel.py`** en el repo; el funnel se define **inline** en el EDA (celda 17) y se **re-implementa** en `03` (que sí admite "se replican las definiciones"). Riesgo de defensa: un profesor que abra el repo no encontrará el módulo. | `eda_ecommerce.ipynb` celdas 15/17/30; `find . -name "*.py"` → vacío; `03_gold_agregada_bi.ipynb` celda 0 | **(parche listo en §7, pendiente de aplicar a mano)** corregir el comentario del EDA para reflejar la realidad (definiciones replicadas, no módulo compartido). Opcional post-entrega: extraer `src/funnel.py` real e importarlo en ambos. |
| **Ámbar** | 8 | **`_tmp_eda_units` no se borra.** En `03` la línea de limpieza de `_tmp_eda_units` está **comentada** (solo se borra `_tmp_bi_units`). Queda un Delta temporal residual en el Volume. | `03_gold_agregada_bi.ipynb` celda 15 (línea comentada); `eda_ecommerce.ipynb` celda 96 ítem 5 | **(requiere corrida)** ejecutar una vez `dbutils.fs.rm("/Volumes/workspace/default/e_commerce/gold/_tmp_eda_units", recurse=True)` y dejar registro. Es el único pendiente explícito del cierre de pasada 2. |
| Menor | 2 | **`user_id` de sesiones multi-usuario no determinista.** `02b` detectó `user_session` con >1 `user_id`. La Gold lo resuelve con `F.first("user_id", ignorenulls=True)` **sin `orderBy`** → conserva un `user_id` **arbitrario**; re-correr podría conservar otro. El grano (1 fila/sesión) **sí** queda garantizado (el `join` no multiplica). | `02_medallion.ipynb` celda 13 (`df_session`) | **(recomendado, requiere re-correr Gold)** documentar la regla ("se conserva un `user_id` no nulo arbitrario") **o** hacerlo determinista (`F.min("user_id")` o `first` con `Window.orderBy("event_time")`). Aditivo-seguro; opcional antes de entrega por costo de re-corrida. |
| Menor | 8/9 | **`FORCE_REBUILD_UNITS=True`** deja el EDA re-escaneando Silver en cada corrida (se puso en `True` para el rebuild único de la cuarentena, 4-jun). | `eda_ecommerce.ipynb` celda 16 | **(parche listo en §7, pendiente de aplicar a mano)** volver a `False` (ya se hizo el rebuild); el reuso del Delta temporal cuida la cuota. |
| Menor | 8 | **Delta temporal de diagnóstico** `_tmp_diag/silver_sample` queda en el Volume tras `02b`. | `02b_diagnostico_gold_join.ipynb` celda 2 | **(requiere corrida)** borrarlo al cerrar (`dbutils.fs.rm(..., recurse=True)`); o aceptarlo como scratch (no contractual). |
| Nota | 5 | El EDA §5 (perfilado/multicolinealidad 0.92) se hizo sobre el **set original de 5 features**; las 8 features de pasada 2 (`revisit_intensity`, etc.) se añadieron **después** por recomendación del EDA y **no están perfiladas/correlacionadas** en el notebook. La afirmación "`revisit_intensity` decorrelaciona el par 0.92" es **intención de diseño no verificada en el EDA**. | `eda_ecommerce.ipynb` celdas 72–75 (FEATS = 5 originales) | Verificación es territorio de Sara (modelado). Dejar como watch-item: que Sara confirme la decorrelación en su matriz de correlación antes del fit. **No es error de la capa.** |

---

## 3. Detalle por check (verificación → criterio → resultado)

### Check 1 — Anti-fuga 🟢 (lógica correcta; requiere corrida para confirmar conteos)
**Verificación.** `02_medallion.ipynb` celda 13:
1. `t_crit` = `min(event_time)` de eventos `cart`/`purchase` por `user_session`.
2. `df_pre = silver.join(t_crit, "left").filter(t_crit.isNull() OR event_time < t_crit)` → **estrictamente
   anterior** al primer evento crítico; sesiones sin crítico conservan todos sus eventos.
3. **Las 12 conductuales** (`total_views`, `distinct_products_viewed`, `brands_compared`,
   `categories_explored`, `categories_explored_cid`, `browsing_duration_sec`, `avg_price_viewed`,
   `max_price_viewed`, `electronics_view_share`) se agregan sobre **`df_pre`**; las derivadas
   (`revisit_intensity`, `views_per_minute`, `avg_inter_event_sec`) salen de esos agregados pre-corte.
4. `target_purchase` = `max(purchase)` sobre la **sesión completa** (correcto: es la etiqueta, no una feature).
5. `session_start_ts = min(event_time)` de la sesión completa = **primer evento** (necesariamente ≤ corte) →
   features de calendario (`session_date/hour/day_of_week/is_weekend/is_black_friday`) legítimas (se conocen
   al inicio de la sesión, no son fuga).
6. `sin_navegacion_previa = total_views.isNull()` + `fillna(0)` para sesiones sin eventos pre-corte.

**Gotcha RANGE.** El `sum().over(Window.orderBy(...))` con frame RANGE (que excluiría vistas del *mismo
segundo*) vive **solo en el diagnóstico `02b`** (celda 4, reproducción del bug). **Producción no lo usa**:
usa el corte con `min` + comparación escalar `<`, inmune al frame. El `<` estricto excluye además la vista
del *mismo segundo* que el corte (dirección **conservadora**, anti-fuga segura).

**Criterio de aceptación.** ✅ Ninguna feature incluye el evento de corte ni posterior; sesiones sin
navegación previa → `sin_navegacion_previa=true` y features en 0. **Cumple.**
*Confirmado en vivo 5-jun:* `tasa_gold = 0.0610 full / 0.0589 limpia` y `sin_navegacion_previa` = 34.144
sesiones (55.2% positivas), medidos contra la Gold real (ver §Verificación en vivo).

### Check 2 — Grano e integridad 🟢 (nota menor)
**Verificación.** `df_session` y `df_features` agrupan **solo por `user_session`** → 1 fila/sesión por
construcción; el `LEFT join` desde el universo (`df_session`) no puede multiplicar filas aunque una sesión
tenga >1 `user_id`. La celda QA (15) asienta `n_rows == n_sess and n_dup == 0`. La resolución del caso
`user_session` con >1 `user_id` (detectado en `02b`): **se agrupa por `user_session` y se lleva `user_id`
con `first(ignorenulls)`** → el join ya no duplica.

**Hallazgo menor.** *Cómo* se conserva el `user_id` no está documentado y es **no determinista** (sin
`orderBy`). Ver tabla de hallazgos. El grano **no** se ve afectado.

**Criterio.** ✅ Aserciones de grano pasan (por construcción). *Confirmado en vivo 5-jun:* **22.995.676 filas
= sesiones distintas, 0 duplicados**; tasa 0.0610 full / 0.0589 limpia (ver §Verificación en vivo). El `*` y la
nota del `user_id` no determinista **se conservan** (sigue siendo recomendación opcional vigente, no ejecutada).

### Check 3 — Cuarentena 15–17 nov 🟢
**Verificación.** Gold (celda 13) crea `session_date` y `label_window_corrupt = session_date.between(15,17
nov)`. `agg_metricas_diarias.csv`: **15-nov purchases = 0** (con 466k carritos), **16-nov = 68.247**,
**17-nov = 185.195** (volcado). Las tasas **a nivel sesión** (15=0.0000, 16=0.0573, 17=0.1555) están
documentadas en doc 00 §17.5. **Watch-item 13–14 nov** (0.042/0.033) registrado para el split de Sara
(§17.5 y §18.5); la ventana **no se expandió** unilateralmente. La cuarentena es **filtro** (no borrado):
la Gold conserva las filas con el flag.

**Criterio.** ✅ Flag correcto + nota registrada. **Cumple.**

### Check 4 — Categóricas 🟢
**Verificación.** `categories_explored_cid = countDistinct(category_id)` (celda 13) — poblada y distinta de
la macro donde hay `Unknown` (la macro colapsa todas las categorías "Unknown" en un solo valor; `_cid` no).
`categories_explored` (macro) se conserva por el contrato/tablero. `Unknown` se imputa en Silver
(`fillna brand/macro_category`) y se etiqueta "Sin taxonomía" en BI. Consistencia de valores verificada en
§17.5 (sin colisiones case/espacio).

**Criterio.** ✅ `_cid` difiere de la macro donde hay `Unknown`, ambas no-nulas. **Cumple.**

### Check 5 — Consistencia de cifras 🟠→🟢 — TABLA DE RECONCILIACIÓN

**Fuente de verdad fijada: BASE LIMPIA** (cuarentena 15–17 nov), unidad = producto-en-sesión para el funnel.
*Actualizada tras generar `agg_funnel_global.csv` (5-jun): todas las filas cuadran.*

| Métrica titular | EDA §17 / titular (incl. `Unknown`, 58.6M) | Derivable de los CSV | ¿Cuadra? | Comentario |
|---|---|---|:---:|---|
| Funnel global — cart rate | **3.93%** | **3.93%** (`agg_funnel_global.csv`) | ✅ | el CSV global (incl. Unknown) cuadra; sumar el per-categoría daría 4.40% (excl. Unknown) |
| Funnel global — conv rate | **2.24%** | **2.24%** (`agg_funnel_global.csv`) | ✅ | idem (per-categoría daría 2.54%) |
| Funnel global — abandono | **43.1%** | **43.14%** (`agg_funnel_global.csv`) | ✅ | idem (per-categoría daría 42.35%) |
| Carritos abandonados (total) | **994k** | **994.150** (`agg_funnel_global.csv`) | ✅ | idem (per-categoría daría 720k) |
| Revenue en juego (total) | **$283.6M** | **$283.622.918** (`agg_funnel_global.csv`) | ✅ | idem (per-categoría daría $254.5M) |
| Revenue en juego — electronics | **~$211M** | **$211.1M** (`agg_revenue_en_juego`) | ✅ | coincide exacto |
| Electronics — conv rate | **3.52%** | **3.52%** (`agg_funnel_categoria`) | ✅ | coincide |
| Samsung+Apple = % carritos electronics | **68.6%** | **68.6%** (853.708 / 1.244.360) | ✅ | coincide |
| Samsung/Apple abandono | **~39–40%** | **39.0% / 40.4%** (`agg_marca_electronics`) | ✅ | coincide |
| Recurrentes — % compradores / % revenue | **35.8% / 73.9%** | **35.8% / 73.9%** (`agg_segmentos_comprador`) | ✅ | coincide |
| Ticket recurrente / one-time | **$1.464 / $289** | **$1.463,88 / $289,12** | ✅ | coincide |
| Tasa etiqueta sesión (base limpia) | **0.0589** | **5.893%** buyer (`agg_tipologia_visitante`) | ✅ | coincide |
| Concentración revenue electronics 76.9% / top-3 87.3% | titular §17 | — (no hay CSV de revenue **comprado** por categoría) | ⚠️ | métrica distinta (revenue comprado, no en-juego); no exportada |

**Lectura.** Todo lo **por-categoría / marca / segmento** ya cuadraba al céntimo entre EDA y CSV. El **funnel
global y los totales** (carritos, $) **no** se reproducían sumando los CSV per-categoría porque estos
**excluyen `Unknown`** (58.6M titular vs 38.6M sin Unknown) — **resuelto** al exportar
`agg_funnel_global.csv`, que da el global **incl. Unknown** y cuadra exacto con el titular. La exclusión de
`Unknown` en los per-categoría es **por diseño** (no se puede poner "Sin taxonomía" en un treemap de
categorías) y queda **advertida** en `reports/data/README.md`. Pendiente menor (no bloqueante): la
concentración "revenue electronics 76.9% / top-3 87.3%" es revenue **comprado** (no en-juego) y **no tiene
CSV**; si Kelly la quiere en el tablero, se añade una tabla `agg_revenue_comprado_categoria`.

**Discrepancias y fix.**
- **5a (totales global) — RESUELTO:** `agg_funnel_global.csv` (fila TOTAL incl. `Unknown`: 58.598.189 unidades,
  3.93/2.24/43.14, 994.150 carritos, $283.622.918) **generado y commiteado**; cuadra con el titular. La
  exclusión de `Unknown` en los per-categoría está documentada en `reports/data/README.md`. ✅
- **5b (src/funnel.py) — RESUELTO:** comentario del EDA corregido (la lógica es inline, replicada en `03`); ver §7. ✅
- **§6 full-data:** revisado — el EDA usa **base limpia** en §4/§5/§6 (la cuarentena es transversal, celda
  13); **solo §7** recarga la base completa, y está **rotulado**. doc 00 §6 y §7 **etiquetan** explícitamente
  base-limpia vs full-data (abandono 51.7% full / 43.1% limpia). **No quedan números full-data sin rótulo.**

### Check 6 — Esquema vs contrato 🟢
**Verificación.** La Gold (celda 13) produce exactamente estas **22 columnas**, idénticas al contrato §13:
`user_session, target_purchase, user_id` (3) · `total_views, distinct_products_viewed, brands_compared,
categories_explored, categories_explored_cid, browsing_duration_sec, avg_price_viewed, max_price_viewed,
electronics_view_share` (9) · `revisit_intensity, views_per_minute, avg_inter_event_sec` (3) · `session_date,
session_hour, day_of_week, is_weekend` (4) · `sin_navegacion_previa, is_black_friday, label_window_corrupt`
(3). Tipos coherentes con el contrato (ints, floats con `round`, `date`, `bool`/`int 0/1`). Evolución
**aditiva** (las 8 nuevas se añaden; nada se renombró/eliminó respecto a la v1).

**Criterio.** ✅ Diff de schema (código) == contrato §13. **Cumple.** *Confirmado en vivo 5-jun:* la Gold real
materializa **exactamente 22 columnas** (ver §Verificación en vivo).

### Check 7 — Particionamiento 🟢 (código; físico pendiente)
**Verificación.** Bronze: `partitionBy("event_date")` (celda 5). Silver: `partitionBy("date")` +
`OPTIMIZE … ZORDER BY (category_id)` (celda 7). Gold: **sin** `partitionBy` + `OPTIMIZE … ZORDER BY
(session_date, user_id)` (celda 13). Coincide con §13 / doc 02 §3 (Gold ≪ 1 TB → clusterizar, no
particionar). doc 02 §3 ya cita evidencia medida (Silver 6.41 GB/61 particiones; Gold 1.33 GB/6 archivos).

**Criterio.** ✅ Implementado == especificado. **Cumple en código.** *Confirmado en vivo 5-jun* (`DESCRIBE
DETAIL`): Bronze [event_date] 61 arch ~4.31 GB · Silver [date] 61 arch ~6.41 GB · **Gold partición [] 6 arch
~1.33 GB (~222 MB/arch, cero micro-archivos)** — coincide con doc 02 §3 (ver §Verificación en vivo).

### Check 8 — Reproducibilidad / idempotencia 🟠
**Verificación.** Toda escritura usa `mode("overwrite")` (+`overwriteSchema`) → **idempotente** (re-correr
no duplica). El pipeline corre 01→02→(02b)→03 con rutas Delta estables. Convención de nombres/carpetas (§12)
respetada. Ingesta lee `kaggle.json` del Volume; **sin secretos en código** (`01` celda 2) y `.gitignore`
cubre `kaggle.json`/`*secret*`/`*.key`. Muestreos con **semilla fija** (`seed=42` en `02b` y en el 3% del EDA).

**Pendientes (ver hallazgos):** borrar `_tmp_eda_units` (línea comentada en `03`); `FORCE_REBUILD_UNITS=True`
deja re-escaneo (corregido a `False`); no-determinismo de `first(user_id)` y de `dropDuplicates`
(superviviente arbitrario, filas idénticas salvo linaje → impacto nulo).

**Criterio.** ✅ idempotencia, secretos, semillas. *Confirmado en vivo 5-jun:* **gold/ sin Delta `_tmp*`**
(catálogo limpio) y `FORCE_REBUILD_UNITS=False` (ver §Verificación en vivo) → el ámbar quedó **cerrado**.
Queda solo la recomendación opcional del `user_id` determinista (no ejecutada).

### Check 9 — Higiene de cuota en el EDA 🟢
**Verificación.** El EDA (celda 13–19) define una **"Capa de agregados"**: escanea Silver un número acotado
de veces, materializa la tabla `units` (producto-en-sesión) en un Delta temporal y baja **tablas pequeñas**
a pandas; las secciones 1–4 son **solo pandas/Matplotlib** sobre esos objetos. Confirmado: **ningún gráfico
re-escanea Silver**. §7 (diagnóstico) recarga la base completa **a propósito** (una pasada, autocontenida).
Distribuciones de la Gold sobre **muestra 3% (seed=42)**; estadística de etiqueta exacta sobre la tabla
completa.

**Criterio.** ✅ Se confirma. **Cumple.**

---

## 4. Decisiones pendientes con Sara/equipo (NO son hallazgos de auditoría)

Estas quedaron **abiertas a propósito** (doc 00 §13, §17.3, §18.5). No son errores de la capa:

1. **Split train/test.** Recomendado **Opción C** (train Oct + Nov→~23 / test Nov 24–30); **Opción A** como
   sensibilidad. Invariantes: cuarentena 15–17, `StratifiedKFold` interno, calibración+Brier,
   `is_black_friday` solo para estratificar evaluación. *Watch-item:* 13–14 nov blandos (0.042/0.033) al
   fijar el corte exacto. → cerrar con Sara.
2. **Flag `sin_navegacion_previa`** (~34.144 sesiones, ~0.15%, ~55% positivas): (A) mantener con flag, (B)
   excluir, (C) segmento aparte. → decisión de Sara antes del fit.
3. **Contrato 2 — salida del modelo** (`user_session`, prob. calibrada, segmento, persistencia): lo define
   Sara.

---

## 5. Qué actualizar en doc 00

- **§13 / §17:** **sin cambios de fondo** — el contrato y los hallazgos cuadran con el código y los CSV.
  Las cifras verificadas (0.0589 base limpia, electronics $211M, Samsung+Apple 68.6%, 35.8%/73.9%) son
  correctas.
- **Banner de seguridad:** **ya cerrado** — el doc (5-jun) registra que Heider expiró la llave Kaggle y que
  no hay secretos en repo/historial. Auditoría confirma: `01` lee del Volume, `.gitignore` cubre
  `kaggle.json`/secretos. **Pendiente menor cerrado.** ✓
- **Añadir (cross-ref):** nota de que los CSV BI **por-categoría excluyen `Unknown`**; el funnel global y
  los totales (994k carritos, $283.6M) viven aparte → ver `agg_funnel_global.csv` (una vez generado) y
  `reports/data/README.md`.
- **§17.5 / §18.6 (opcional):** registrar que la **decorrelación de `revisit_intensity`** sobre el par 0.92
  es intención de diseño **a verificar por Sara** (no está perfilada en el EDA, que usa el set original).

---

## 6. Cambios aplicados en esta rama (`feat/auditoria-datos`)

| Cambio | Archivo | Estado |
|---|---|---|
| Reporte de auditoría (este documento) | `docs/auditoria_capa_datos_2026-06-05.md` | ✅ aplicado |
| Nota de exclusión de `Unknown` + funnel global para Kelly | `reports/data/README.md` | ✅ aplicado |
| Celda `agg_funnel_global` (funnel + totales incl. `Unknown`) | `notebooks/pipeline/03_gold_agregada_bi.ipynb` | ✅ aplicado · ✅ **corrido** → `agg_funnel_global.csv` cuadra con el titular |
| Corregir claim falso de `src/funnel.py` "compartido" | `notebooks/analysis/eda_ecommerce.ipynb` (celdas 15/17/30) | ✅ **aplicado** (vía script JSON) |
| `FORCE_REBUILD_UNITS = True → False` (higiene de cuota) | `notebooks/analysis/eda_ecommerce.ipynb` (celda 16) | ✅ **aplicado** |
| Borrar `_tmp_eda_units` / `_tmp*` (Delta temporal) | Databricks (`dbutils.fs.rm`) | ✅ **hecho** (catálogo limpio) |
| `user_id` determinista en sesiones multi-usuario | `notebooks/pipeline/02_medallion.ipynb` | 💡 **recomendado, opcional** (requiere re-correr Gold; aditivo-seguro) |

> **Nota:** las correcciones del EDA se aplicaron con un script de reemplazo sobre el JSON del notebook
> (`eda_ecommerce.ipynb` excede el límite del editor para `NotebookEdit`). Son cambios de **comentario** y de
> **una constante** — sin efecto sobre datos ni sobre el esquema Gold. **Sincronizar con `git pull` en el Git
> folder de Databricks.**

**No se modificó la lógica de features de la Gold** (esquema CONGELADO §13). Los parches aplicados son de
documentación, higiene y export BI; ninguno re-congela el contrato.

---

## 7. Parches aplicados a `eda_ecommerce.ipynb` (✅ hechos, registro)

> Aplicados el 5-jun vía script de reemplazo sobre el JSON. Se documentan aquí para trazabilidad.

**Celda 16 (`id: b1-agg-units`) — higiene de cuota.** Se cambió la línea:
```python
FORCE_REBUILD_UNITS = True   # cuarentena: la base cambio a limpia -> rebuild 1 vez; luego puede volver a False
```
por:
```python
FORCE_REBUILD_UNITS = False  # auditoria 5-jun: rebuild de cuarentena ya hecho (4-jun) -> reusa el Delta temporal (cuida cuota). Pon True solo si Silver cambia.
```

**Celda 30 (`id: 24`, markdown) — claim falso de `src/funnel.py`.** Reemplazar el párrafo:
> *La lógica vive en `src/funnel.py` (`global_funnel`, `category_funnel`), compartida con el dashboard para que ambos reporten exactamente lo mismo.*

por:
> La lógica del funnel (`global_funnel_spark`, `category_funnel_spark`) está **definida inline** en la "Capa de agregados" (arriba) y se **replica** en `pipeline/03_gold_agregada_bi.ipynb` para los CSV del tablero (no existe un módulo `src/funnel.py`). El funnel **global** incluye `Unknown` (58.6M); el desglose **por categoría** lo excluye, así que sumar el per-categoría **no** reproduce el global — para el KPI global usar `agg_funnel_global.csv`.

**Celdas 15 (`id: b1-aggs-md`) y 17 (`id: b1-agg-funnel`) — comentarios "porta `src/funnel.py` a Spark".**
Cambiar esa frase por "definido inline (replicado en `pipeline/03` para el BI)". Es solo un comentario.

*(Estos tres cambios son de documentación/constante; ninguno altera datos ni el esquema Gold.)*
