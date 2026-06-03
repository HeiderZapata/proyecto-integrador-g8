# Pack de contexto — Aprendizaje Automático

> Destilación propia (no copia de las diapositivas) del material del curso **Aprendizaje Automático** (SI7009, Universidad EAFIT, 2026), profesor **Marco Terán**. El curso fueron 5 sesiones intensivas con un caso guía de **fraude transaccional** (evento raro), más sesiones de boosting/HPO, series de tiempo/walk-forward y aprendizaje no supervisado/confiabilidad. Este pack prioriza lo que el profe enfatizó y lo que se evalúa, y lo ata a nuestro alcance (propensión + clustering + diagnóstico + diseño de A/B test). Las **notas de alcance** señalan dónde el material va más allá —o difiere— de lo que hacemos.

---

## 1. Técnicas y conceptos enseñados

El hilo conductor del curso no es "qué algoritmo", sino **"cómo defender una decisión de modelado"**. Cada sesión añade una capa de rigor metodológico sobre el mismo caso de evento raro.

**1.1 Workflow y evaluación moderna (Sesión 1)**
- **La cadena defendible:** problema → baseline → métrica → validación → threshold → decisión. Un score alto no prueba nada por sí solo.
- **Trampa del accuracy:** con clases muy desbalanceadas, un clasificador "todo negativo" alcanza 99.8 % de accuracy y 0 % de recall. Accuracy oculta el fracaso.
- **Train / Validation / Test / Inferencia** como roles distintos. Regla de oro: *"si el test se usa para escoger el modelo o el threshold, deja de ser test."*
- **Baseline serio (no es un modelo malo):** referencia explícita para juzgar mejora. Niveles: trivial (`DummyClassifier`, revela la trampa), regla de negocio, probabilístico (regresión logística / árbol superficial). "El baseline trivial enseña; el baseline serio compara."
- **Matriz de confusión como mapa del error.** Precision (calidad de las alertas, vigila FP), Recall (cobertura de positivos, vigila FN), F1 (compromiso, no reemplaza el costo del error).
- **La métrica depende del error dominante.** En eventos raros y revisión limitada: **PR-AUC** y precision/recall en el punto de operación, no accuracy ni solo ROC-AUC.
- **ROC vs PR:** ROC mira TPR vs FPR (separación global, puede verse optimista con positivos raros); **PR pone la clase positiva rara en primer plano**.
- **Threshold = política operativa.** "El modelo estima; el threshold decide." 0.5 no es universal; se defiende con costo del error y capacidad operativa.
- **Árboles y ensembles (fundamento):** CART (particiones recursivas), impureza Gini vs Entropy, Information Gain, regularización (`max_depth`, `min_samples_leaf`, `min_samples_split`), sesgo-varianza, overfitting. Ensembles: voting (hard/soft), bagging (reduce varianza), boosting (corrige errores secuencialmente). Requisito clave del ensemble: **diversidad** (errores no correlacionados).
- **Estrategias mínimas de desbalance:** `class_weight` (pondera la pérdida), under/oversampling, SMOTE/ADASYN. **Solo en train, nunca en test.** Aplicar SMOTE antes del split = *leakage*.

**1.2 Boosting moderno e HPO (Sesión 2)**
- **Boosting = corrección secuencial controlada** (no "muchos árboles"): cada árbol corrige el residuo/gradiente anterior; exige control de `learning_rate`, profundidad, regularización y subsampling.
- **Familias:** XGBoost (referencia madura, boosted baseline defendible), LightGBM (histogramas, crecimiento leaf-wise, rápido), CatBoost (categóricas como señal con *ordered boosting*, evita fuga de target). Se eligen por ajuste al problema, **no por moda**; probar al menos dos.
- **Hiperparámetros como controles científicos:** cada uno es una hipótesis de cómo debe aprender el modelo, no "un botón de rendimiento". `learning_rate` y `n_estimators` se leen juntos.
- **Early stopping ≠ validación:** controla cuándo para una configuración; no valida la comparación entre pipelines.
- **Validación cruzada estratificada** dentro del tuning. En desbalance, estratificar evita folds casi sin positivos y "ganadores falsos".
- **HPO como diseño experimental (Optuna):** optimización bayesiana (modelo sustituto + función de adquisición, p. ej. *Expected Improvement*), *define-by-run*. "Optuna no decide qué es bueno: ejecuta la definición de calidad que programaste." Una mala función objetivo produce evidencia falsa más rápido.
- **Congelar el experimento antes de buscar:** métrica, CV, espacio de búsqueda, criterio de parada y regla de selección final.
- **Robustez antes de declarar ganador:** leer promedio **y** variabilidad entre folds; menor variabilidad puede ser mejor evidencia que un máximo aislado.

