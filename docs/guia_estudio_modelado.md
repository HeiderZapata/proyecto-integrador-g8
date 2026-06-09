---
title: "Guía de estudio — Auditoría del modelado"
subtitle: "Proyecto Integrador G8 · preparación de defensa (9-jun-2026)"
author: "Frente de modelado (Sara) — revisión Yeison + Claude Code"
date: "Estado: modelado, datos, tablero e informe CERRADOS"
---

> **Cómo leer esto.** Es la auditoría completa del frente de modelado, organizada en 6 bloques (Parte 0 a Parte 5) + síntesis. Cada parte responde "qué se hace, por qué, cómo se defiende". Las cifras están **verificadas contra el repo final**. Los matices marcados **(ojo defensa)** son trampas a evitar al hablar.

---

## TL;DR — estado actual (todo cerrado)

- **Propensión y clustering:** cerrados y verificados. Modelo final **LightGBM+Optuna calibrado**.
- **Contrato 2 (scoring):** materializado y **correcto** en Delta (19.71 M filas; `avg prob` 0.060 ≈ tasa base).
- **Informe + tablero:** consolidados; `main` conciliado y pusheado.
- **Lo único vivo:** la **PPT** y el **ensayo de defensa**.

---

# Parte 0 — El mapa mental: qué corre dónde y por qué

Hay **cinco notebooks** repartidos en dos mundos:

| Notebook | Dónde | Motor | Qué hace |
|---|---|---|---|
| `01_extraccion_gold_a_parquet` | Databricks | Spark | Vuelca la Gold Delta a un parquet (`gold_snapshot`) |
| `02_modelado_propension` | **Local** | pandas/sklearn | Entrena y evalúa la propensión |
| `03_drift_split_diagnostico` | Databricks | Spark | Diagnóstico de deriva para justificar el split |
| `04_clustering` | **Local** | pandas/sklearn | Clustering de visitantes |
| `06_scoring_mlflow_databricks` | Databricks | pandas+Spark | Scoring batch + MLflow + Contrato 2 |

**La estrategia es "entrena local, sirve en la nube":** Databricks materializa la Gold (~4.7 GB) como parquet, ese parquet se baja al repo local, el **modelado de exploración (02, 04) corre local** en pandas/sklearn (Optuna, calibración, permutación — el ecosistema que enseña el curso), y lo **distribuido/producción vuelve a Databricks** (drift sobre 109 M eventos crudos en `03`, y scoring + MLflow + Contrato 2 en `06`).

**No es "prototipo local y reimplemento todo en Spark".** Es **división por naturaleza de la tarea**: Spark donde el dato es grande o es streaming; pandas/sklearn donde el cómputo es modelado tabular sobre la sesión agregada (~23 M filas que caben en memoria, y donde SparkML es más pobre que sklearn).

**Respuesta defendible ante el profe de Grandes Datos** (pregunta fija del jurado): *"Usamos Spark donde el volumen lo justifica —ingesta streaming de 109 M eventos, Medallion en Delta, diagnóstico de deriva, materialización del scoring a Delta— y bajamos a pandas/sklearn para el modelado tabular porque la unidad de análisis es la sesión agregada (~23 M filas, ~4.7 GB), que cabe en memoria y donde el ecosistema de ML es superior. Distribuir un sklearn de 23 M filas no aportaba nada y perdía herramientas del estándar."* Esto convierte "¿por qué no todo en Spark?" en una **decisión por tamaño + patrón**, no por defecto.

---

# Parte 1 — Dónde se cruzan los dos modelos (el insight central)

El cruce clasificador × clustering ocurre en **tres niveles**:

1. **Cruce conceptual (en `04` §6):** se calcula la **tasa de conversión real por cluster** y se grafica contra el valor. Identifica **C3 «electrónica gama media»** como el segmento objetivo: 29.3 % del tráfico, conversión 8.4 % (la mayor), pero 91.6 % no compra → bolsa recuperable. Responde *qué tipo de visitante concentra la oportunidad*.
2. **Cruce individual (en `06`, el Contrato 2):** aquí se unen de verdad. El clustering dice *en qué segmento* está la sesión; el modelo de propensión dice *con qué probabilidad* compra esa sesión. **El segmento dice dónde enfocar; el modelo dice a quién exactamente.** Dentro de C3 hay sesiones con 5 % y con 40 % de probabilidad.
3. **Cruce operativo:** el Contrato 2 (`user_session`, `prob_calibrada`, `segmento`) alimenta el targeting del A/B (Yeison) y la matriz de oportunidad del tablero (Kelly).

**Estado: el cruce individual ya está MATERIALIZADO** (antes solo narrado). El Contrato 2 corrió correcto en Databricks (19.71 M filas, `avg prob` 0.060).

**¿El clustering usa la propensión predicha como feature? NO — y es correcto.** Meter la propensión predicha sería circular (agruparías por la salida de otro modelo) y contaminaría la interpretación de "tipos de visitante". El clustering describe *quién es* el visitante por su conducta; la propensión es una capa separada que se cruza después, **por sesión, en el Contrato 2** (no anidada).

---

# Parte 2 — Modelo de propensión (dudas, una por una)

## 2.1 El "Caveat Abierto (Riesgo Técnico)"

Es la honestidad intelectual frente a la **anomalía de noviembre**: en 14–17 nov hay un pico de `cart` con `purchase` deprimido. Dos explicaciones posibles: (a) comportamiento real cerca de Black Friday, o (b) artefacto del pipeline de medición. **No hay que resolver cuál es** para que el modelo sea válido, porque se **cuarentenó la ventana 14–17** (fuera de train y test). Es un **riesgo declarado y acotado**, no un bug. Para la defensa es bueno tenerlo escrito: viste la anomalía, la aislaste, y eres transparente sobre lo no confirmable con datos observacionales.

## 2.2 Por qué estos baselines y modelos (no otros)

La secuencia **Dummy → Logística → familias de árboles (RF/XGB/LGBM) → LightGBM+Optuna** existe por el estándar del profe:

- **Dummy** ("nadie compra"): expone la **trampa del accuracy** — 94.4 % accuracy y **PR-AUC 0.056** detectando cero compras. Fija el piso: superar 0.056 de PR-AUC, no 0.94 de accuracy.
- **Logística:** baseline probabilístico, lineal, interpretable (PR-AUC 0.097, ~1.7× el azar). Responde *cuánta señal captura un modelo lineal simple*; si un GBM costoso no le ganara claro, no estaría justificada su complejidad.
- **Por qué árboles/ensembles:** el problema es **tabular**, con **interacciones no lineales** (precio × categoría × ritmo), **escalas mixtas** (árboles invariantes a escala), **desbalance fuerte** (manejable con `scale_pos_weight`) e **importancia interpretable** (exigida por la rúbrica). Los GBM son el estado del arte en tabular.
- **Por qué estas tres familias:** cubren las tres estrategias de ensemble del curso — **RF = bagging**, **XGBoost = boosting** (referencia madura), **LightGBM = boosting histograma/leaf-wise**. El curso pide probar ≥2 familias bajo **misma CV, métrica y muestra** (comparación justa).
- **Por qué no CatBoost (ojo defensa):** ya no hay categóricas crudas de alta cardinalidad (se codificaron a numéricas). Mencionarlo como "considerado y descartado por ausencia de categóricas crudas" suma.
- **Por qué LightGBM (matiz fino):** las tres familias quedan **empatadas dentro del ruido** (XGB 0.1185 vs LGBM 0.1184, diferencia < variabilidad entre folds). Se elige LightGBM por **menor brecha de overfitting (0.018 vs 0.027)** → generaliza mejor, **no** porque tenga mejor score. Cierre: *"lo que mueve la aguja es el tuning, no la familia."*

## 2.3 Feature importance: built-in vs permutación

Los árboles dan importancia intrínseca, pero el notebook usa **ambas** y la lectura válida es la **permutación**:

- **Built-in `split`** (nº de cortes): sesga hacia features de **alta cardinalidad**.
- **Built-in `gain`** (reducción de impureza): más fiable, pero medida interna sobre el train.
- **Permutación:** baraja una feature en datos **no vistos** (test) y mide cuánto **cae el PR-AUC** → mide importancia respecto a la métrica que importa, agnóstica al modelo.

**El caso estrella `electronics_view_share`:** split 0.042 (casi al fondo) vs permutación **0.203 (#2)**. Es de **baja cardinalidad** (valores 0/0.5/1): se usa en pocos cortes pero cada uno es muy informativo. *"Si nos quedamos con el split count, habríamos descartado erróneamente la feature de electrónica."*

**Qué dicen (negocio):** las top por permutación son `max_price_viewed`, `electronics_view_share`, `avg_price_viewed` → **lo que predice la compra es cuán caro/electrónico es lo que miras, no cuánto navegas**. Triangula con EDA y clustering: tres análisis independientes, un mismo insight. **SHAP** solo como capa extra (cero menciones en el curso); la permutación es la base.

## 2.4 Qué aporta "GBM (LightGBM) + Optuna"

Convierte "un LightGBM razonable" en "el LightGBM **justificado**". Optuna hace **HPO como diseño experimental** (TPE bayesiano) sobre 9 hiperparámetros; la **función objetivo** devuelve la media de PR-AUC en **StratifiedKFold 5-fold**; 20 trials. El tuning sube de 0.118 (sin tunear) a **CV 0.1235**.

**R4 — variabilidad entre folds (verificado):** el modelo elegido da **PR-AUC 0.1235 ± 0.0014** (5 folds, rango 0.122–0.126). La baja varianza confirma que **no es un máximo aislado** sino un resultado estable — exactamente lo que pide el profe (*"menor variabilidad puede ser mejor evidencia que un máximo aislado"*).

## 2.5 Cómo interpretar la evaluación en test

| Subconjunto | PR-AUC | Brier | tasa |
|---|---|---|---|
| Test completo | **0.1172** | 0.0515 | 0.0560 |
| Test sin Black Friday | **0.1115** | 0.0496 | 0.0536 |
| Test solo BF (29–30) | 0.1287 | 0.0559 | 0.0613 |

Lectura por capas:

1. **PR-AUC 0.1172 vs base 0.056 → ~2.1× el azar.** Nunca lo compares con 1.0; compáralo con la tasa base y los baselines. En evento raro (~5.6 %), el doble de la base es sólido.
2. **Test (0.1172) ≈ CV (0.1235) → no hay fuga ni sobreajuste.** Es el chequeo más importante: el desempeño fuera-de-tiempo casi iguala la validación interna.
3. **Con/sin Black Friday:** el **0.1115 ex-BF es el número "régimen normal honesto"** (úsalo de titular); el global 0.1172 incluye el empujón de BF. Reportar ambos es maduro.
4. **Brier 0.0515 (calibración):** mide si las probabilidades son honestas (si dice 0.30, ~30 % compran). Bajo el baseline de la tasa base → calibración decente. Cumple *discriminar ≠ calibrar*.

**(Ojo defensa) Dos tasas base:** la **5.6 %** es la prevalencia del **test** (nov 24–30) — de ahí el Dummy 0.056 y el lift. La **0.0597** es la base limpia **global**. Usa 5.6 % para el lift, 0.0597 para describir los datos; no las mezcles.

## 2.6 El umbral operativo: implicación de negocio

El modelo entrega una probabilidad; el **umbral la vuelve decisión**. No es 0.5 (eso sería un error bajo desbalance). El **F1-óptimo es 0.092**:

> Marca el **15 % de las sesiones**; dentro de ese 15 %, **precisión 12.1 %** (vs 5.6 % general) → **lift ~2.2×**; captura el **32 % de compradores** (recall) con solo el 15 % del tráfico.

La tabla de alternativas es la herramienta de decisión: **intervención barata** (banner/email) → umbral bajo ~0.09–0.10 (más alcance); **intervención cara** (cupón/agente) → umbral alto ≥0.20 (a 0.20 precisión ~30 %, a 0.30 ~50 %, pero marca <1 %). Conecta con el A/B (a quién se le muestra el incentivo dentro de C3/C4) y con la matriz deslizable del tablero. Frase: *"el modelo estima; el umbral decide."*

**(Ojo defensa)** el umbral se calculó sobre el test → estrictamente roza *"si el test fija el threshold, deja de ser test"*. Defensa: es una **palanca de negocio** ajustable en producción, no un hiperparámetro del modelo, y se reporta como tal.

## 2.7 ¿Genera valor o "modelamos por modelar"?

Genera valor real: (1) un **ranking accionable** (el 15 % con 2.2× propensión → concentra el presupuesto); (2) el **diagnóstico del porqué** (precio + electrónica mandan); (3) el **insumo del experimento** (a quién targetear en el A/B). *La predicción es el instrumento; la decisión es el producto.*

---

# Parte 3 — Qué hace `03_drift_split_diagnostico`

Es un **notebook de evidencia, no de modelado** (no entrena nada). Corre en **Databricks/Spark** porque mira los crudos (109 M eventos). Su trabajo: **justificar empíricamente** las decisiones de partición que `02` da por hechas. Mide deriva por tres vías:

1. **Drift de etiqueta (conversión por sesión):** ~6.8 % (oct) → ~5.0 % (inicio nov) → ~6.1 % (fin nov). Drift moderado del *target*, no catastrófico. La vista diaria **expone el desplome corrupto** 15–17 nov.
2. **Drift de feature (PSI del precio):** umbrales 0.10/0.25; todos **≈ 0** (máx 0.0058) → el precio **no deriva**.
3. **Composición (event_type / categoría):** confirma el `cart` anómalo de noviembre.

**Hallazgo clave:** *el drift está en el comportamiento del target (conversión), no en la escala de las features (PSI≈0)*. Eso justifica **incluir noviembre en el train** (para calibrar la tasa al régimen de evaluación) y que **no haga falta reescalar**. Sustenta la **Opción C** (train oct+nov≤23 / test 24–30, *out-of-time*) y la cuarentena. La cuarentena ya quedó alineada a **14–17** en el markdown.

---

# Parte 4 — Clustering (dudas, una por una)

## 4.1 ¿"Cruzar con la conversión" = comparar o cruzar con el modelo?

Hoy en `04` §6 es lo primero: la **tasa de conversión real por cluster** (no la predicción). El notebook la llama "propensión del segmento" pero es un promedio grueso. El cruce con el **modelo** (la predicción individual) ocurre **por sesión, en el Contrato 2 (06)** — y ya está materializado.

- **Features del clustering (12):** comportamiento de sesión (vistas, productos, marcas, categorías, duración, precios, electronics_share, revisit, ritmo). Escaladas con StandardScaler. **PCA solo para visualizar** (2 comp.), no como entrada de KMeans.
- **¿Usa la propensión predicha? NO** (correcto, evita circularidad).

## 4.2 ¿Solo para el A/B? No

Tres usos: (1) **definir el segmento objetivo del A/B** (C3, principal); (2) **insight para directivos** (§10 arquetipos: ~54 % de compradores es electrónica, élite premium 14 %, 5 % "investigador"); (3) **alimentar el tablero** + calidad de datos (C2 / DBSCAN detectan sesiones rotas).

## 4.3 El gráfico de §6 (visitante × propensión)

Scatter de **5 burbujas** (una por cluster): **X = conversión real**, **Y = "valor" proxy** (`electronics_view_share × avg_price_viewed`, **no monto real**), **tamaño = % de tráfico**. **C3 domina las dos dimensiones** (mayor conversión + alto valor) con volumen accionable. **Sirve para PPT/tablero** (legible), pero **siempre aclara que el eje Y es un proxy compuesto, no dólares**. Quita C2 (anómalo) del gráfico de presentación.

## 4.4 Las visualizaciones de §7: cuáles sirven

| Gráfico | ¿Limpio? | ¿Dónde? |
|---|---|---|
| §6 burbujas (conversión × valor) | Sí, muy legible | **PPT y tablero** (el mejor para negocio) |
| §7.2 ejes de negocio (electronics × precio) | Aceptable, con bandas verticales | **PPT/tablero con la advertencia** de las bandas |
| §7.1 scatter PCA (57 % var) | Ruidoso/solapado, ejes abstractos | Solo backup técnico |
| §8 DBSCAN (KMeans vs DBSCAN) | Técnico | Solo si preguntan por validación |

**La verdad a defender:** k=5 tiene **silhouette 0.30** (de las más bajas; k=2 daría 0.52 pero solo separa "electrónica sí/no", demasiado grueso). Se eligió k=5 **por accionabilidad** (5 grupos nombrables con acción distinta), no por la métrica. Los clusters **se rozan** porque la estructura es un **gradiente, no islas** (lo confirma DBSCAN: 1 masa + 3.7 % ruido). Frase-ancla: *"el mejor k no es el que maximiza una métrica, es el que sostiene una partición defendible."*

**Cómo comunicarlo bien:** no muestres el scatter PCA como "mira qué bien separan" (se ve solapado). **Lleva al frente la tabla de perfilado** (tamaño/conversión/precio/electrónica por cluster) y un **gráfico de barras de conversión por cluster** — mucho más convincente. Usa el de burbujas (§6) como cierre.

**(Ojo defensa) dos correcciones de exactitud:**
- El cluster con **duración anómala ~15 días es C2** (sesiones rotas), **no C3**. C3 (target) tiene duración normal.
- La numeración C0–C4 depende de la semilla → **nombra por perfil**, no por índice. El `06` reprodujo las proporciones del `04` (49.8/29.5/13.6/7.0/0.05 %), lo que es **evidencia de estabilidad**.

## 4.5 Los 5 segmentos (tabla de defensa)

| Segmento | % | Conv. | Perfil |
|---|---|---|---|
| **C3 Electrónica gama media** | 29.3 % | **8.4 %** | ~99 % electrónica, ~$257, decide rápido. **OBJETIVO A/B** |
| C4 Electrónica premium | 13.8 % | 5.8 % | Ticket ~$1,024, mayor valor |
| C0 General bajo valor | 49.8 % | 4.9 % | No electrónica, ~$154, mucho volumen |
| C1 Explorador que no cierra | 7.0 % | 3.2 % | Navega mucho (18.5 vistas), compra poco |
| C2 Anómalo | 0.04 % | — | 106 sesiones rotas (~14.6 días); calidad de datos |

---

# Parte 5 — `06_scoring_mlflow_databricks` (de "no corría" a CERRADO)

Esta parte cambió por completo: **pasó de roto a resuelto y verificado.**

## 5.1 Qué hace

Cierra el frente: (1) **carga** el modelo calibrado entrenado en local (`joblib` subido al Volume — no re-entrena); (2) lo **registra en MLflow** (params + PR-AUC/Brier + signature); (3) **scorea las 19.71 M sesiones** distribuido; (4) **asigna el cluster**; (5) materializa el **Contrato 2** (`user_session`, `prob_calibrada`, `segmento`, `segmento_nombre`) en Delta.

## 5.2 El patrón correcto (Serverless)

El truco que lo hace viable en Free Edition: **Spark hace lo masivo, pandas solo lo que cabe.** Tres restricciones del entorno, cada una con su decisión (esto va en el informe §4.1 y es oro para el jurado de Grandes Datos):

1. **Memoria del driver:** cargar 19.71 M filas en pandas mata el driver (OOM, `exit 137`). Solución: **Spark DataFrame perezoso + `pandas_udf`** que procesan particiones; solo baja al driver la muestra de 300 k para el KMeans.
2. **Sin `sparkContext` en Serverless:** no hay broadcast. Solución: **carga del modelo desde el Volume dentro del UDF** + closures para los objetos ligeros (scaler, KMeans).
3. **Incompatibilidad de NumPy:** el modelo se serializó con NumPy 2.x (`numpy._core`); Databricks tiene 1.x. Solución: pin de versiones + alias de módulo.

## 5.3 El bug que cazamos (y por qué importa)

La primera corrida exitosa dio una señal mala: `avg(prob_calibrada) = 0.0275`, la mitad de la tasa base. Como el **PR-AUC también cayó** (0.117 → 0.081) y el PR-AUC es de ranking (la calibración monótona no lo cambia), el problema **no era calibración**: era **orden de features**. `MODEL_FEATURES` en el `06` estaba en distinto orden que el de entrenamiento (`02`) — 6 features permutadas — y el scoring posicional las revolvía.

**Fix:** alinear `MODEL_FEATURES` al orden exacto del `02` + pinear `scikit-learn==1.6.1`. **Re-corrida verificada: `avg prob` = 0.0601 ≈ tasa base**, PR-AUC reproduce 0.117. **Contrato 2 correcto.** (Lección de defensa: un scoring que "corre" no es un scoring "correcto"; lo validamos contra las métricas offline.)

## 5.4 Estado: CERRADO

Contrato 2 = 19,711,743 filas; distribución de segmentos consistente con el clustering; probabilidades reales. Ya alimenta el tablero (Kelly) y el targeting del A/B (Yeison).

---

# Síntesis — estado y qué falta

| Frente | Estado |
|---|---|
| Propensión (02) | CERRADO — LightGBM+Optuna, CV 0.1235 ± 0.0014, test 0.1172, Brier 0.0515 |
| Drift/split (03) | CERRADO — evidencia de Opción C; cuarentena 14–17 alineada |
| Clustering (04) | CERRADO — k=5, C3 objetivo; estabilidad confirmada por el 06 |
| Scoring (06) | CERRADO — Contrato 2 correcto (avg 0.060), bug de orden de features resuelto |
| Informe | CERRADO — consolidado en `entrega_final/`, referencias APA, R4, fidelidad auditada |
| `main` | Conciliado y pusheado |
| **Pendiente** | **PPT + ensayo de defensa** |

---

# Apéndice — matices verificados para el Q&A (no tropezar)

- **$250 M vs $224 M:** el $250 M de carritos en juego es el **total global incl. `Unknown`**; las barras por-categoría del tablero **excluyen `Unknown`** y suman ~$224 M. No las sumes en vivo esperando $250 M.
- **2.2 min vs C3 ~12 min:** 2.2 min es la mediana **global** de compradores; **C3** perfila ~12 min. No cites "2.2 min" con C3 en pantalla.
- **Base rate:** 5.6 % = test (para lift/Dummy); 0.0597 = base limpia global (para datos).
- **A/B (cifras):** MDE +0.5 pp (8.4 %→8.9 %) → **~49,600 por brazo, ~99,200 total**; alcanzable en pocos días.
- **Jurado:** Marco Terán (ML). Edison Valencia ("BA/HPC") y Mauricio Árias ("Diseño") para narrativa/diseño; **confirmar quién juzga Grandes Datos** (el tag HPC sugiere Edison Valencia).

# Apéndice — frases-ancla del profe (para soltar en la defensa)

- *"El código que corre no basta: el experimento debe ser válido, comparable y defendible."*
- *"Accuracy puede reportar 99.8 % y no detectar ningún evento."* (→ por eso PR-AUC)
- *"El modelo estima; el umbral decide."* / *"Una probabilidad no es una acción."*
- *"Si el test se usa para escoger el modelo o el threshold, deja de ser test."*
- *"El futuro no entrena el modelo que evalúa el pasado."* (→ split temporal + anti-fuga)
- *"Nunca digas «mejor modelo» sin decir según qué métrica, validación y comparación."*
- *"El mejor k no es el que maximiza una métrica, es el que sostiene una partición defendible."* / *"Cluster ≠ segmento."*
- *"Discriminar bien no es calibrar bien."* (→ Brier + curva de calibración)
- *"Una visualización abre una hipótesis; no cierra una decisión."*