**1.3 Series de tiempo, forecasting tabular y walk-forward (Sesión 3)**
- **El tiempo rompe la intercambiabilidad de las filas:** el pasado puede explicar el futuro, pero el futuro no puede participar en la evidencia que evalúa el pasado.
- **Disponibilidad operacional:** una feature es válida solo si su versión existía **antes** del momento de predicción `t` (conjunto de información `Iₜ`). "La validez de una feature depende de su disponibilidad operacional, no de su presencia en el archivo."
- **De serie a tabla supervisada:** lags, rolling windows (alineadas solo al pasado), codificación cíclica del calendario (seno/coseno). Perder filas al desplazar es consecuencia de respetar el tiempo, no un error.
- **Leakage temporal:** rolling mal alineado, escalamiento/imputación global (antes del split), variable externa publicada tarde, folds que mezclan futuro y pasado. Se audita el **pipeline completo**, no solo el estimador.
- **Random split puede mentir** en datos con dependencia temporal; **walk-forward validation** (entrenar en pasado, validar en bloque futuro, avanzar el origen) responde "¿cómo habría funcionado si se hubiera usado en ese momento?". Variantes: holdout temporal, ventana expansiva, ventana deslizante, *embargo* (brecha entre train y val).
- **Baselines temporales obligatorios:** naïve, seasonal naïve, moving average. Métricas: MAE, RMSE y **MASE** (relativo al naïve: <1 mejora al baseline). Evaluación **multi-horizonte** (no esconder varios horizontes en un solo número).

> **Nota de alcance (importante):** la Sesión 3 es de **forecasting de un valor continuo** (regresión, tráfico horario) con walk-forward. **Nuestro proyecto NO es forecasting** ni serie de tiempo: es **clasificación de propensión** con un **split temporal** (Oct entrena, Nov prueba). Lo que transferimos es el *criterio metodológico* —respetar la dirección del tiempo y la disponibilidad de información— que justifica nuestro **split temporal** y el **corte anti-fuga**. No debemos presentar walk-forward, lags ni MASE como parte del modelo supervisado; sí podemos citar el principio "el futuro no entrena el pasado" para defender el split y el anti-leakage.

**1.4 Representación, clustering y confiabilidad (Sesión 4/5 — no supervisado)**
- **De tabla a geometría:** cada fila es un punto en ℝᵖ. Representar no es neutral: enfatiza unas direcciones y comprime otras.
- **Reducción de dimensión:** **PCA** (direcciones lineales de máxima varianza; inspecciona, no "prueba" clusters), **t-SNE / UMAP** (estructura local; *no* preservan distancias globales). "Si una conclusión depende de un solo embedding, todavía es una hipótesis visual."
- **K-Means:** centroides, distancia euclidiana, minimiza compactación intra-cluster (inercia `J`). **El escalado es parte de la definición de similitud** (una variable de mayor escala domina la distancia). Sensible a semilla, outliers y formas no convexas; Mini-Batch K-Means para escala.
- **Elegir k con criterio:** codo (inercia) + **silhouette** (cohesión vs separación), pero "el mejor k no es el que maximiza una métrica, es el que sostiene una partición defendible". Cruzar con tamaño, estabilidad e interpretación.
- **RFM** (Recency, Frequency, Monetary) como representación mínima e interpretable del comportamiento del cliente, base de la segmentación.
- **"Cluster ≠ segmento":** el algoritmo entrega una partición; el analista debe demostrar que se puede **nombrar, dimensionar, perfilar, validar (estabilidad) y accionar** de forma diferenciada. Si no cambia una acción, es agrupamiento exploratorio, no segmentación aplicada.
- **Outliers:** ruido, señal o cambio. Isolation Forest (rareza global) y LOF (densidad local). No eliminar por reflejo; priorizar para investigación.
- **Confiabilidad probabilística:** **discriminación ≠ calibración**. Curva de calibración (`calibration_curve`) + **Brier score**. "Si el modelo predice 0.80, ~80 % de casos similares deben ser positivos." **Threshold tuning ≠ calibration** (mover el corte no arregla una probabilidad mal calibrada).
- **Drift:** feature / score / label / performance drift. La respuesta no es pánico, es diagnóstico (comparar distribución de referencia vs periodo actual y decidir: ajustar threshold, recalibrar, reentrenar).

> **Nota de alcance:** la Sesión 4 de recomendación (MovieLens, top-N) y *uplift modeling / causal ML* aparecen en el **banco de temas de exposiciones** (línea Base/Expansión), **no** en el núcleo evaluado. Esto respalda nuestra decisión: el uplift causal no es parte del temario central y no es estimable con datos observacionales; va como límite o arquitectura de referencia, nunca como algo que hacemos.

---

## 2. Vocabulario y énfasis del profe (qué valora, qué penaliza)

**Frases-ancla del profe (literales del material):**
- *"El código que corre no basta: el experimento debe ser válido, comparable y defendible."*
- *"La evaluación mide criterio técnico, no recitación de algoritmos."*
- *"El modelo estima; el threshold decide."* / *"Una probabilidad todavía no es una acción."*
- *"Si el test se usa para escoger el modelo o el threshold, deja de ser test."*
- *"Nunca diga «mejor modelo» sin decir mejor según qué métrica, bajo qué validación y frente a qué comparación."*
- *"Un modelo no gana cuando suena más sofisticado; gana cuando sostiene el argumento más defendible."*
- *"La visualización abre una hipótesis; no cierra una decisión."* / *"Cluster no es segmento."*

**Qué VALORA (sube nota):**
- **Criterio defendible** sobre virtuosismo algorítmico. Cada decisión (target, baseline, métrica, validación, threshold, estrategia de desbalance) debe poder defenderse con una frase técnica.
- **Baseline honesto y explícito** antes de cualquier modelo complejo.
- **Métrica alineada al error dominante** (PR-AUC en eventos raros), reportada con **variabilidad** entre folds, no solo el promedio.
- **Validación que protege la generalización** y respeta la estructura del dato (estratificación; dirección temporal cuando hay tiempo).
- **No-leakage:** preprocessing/rebalanceo dentro del pipeline de cada fold; auditar el pipeline completo.
- **Interpretación que termina en decisión** (en clustering: nombrar y accionar; en clasificación: política de threshold y costo).
- **Conclusiones que "nombran el experimento":** "bajo CV estratificada, usando PR-AUC, el pipeline X superó a Y con variabilidad aceptable y complejidad justificada".

**Qué PENALIZA (errores que invalidan):**
- Reportar **accuracy** (o "mejor score") en evento raro sin métrica/validación que lo sostenga.
- **SMOTE/rebalanceo antes del split** o sobre todo el dataset → leakage.
- **Tunear con una métrica débil** (accuracy) o elegir por **el mejor trial aislado** (premia el azar, no la estabilidad).
- **Usar el test para iterar** o comparar **pipelines heterogéneos** (cambiar varias cosas a la vez → la mejora no es atribuible).
- **Early stopping presentado como validación.**
- En no supervisado: confundir **visualización bonita con prueba**, llamar "segmento" a cualquier color, o **elegir k solo por silhouette**.
- Eliminar outliers "por reflejo"; confundir **discriminación con calibración**.

---

## 3. Herramientas / librerías vistas en clase

Todo en **Python**, con notebooks reproducibles como artefacto de evidencia (las slides enseñan criterio, el notebook produce los números).

- **scikit-learn:** `DummyClassifier`, árboles (`DecisionTreeClassifier`), `cross_val_score`/`cross_validate`, CV estratificada, `Pipeline`, `StandardScaler`, `RandomizedSearchCV`, `HistGradientBoostingRegressor`/`Ridge` (sesión temporal), `KMeans`/`MiniBatchKMeans`, `PCA`, `TSNE`, `silhouette_score`, `IsolationForest`, `LocalOutlierFactor`, `calibration_curve`, `brier_score_loss`.
- **Boosting:** `xgboost` (`XGBClassifier`, `eval_metric="aucpr"`), `lightgbm` (`LGBMClassifier`, `num_leaves`, `min_child_samples`), `catboost` (`CatBoostClassifier`, `cat_features`, `eval_metric="PRAUC"`).
- **Desbalanceo:** `imbalanced-learn` (`imblearn.pipeline.Pipeline` + `SMOTE` **dentro** del fold).
- **HPO:** `optuna` (`create_study(direction="maximize")`, `trial.suggest_*`, función objetivo que devuelve el promedio de PR-AUC en CV).
- **Reducción/visualización no lineal:** `umap-learn` (UMAP), además de t-SNE y PCA.
- **Visualización:** `matplotlib` (principal en los notebooks; barras de importancia, curvas), uso puntual de `plotly`.
- **Métricas centrales:** PR-AUC / average precision, ROC-AUC, precision/recall en punto de operación, Brier score; MAE/RMSE/MASE en la sesión temporal.

> **Nota de alcance — importancia de variables:** la importancia se enseña con **`feature_importances_`** nativo del modelo final (y `get_feature_importance()` de CatBoost), graficada como "Top-20 variables por importancia". **SHAP NO aparece en el material del curso** (cero menciones reales). El doc 00 menciona "SHAP" para el diagnóstico: usarlo es legítimo pero es una **extensión** más allá de lo enseñado. Recomendación defendible: usar `feature_importances_`/importancia por permutación como base (lo que el profe reconoce) y, si añadimos SHAP, presentarlo como capa interpretativa adicional, no como "lo visto en clase". No mencionar MLflow como contenido del curso de AA (es de la línea de Grandes Datos / doc 02).

---

## 4. Expectativas de evaluación (qué espera ver en el PI)

El **Proyecto Integrador vale 35 %** de la materia. En AA se evalúan: **comunicación técnica, ejemplo en Python y reproducibilidad**, más el **aporte analítico defendible**.

**Rúbrica mínima en AA (lo que el profe exige sí o sí):**
1. **Problema y target claros** — qué define el evento positivo.
2. **Baseline explícito** — trivial y/o serio antes del modelo fuerte.
3. **Validación correcta** — esquema honesto que protege la generalización y respeta la estructura del dato.
4. **Modelos pertinentes** — elegidos por ajuste al problema, comparados de forma justa.
5. **Análisis de resultados** — lectura del error, métrica correcta, threshold y decisión.

**El estándar explícito para el componente supervisado del PI:**
> *"El componente supervisado del PI debe defender el experimento, no solo mostrar un resultado."*

Lo que esta materia exige aplicar (checklist del propio profe):
- No presentar modelos sin baseline; no reportar "mejor score" sin métrica y validación.
- No tunear sin explicar el criterio; no usar SMOTE/`class_weight` como recetas automáticas.
- No comparar pipelines si cambian demasiadas cosas a la vez.
- Conclusión que nombra métrica, validación, baseline y evidencia (promedio + variabilidad + costo).
- En la capa no supervisada/diagnóstica: representar con cautela, segmentar solo lo accionable, calibrar/auditar confiabilidad.

Entregables del curso (contexto): documento consolidado + presentación + **GitHub reproducible** + sustentación (~20 min). Entrega de productos **8-jun**, sustentación **9-jun**.

---

## 5. Mapeo "qué se enseñó → dónde aparece en nuestro proyecto"

| Lo que se enseñó | Dónde aparece en nuestro PI |
|---|---|
| **Definir target del evento positivo** | **Propensión de compra:** unidad = sesión; target = ¿la sesión contiene `purchase`? Clasificación binaria supervisada. Es exactamente la "formulación del problema y target" que pide la rúbrica. |
| **Trampa del accuracy + métrica según error dominante (PR-AUC)** | Conversión = **2.22 %** (positivo raro, igual que el fraude del caso guía). Por eso reportamos **PR-AUC / average precision** y precision/recall en el punto de operación, **no accuracy** (un "todo-negativo" daría ~97.8 % de accuracy y 0 % de recall). Este es el argumento central de por qué PR-AUC en vez de accuracy. |
| **Desbalanceo: `class_weight` / SMOTE solo en train** | Aplicamos `class_weight` o SMOTE **dentro del pipeline de cada fold** (`imblearn.Pipeline`), nunca sobre todo el dataset → evita el leakage que el profe penaliza. Lo tratamos como hipótesis a comparar, no como receta. |
| **Baseline serio + comparación congelada** | Baseline trivial (`DummyClassifier`) para exponer la trampa + baseline probabilístico (logística) antes de GBM (XGBoost/LightGBM/CatBoost). Misma métrica, misma CV, mismo preprocessing para todos. |
| **Boosting + HPO con Optuna y CV estratificada** | Modelo principal = boosting sobre features de sesión; tuning con Optuna optimizando PR-AUC promedio en **StratifiedKFold**; selección por promedio **y** variabilidad, no por máximo aislado. |
| **Disponibilidad de info en `t` / "el futuro no entrena el pasado"** | Justifica nuestro **split temporal** (Oct entrena → Nov prueba) y el **corte anti-fuga** (solo comportamiento **previo** al primer `cart`/`purchase` de la sesión). Es la traducción del criterio temporal del curso a un problema de clasificación. *(Nota: no hacemos walk-forward ni forecasting; usamos el principio, no la maquinaria.)* |
| **Calibración vs discriminación + Brier score** | Reportamos curva de calibración + Brier además de PR-AUC/ROC-AUC, porque la salida del modelo (probabilidad de compra) alimenta una decisión; "si dice 0.80, ~80 % deben comprar". Cumple la "calibración" del doc 00. |
| **Threshold como política operativa** | El corte que convierte `p̂` en acción se defiende con costo del error / capacidad, no con 0.5. Conecta con la matriz deslizable del tablero y con el A/B test (a quién se interviene). |
| **Importancia de variables** | Diagnóstico con **`feature_importances_`** del modelo final (Top-20). *(SHAP, si se usa, es extensión — ver nota §3.)* Alimenta la narrativa del funnel: qué variables explican la (no) conversión. |
| **Clustering con metodología (PCA → K-Means → k por codo/silhouette → perfilado)** | **Línea de clustering de visitantes:** escalado obligatorio, PCA opcional para inspección, K-Means, k por codo+silhouette **cruzado con interpretación**, perfilado y validación de estabilidad. Aplicamos "**cluster ≠ segmento**": cada grupo debe nombrarse y accionarse. |
| **RFM como representación de comportamiento** | Plantilla directa para construir el perfil de cada visitante a partir del clickstream (recencia/frecuencia/intensidad de interacción) antes de agrupar. |
| **"Cluster ≠ segmento" + el cruce con propensión** | El **cruce clasificador × clustering** es donde nace el insight: qué *tipo* de visitante concentra la alta propensión y la fuga → define el segmento objetivo del experimento. |
| **Diseño experimental / función objetivo / "no estimes lo que el dato no permite"** | **Diseño del A/B test:** como el dataset es observacional (sin tratamiento/control), el uplift causal no es estimable; entregamos el **diseño** del experimento (hipótesis, unidad de aleatorización, métrica primaria, tamaño de muestra/poder) que *mediría* el incentivo sobre el segmento de mayor intención. Coherente con que el profe trata uplift/causal como tema de extensión, no de núcleo. |
| **Drift / confiabilidad en operación** | Opcional para la narrativa de "qué seguiría": monitorear drift de scores entre periodos si el modelo se desplegara (scoring batch). No es pieza central. |

---

## 6. Conceptos/citas que conviene nombrar en la defensa

Frases y conceptos del profe que, dichos en la sustentación, demuestran que aplicamos *su* estándar:

1. **"El código que corre no basta: el experimento debe ser válido, comparable y defendible."** — Marco para abrir la capa de modelado.
2. **"Accuracy puede reportar 99.8 % y no detectar ningún evento."** — Justifica PR-AUC: con conversión de 2.22 %, accuracy es engañosa; medimos sobre la clase positiva rara.
3. **"El modelo estima; el threshold decide."** / **"Una probabilidad no es una acción."** — Conecta el clasificador con la decisión de negocio y con el A/B test.
4. **"Si el test se usa para escoger el modelo o el threshold, deja de ser test."** — Defiende nuestro split temporal Oct→Nov como evaluación honesta.
5. **"El futuro no puede entrenar el modelo que evalúa el pasado"** + disponibilidad de la feature en `t`. — Defiende el **corte anti-fuga** (solo señal previa al primer cart/purchase).
6. **"Rebalancear solo en train, nunca en test; SMOTE antes del split es leakage."** — Muestra que conocemos el error que más penaliza.
7. **"Nunca diga «mejor modelo» sin decir según qué métrica, bajo qué validación y frente a qué comparación."** — Plantilla para enunciar la conclusión del modelo (PR-AUC, StratifiedKFold, vs baseline, con variabilidad).
8. **"El mejor k no es el que maximiza una métrica, es el que sostiene una partición defendible."** + **"Cluster no es segmento."** — Defiende la metodología de clustering y el perfilado/nombrado de segmentos.
9. **"Discriminar bien no es calibrar bien."** — Justifica reportar calibración + Brier además de PR-AUC.
10. **"Una visualización abre una hipótesis; no cierra una decisión."** — Cautela al leer PCA/UMAP del clustering; el insight se valida, no se declara.
11. **Boosting = corrección secuencial controlada; HPO = diseño experimental ("Optuna ejecuta tu definición de calidad").** — Para explicar por qué tuneamos con criterio y no "probamos botones".
12. **Banco de temas:** uplift/causal ML está como **extensión de exposiciones**, no en el núcleo evaluado → respalda que el uplift va como *límite/arquitectura de referencia* y que lo que entregamos es el **diseño del A/B test**, no una estimación causal.

> **Cierre defendible (estilo del profe):** "No construimos un modelo para impresionar con un score; construimos un experimento defendible —target claro, baseline serio, PR-AUC bajo validación que respeta el tiempo, sin leakage, con probabilidades calibradas— cuyo resultado alimenta una decisión de negocio (segmento objetivo) y un experimento que la mediría (A/B test)."
