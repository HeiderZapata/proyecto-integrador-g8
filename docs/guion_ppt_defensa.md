# Guion de la PPT de defensa — PI Grupo 8

> **Qué es esto:** guion slide-por-slide para la defensa de 20 min (mar 9-jun, ante los tres profesores). Generado desde [`prompt_ppt_defensa.md`](prompt_ppt_defensa.md) con cifras **verificadas contra el repo** (notebooks ejecutados + CSV del tablero + informe). La PPT lleva el peso narrativo; el tablero se muestra en vivo solo al final.
>
> **Reparto:** **Yeison** (problema/cruce/A-B/cierre) · **Heider** (datos/arquitectura/despliegue) · **Kelly** (EDA-narrativa/tablero) · **Sara** (modelado: propensión + clustering). La historia es **una sola y continua**.
>
> **Paleta (coherente con el tablero):** gris oscuro `#2D2D2D` (estructura) · **verde `#3B6D11`** (oportunidad) · **rojo `#A32D2D`** (pérdida/fuga) · gris `#BBBBBB` (contexto). **Color solo en el dato clave; el resto en grises.** Ejes marcados y legibles (crítico por el proyector).
>
> **Tiempo:** 18 slides ≈ **19:55** incluido el demo de 2 min. Q&A va aparte, después. Si te pasas, recorta en slides 8 y 16.

---

## Arco narrativo

**Inicio** (contexto: la fuga) → **Conflicto** (¿dónde y quién?) → **Resolución** (modelo + segmento + experimento → acción). Abrimos y cerramos con la **Pregunta de Oro**.

---

### Slide 1 — Portada
- **Mensaje único:** Quiénes somos y qué defendemos, en una línea.
- **Contenido:** Título: *"Optimización de la conversión en e-commerce: un sistema de decisión sobre propensión de compra"*. Autores (Kelly, Sara, Heider, Yeison). Las 3 materias (Aprendizaje Automático · Grandes Datos · Visualización). EAFIT — Maestría en Ciencia de Datos. Fecha.
- **Visual:** Portada minimalista, fondo `#2D2D2D`, título en blanco, una sola línea de acento verde. Sin gráficos. (Misma estética que la página *Inicio* del tablero → coherencia.)
- **Notas del orador:** Una frase: *"Vamos a mostrar dónde se fuga la conversión de la tienda y qué hacer con eso."* No leer la portada.
- **Tiempo:** 0:15 — **Quién presenta:** Yeison

---

### Slide 2 — El problema (la fuga)
- **Mensaje único:** La tienda pierde 98 de cada 100 visitas y 4 de cada 10 carritos — esa es la pregunta del negocio.
- **Contenido:** "98 de cada 100 visitas no compran." "4 de cada 10 carritos se abandonan." **Pregunta de Oro (grande):** *¿Dónde se concentra la fuga de conversión y qué segmento de visitantes representa la mayor oportunidad de recuperarla?*
- **Visual:** Pictograma de 100 puntos (waffle 10×10): **2 verdes** (`#3B6D11`, compran) y 98 grises (`#BBBBBB`). Flecha/etiqueta roja (`#A32D2D`) integrada: *"98 % se va sin comprar."* Sin ejes (es pictograma); el mensaje va dentro.
- **Notas del orador:** *"De cada 100 personas que entran, 98 se van sin comprar; y de los que sí ponen algo en el carrito, 4 de cada 10 lo abandonan. No vinimos a construir un modelo para adivinar quién compra: construimos un **sistema de decisión** para entender dónde y por qué se fuga la conversión, qué visitantes la explican y dónde intervenir. La predicción es el instrumento; la decisión de negocio es el producto."* Plantar la **Pregunta de Oro** y prometer responderla al cierre.
- **Tiempo:** 1:15 — **Quién presenta:** Yeison

---

### Slide 3 — Los datos
- **Mensaje único:** Es un problema de Big Data real, y nuestra unidad de análisis es la sesión.
- **Contenido:** Dataset abierto **REES46** (clickstream multi-categoría, Oct–Nov 2019). **109.95 M eventos** (view/cart/purchase) · **~14.5 GB** crudos. → consolidados en **22.99 M sesiones** (Gold). Unidad = **sesión** (1 fila = 1 `user_session`). Base limpia de modelado: **19.71 M** sesiones, tasa de compra **5.97 %**.
- **Visual:** Barra de "embudo del dato" (no del negocio): 109.95 M eventos → 22.99 M sesiones. La barra de **sesiones en verde** (nuestra unidad), eventos en gris. Eje x = escala (M, log si hace falta). Anotación: *"agregamos a sesión: 1 fila = 1 visita con intención."*
- **Notas del orador:** *"Trabajamos sobre un dataset real de ~110 millones de eventos y 14.5 GB. Lo importante: no modelamos eventos sueltos, sino **sesiones** — porque la pregunta de negocio es por visita, no por clic."* (Encuadre Big Data para el jurado de Grandes Datos.)
- **Tiempo:** 1:00 — **Quién presenta:** Heider

---

### Slide 4 — Arquitectura de datos *(núcleo Grandes Datos)*
- **Mensaje único:** Diseñamos para producción (referencia) y lo implementamos acotado a la realidad de Databricks Free — con la frontera train/test visible en el flujo.
- **Contenido:** Medallion sobre **Delta Lake** (Bronze→Silver→Gold). Ingesta estilo **Kappa**: Auto Loader (`cloudFiles`) + `Trigger.AvailableNow` + checkpoint (replay de streaming, sin duplicar). **Particionamiento por tamaño:** Bronze/Silver por fecha + ZORDER; **Gold sin particionar** (~1.33 GB ≪ 1 TB → `OPTIMIZE` + `ZORDER(session_date, user_id)`). **Frontera train/test** = filtro sobre `session_date` con *data-skipping*. Gobernanza: **Unity Catalog** (permisos/accesos) en la referencia.
- **Visual:** **Dos diagramas lado a lado.** Izq. *Referencia* (Kafka/Kinesis → Flink → S3/Delta → warehouse → serving), en **gris** (lo ideal). Der. *Implementada* (Volume → Auto Loader/Kappa → Bronze/Silver/Gold Delta → Spark SQL → MLflow + scoring → Power BI), en **color institucional**. Marca roja/verde de la **línea train|test** sobre la Gold. Anotación: *"decidimos partición por **tamaño**, no por defecto."*
- **Notas del orador:** Plantilla "escenario ideal vs. desarrollado" (lo premian): *"En producción esto sería Kafka→Flink→S3; nosotros lo implementamos acotado en Databricks Free con un replay de streaming Kappa, que demuestra lo mismo sin broker externo."* Sobre partición: *"La Gold no se particiona a propósito: a 1.33 GB, particionar por fecha daría micro-archivos de ~22 MB — el anti-patrón. Por tamaño: ZORDER + data-skipping."* Mencionar que **el split se ve en el flujo** (frontera por `session_date`).
- **Tiempo:** 1:45 — **Quién presenta:** Heider

---

### Slide 5 — EDA · El embudo: dónde se fuga
- **Mensaje único:** La fuga grande no es solo "no llegan al carrito": 4 de cada 10 carritos mueren ahí.
- **Contenido:** Funnel (producto-en-sesión, base limpia): vistas → **carrito 3.86 %** → **compra 2.27 %**. **Abandono de carrito 41.19 %** (903 k carritos · **$250.4 M en juego**).
- **Visual:** **Gráfico de embudo** (3 etapas), ancho proporcional. La etapa **abandono en rojo** (`#A32D2D`), el resto gris. Eje: etapas en vertical, % a la derecha. Anotación integrada: *"41 % de los carritos se abandonan → la palanca A."*
- **Notas del orador:** Plantilla "Declaración + conector + razón": *"El abandono de carrito es 41 %, **porque** la intención existe pero algo frena el cierre. Ahí hay dinero recuperable sin traer más tráfico."* Conecta con la Pregunta de Oro: *"primera parte del 'dónde'."*
- **Tiempo:** 0:50 — **Quién presenta:** Kelly

---

### Slide 6 — EDA · La paradoja de la electrónica
- **Mensaje único:** Casi todo el dinero en juego está en una sola categoría: electrónica.
- **Contenido:** De los **$250 M en juego**, **~$187 M son electrónica**. Electrónica = mayor volumen y mayor pool de carritos abandonados. **Samsung + Apple = 68.6 %** de los carritos de electrónica.
- **Visual:** **Barras horizontales ordenadas por valor** = "revenue en juego" por categoría. **Electrónica en rojo** (`#A32D2D`), aplastando al resto (todas en gris). Eje x = $ en juego; eje y = categoría. Anotación: *"$187 M de $250 M = 1 categoría."*
- **Notas del orador:** *"La oportunidad no está repartida: se concentra en electrónica, y dentro de ella en dos marcas. Eso enfoca dónde mirar."* **Caveat para Q&A (no en slide):** el $250 M es el total **global incl. `Unknown`**; las barras por-categoría **excluyen `Unknown`** y suman ~$224 M — si alguien suma las barras, no dan $250 M (la diferencia es `Unknown`). No invitar a sumarlas en vivo.
- **Tiempo:** 1:00 — **Quién presenta:** Kelly

---

### Slide 7 — EDA · El núcleo recurrente
- **Mensaje único:** Pocos compradores explican casi todo el revenue — retenerlos es la segunda palanca.
- **Contenido:** **35.7 %** de los compradores (los recurrentes) generan **73.7 % del revenue**. Ticket recurrente **$1,460** vs one-time **$289** (5×). 2ª compra: mediana **2.9 días**, 84 % a la misma categoría.
- **Visual:** **Sankey / barras apiladas** compradores → revenue: dos bloques. El bloque **recurrente en verde** (`#3B6D11`, oportunidad de retención), one-time en gris. Anotación: *"35.7 % de clientes = 73.7 % del dinero."* (Mismo Sankey de la página *Análisis de cliente* del tablero → coherencia.)
- **Notas del orador:** Plantilla "Aumentó X debido a Y + pregunta": *"El revenue se concentra en los recurrentes **porque** vuelven y gastan 5× más. ¿Vale más perseguir desconocidos o blindar a estos? — esa es la **palanca B** (retención)."*
- **Tiempo:** 0:50 — **Quién presenta:** Kelly

---

### Slide 8 — Estrategia de modelado
- **Mensaje único:** La métrica y el split se eligen antes de competir modelos — y el split se hace antes de construir features.
- **Contenido:** Target = ¿la sesión contiene `purchase`? (tasa 5.97 %). Métrica = **PR-AUC + Brier**, *no accuracy* (evento raro). **Baseline Dummy = 0.056** (la "trampa del accuracy"). **Split temporal "Opción C"** (out-of-time): train = Oct + Nov ≤ 23 / **test = 24–30 Nov**. Cuarentena 14–17 nov. Anti-fuga: **features intra-sesión y pre-corte; el split se hace ANTES de las features**.
- **Visual:** Línea de tiempo Oct→Nov con la **banda train (gris)** y la **banda test (verde)**, la **ventana corrupta 14–17 en rojo** (cuarentena), y un sello *"features se construyen después del corte"*. Eje x = fechas. Anotación: *"un solo split, en la frontera."*
- **Notas del orador:** Pre-emptar las **tres** preguntas de Terán: *"**Temporal** porque replica producción (predecimos el futuro, no el pasado). **Uno solo**, en la frontera, para no contaminar; la CV estratificada va **dentro** del train. **Ese corte** lo respalda la evidencia de deriva: PSI de precio ≈ 0 y conversión estable Oct↔Nov (notebook 03)."* Cerrar: *"y lo importante — el split se hace **antes** de construir features; es justo el error que les marcaron a pregrado."*
- **Tiempo:** 1:30 — **Quién presenta:** Sara

---

### Slide 9 — Resultados: la cadena defendible
- **Mensaje único:** Cada salto de modelo está justificado, y el test casi iguala a la validación: no hay fuga ni sobreajuste.
- **Contenido:** Dummy **0.056** → Logística **0.097** → familias RF/XGB/LGBM empatadas **0.116–0.119** → **LightGBM + Optuna** (20 trials): CV **0.1235 ± 0.0014** (5 folds) → **test out-of-time 0.1172** (ex-Black-Friday **0.1115**). **Test ≈ CV ⇒ sin fuga.**
- **Visual:** **Barras ordenadas** de PR-AUC por modelo (eje y = PR-AUC, eje x = modelo). **LightGBM en verde** (elegido), resto gris; línea punteada del Dummy como piso. Anotación: *"test 0.1172 ≈ CV 0.1235 → generaliza."*
- **Notas del orador:** Frases-ancla de Terán: *"Nunca decimos «el mejor modelo» sin decir **según qué métrica, validación y comparación** — aquí es PR-AUC, CV de 5 folds, misma muestra. Y como el **test casi iguala a la CV**, sabemos que no hay fuga: «si el test eligiera el modelo, dejaría de ser test» — por eso el test lo tocamos una sola vez, al final."* Negocio: *"ordena a los compradores ~2× mejor que el azar."*
- **Tiempo:** 1:15 — **Quién presenta:** Sara

---

### Slide 10 — Calibración y umbral operativo
- **Mensaje único:** El modelo estima probabilidades calibradas; el negocio decide el umbral — y no es 0.5.
- **Contenido:** **Brier 0.0515** (probabilidades calibradas; baseline de la tasa base ≈ 0.056). Umbral **F1-óptimo ≈ 0.092** (no 0.5) → marca **15 % de sesiones**, precisión **12.1 %**, recall ~32 %, **lift ~2.2×** sobre la base **5.6 %**.
- **Visual:** Curva de calibración (predicho vs observado, diagonal de referencia en gris, modelo en verde) **o** barra de "lift": grupo marcado **12.1 %** (verde) vs base **5.6 %** (gris). Eje claro. Anotación: *"con el 15 % del esfuerzo se captura ~1/3 de las compras."*
- **Notas del orador:** Ancla de Terán: *"**El modelo estima; el umbral decide.** 0.5 sería un error bajo desbalanceo; fijamos el umbral por costo/F1 en 0.092."* Negocio: *"marcamos el 15 % de sesiones más propensas y ese grupo compra el doble que el promedio."* **Caveat (Q&A):** la base 5.6 % es la prevalencia del *test* (nov 24–30); la base limpia global es 5.97 % — no mezclarlas.
- **Tiempo:** 1:00 — **Quién presenta:** Sara *(apoya Yeison en la lectura de negocio)*

---

### Slide 11 — Qué predice la compra
- **Mensaje único:** Lo que predice la compra es **qué tan caro/electrónico es lo que miras**, no cuánto navegas.
- **Contenido:** Importancia por **permutación**: `max_price_viewed` (+0.038), `electronics_view_share` (+0.030), `avg_price_viewed` (+0.024). La importancia *built-in* (split-count) **subestima** `electronics_view_share` (0.042 vs **0.203** en permutación). Coherente con el EDA: el precio no frena y la compra se decide en **~2.2 min** (mediana global) — los compradores navegan *menos*.
- **Visual:** (a) Barras de **permutación vs built-in** lado a lado para `electronics_view_share` (muestra el sesgo del split-count, en gris vs verde). (b) **PDP / SHAP dependence**: probabilidad de compra **vs** `max_price_viewed` (eje x = precio, eje y = prob.), curva ascendente en verde. Anotación: *"a más caro/electrónico lo que miras, más probable la compra."*
- **Notas del orador:** *"No repetimos el ranking: lo **leemos**. La probabilidad **sube** con el precio y el peso de electrónica de lo que ves — no con el número de vistas. Por eso usamos **permutación** y no la importancia nativa, que subestima a `electronics_view_share` por su baja cardinalidad."* Cerrar con el insight contraintuitivo (2.2 min, navegan menos). *(No citar "2.2 min" cuando esté C3 en pantalla — C3 perfila ~12 min.)*
- **Tiempo:** 1:15 — **Quién presenta:** Sara

---

### Slide 12 — Segmentos (clustering)
- **Mensaje único:** De 5 tipos de visitante, uno destaca por convertir y pesar a la vez: C3, electrónica de gama media.
- **Contenido:** **k = 5** (elegido por **accionabilidad**, no por la métrica). **C3 «electrónica gama media»**: 29.3 % del tráfico, **conversión 8.4 % (la mayor)** → **objetivo del A/B**. C4 premium (13.8 %, $1,024, 5.8 %). C0 general bajo valor (49.8 %). C1 explorador que no cierra (7 %). C2 anómalo (calidad de datos). DBSCAN: 1 masa/gradiente + 3.7 % ruido (no islas).
- **Visual:** **Scatter** conversión (x) × valor de electrónica (y), **tamaño = % de tráfico**. **C3 en verde** (domina ambas dimensiones), resto gris; C2 marcado como ruido. Ejes rotulados. Anotación: *"C3: más conversión + buen volumen = objetivo."*
- **Notas del orador:** Ancla de Terán **cluster ≠ segmento**: *"No llamamos «segmento» a cualquier clúster: los **nombramos y accionamos**. Elegimos k=5 por **accionabilidad** — el silhouette favorecería k=2, pero solo separa «electrónica sí/no», demasiado grueso para decidir."* C3 = el de mayor conversión con volumen → candidato natural a intervenir.
- **Tiempo:** 1:10 — **Quién presenta:** Sara

---

### Slide 13 — El cruce (el insight central)
- **Mensaje único:** El segmento dice *dónde* intervenir; el modelo dice *a quién* — juntos hacen el targeting fino.
- **Contenido:** El bloque de electrónica **C3 + C4** (43 % del tráfico) combina **propensión + valor + volumen**. **Dentro** de ese bloque, el clasificador de propensión ordena sesión a sesión (Contrato 2: `user_session`, `prob_calibrada`, `segmento`). Clustering = el *dónde*; clasificador = el *a quién*.
- **Visual:** Diagrama de dos capas: (1) clustering recorta el bloque electrónica (verde); (2) dentro, un gradiente de probabilidad del modelo marca el 15 % top (verde intenso). Anotación: *"segmento × score = a quién mostrarle el incentivo."*
- **Notas del orador:** *"Aquí está el corazón del proyecto: el clustering nos dice **dónde** está la oportunidad (electrónica), y el modelo de propensión, **dentro** de ahí, **a quién** priorizar sesión a sesión. Ninguno solo basta; el **cruce** es el insight."*
- **Tiempo:** 1:05 — **Quién presenta:** Yeison

---

### Slide 14 — El experimento (A/B)
- **Mensaje único:** No prometemos uplift causal con datos observacionales: entregamos el **diseño** del experimento que lo mediría.
- **Contenido:** Hipótesis: un incentivo sobre **C3** sube la conversión por visitante. **Target = C3**; **aleatorización por visitante**; métrica primaria = conversión por visitante; **guardarraíl de margen**. Tamaño: **MDE +0.5 pp** (8.4 % → 8.9 %) ⇒ **~49,600 por brazo (~99,200 en total)** → alcanzable en **pocos días**.
- **Visual:** Esquema A/B: C3 → split aleatorio por visitante → brazo control vs tratamiento → métrica + guardarraíl. Números de muestra en una tarjeta. Anotación: *"observacional → entregamos el **diseño**, no el uplift."*
- **Notas del orador:** Cierra el alcance (responde al revisor): *"El dato es observacional: no hay variable de tratamiento, así que estimar uplift sería afirmar causalidad no verificable. Lo honesto es **diseñar el A/B** que sí lo mediría — sobre C3, aleatorizando por visitante, con guardarraíl de margen. Con un MDE de medio punto, ~49.600 visitantes por brazo, se corre en días."*
- **Tiempo:** 1:15 — **Quién presenta:** Yeison

---

### Slide 15 — Despliegue tecnológico
- **Mensaje único:** El modelo no quedó en un notebook: se persiste y alimenta el tablero y el A/B, y está verificado.
- **Contenido:** **Entrenamiento local + scoring distribuido** en Databricks (pandas UDFs, sin OOM). **MLflow** (params + PR-AUC/Brier + modelo **calibrado** + signature). **Contrato 2** en Delta: `user_session`, `prob_calibrada`, `segmento` — **19.71 M filas**. **Verificación:** al recargar el modelo reproduce **PR-AUC 0.1172 / Brier 0.0515 / prob. media ≈ tasa base (0.060)**.
- **Visual:** Mini-pipeline: modelo (MLflow) → scoring batch → **Contrato 2 (Delta)** → alimenta {tablero, A/B}. Sello verde *"scoring verificado: prob. media ≈ tasa base"*. Anotación: *"persistencia + reproducibilidad."*
- **Notas del orador:** *"Cerramos el ciclo de vida del dato: el modelo se loguea en MLflow y se aplica a las 19.7 M sesiones; el resultado (el **Contrato 2**) vive en Delta y alimenta el tablero y el experimento. Y lo validamos: al recargarlo, las probabilidades scoreadas reproducen el PR-AUC y el Brier, y su media coincide con la tasa base."* (Cubre *persistencia de modelos* de Grandes Datos.)
- **Tiempo:** 1:00 — **Quién presenta:** Heider

---

### Slide 16 — Conclusiones: respondemos la Pregunta de Oro
- **Mensaje único:** La fuga se concentra en el carrito de electrónica, y C3 es el segmento de mayor oportunidad — con dos palancas accionables.
- **Contenido:** **¿Dónde se fuga?** En el carrito (41 % abandono), concentrado en **electrónica** ($187 M en juego). **¿Qué segmento?** **C3** (electrónica gama media, mayor conversión + volumen). **Palancas:** (A) recuperar carritos de alta intención en electrónica; (B) retener al núcleo recurrente (73.7 % del revenue). **Aporte por materia:** ML (propensión + clustering calibrados), Grandes Datos (Medallion/Kappa + scoring + Contrato 2), Visualización (tablero ejecutivo desplegado).
- **Visual:** La **Pregunta de Oro** arriba, y debajo dos tarjetas de respuesta: *"Dónde: carrito de electrónica"* (rojo) y *"Quién: C3"* (verde). (Misma página *Cierre* del tablero → coherencia.)
- **Notas del orador:** Cerrar el arco: *"Abrimos preguntando dónde se fuga la conversión y qué segmento la explica. Respuesta: se fuga en el **carrito de electrónica**, y el segmento de mayor oportunidad es **C3**. El modelo prioriza a quién, el experimento mediría cuánto. La predicción fue el instrumento; la **decisión de negocio** es el producto."*
- **Tiempo:** 1:20 — **Quién presenta:** Yeison

---

### Slide 17 — Demo del tablero en vivo
- **Mensaje único:** Lo que contamos se sostiene con datos, en vivo, desde el enlace público.
- **Contenido:** Abrir el tablero (Power BI Service, **enlace público**, 7 páginas). Recorrido: *Problema* → *Detalle Electronics* → *Análisis de cliente* → *Cierre*. **Filtrar en vivo** (Top N marcas; categoría) para responder la Pregunta de Oro desde el dashboard.
- **Visual:** El tablero en vivo (no captura). Mostrar **filtros respondiendo** (criterio "funcionalidad y diseño"). Tener una captura de respaldo por si falla la red.
- **Notas del orador:** *"Esto está desplegado y es público. Filtro electrónica… aquí está el pool de carritos en juego; filtro por marca… Samsung y Apple. La historia de las slides es exactamente lo que ven aquí."* Definir público (negocio/dirección) y objetivo (convencer dónde intervenir).
- **Tiempo:** 2:00 — **Quién presenta:** Kelly

---

### Slide 18 — Cierre / Q&A
- **Mensaje único:** Gracias — y vamos a sus preguntas, respaldando cada respuesta con datos.
- **Contenido:** "Gracias." Pregunta de Oro repetida en pequeño. Datos de contacto/repo si aplica.
- **Visual:** Slide de cierre sobria (paleta del tablero). Sin gráficos.
- **Notas del orador:** Invitar preguntas; al responder, **volver al tablero o a la slide concreta** que respalda (no responder "de memoria").
- **Tiempo:** 0:10 + Q&A — **Quién presenta:** Equipo

---

## Mini-banco de Q&A (por profesor)

### Aprendizaje Automático — Marco Terán
1. **¿Por qué split temporal, uno solo, y ese corte?** Temporal porque replica producción (predecir el futuro); uno solo en la frontera para no contaminar (la CV estratificada va dentro del train); ese corte (test 24–30 nov) lo respalda la evidencia de deriva: PSI de precio ≈ 0 y conversión estable Oct↔Nov.
2. **¿Por qué PR-AUC y no accuracy o ROC-AUC?** Evento raro (~6 %): accuracy premia decir "no compra" siempre; PR-AUC se enfoca en la clase positiva escasa, y Brier mide la calibración de la probabilidad, que es lo que el negocio usa.
3. **¿Qué rangos de hiperparámetros y cuántos trials?** Optuna sobre `num_leaves`, `learning_rate`, `min_child_samples`, regularización, etc. (rangos en el notebook 02), **20 trials**; el modelo se eligió por la **media** de PR-AUC con baja varianza (0.1235 ± 0.0014), no por un máximo aislado.
4. **¿Cómo garantizan que no hay fuga?** Las features son intra-sesión y **pre-corte** (antes del primer cart/purchase); el split se hace **antes** de construir features; y el **test (0.1172) ≈ CV (0.1235)** lo confirma empíricamente.
5. **cluster ≠ segmento — ¿por qué k=5?** Por **accionabilidad**: el silhouette favorece k=2, pero solo separa "electrónica sí/no"; k=5 distingue gama media (C3) de premium (C4), que se accionan distinto. Los clústeres se nombran y accionan.

### Grandes Datos — (confirmar; perfil BA/HPC, prob. Edison Valencia)
1. **¿Por qué no particionan la Gold?** A ~1.33 GB (≪ 1 TB), particionar por fecha daría micro-archivos de ~22 MB (anti-patrón); usamos `OPTIMIZE` + `ZORDER(session_date, user_id)` y *data-skipping*. Bronze/Silver sí van por fecha.
2. **¿Dónde se ve la frontera entrenar/testear en el flujo?** Es un filtro sobre `session_date` con data-skipping; train = Oct + Nov ≤ 23, test = 24–30 nov, dibujado sobre la capa Gold del diagrama.
3. **¿Por qué Databricks Free y no Kafka/S3?** Esa es la arquitectura **de referencia**; la **implementada** usa el Volume (ya respaldado por object storage) + Auto Loader/Kappa, sin broker ni credenciales externas, sin quemar cuota ni exponer secretos.
4. **¿Cómo cuidan la cuota?** Iteración local + **una sola** corrida pesada (scoring batch); agregados Spark→pandas para el EDA/tablero; `Trigger.AvailableNow` (replay, no streaming continuo).
5. **¿Persistencia del modelo?** MLflow (params, PR-AUC/Brier, modelo calibrado, signature) + **Contrato 2** materializado en Delta (19.71 M filas), que alimenta tablero y A/B.

### Visualización — Mauricio Árias (Diseño) / Edison Valencia (BA)
1. **¿Cuál es el público y el objetivo del tablero?** Un único público (negocio/dirección) y un objetivo claro: **convencer** de dónde está la fuga y dónde intervenir. 7 páginas con navegación lateral.
2. **¿Por qué esos gráficos?** Embudo para el funnel, barras ordenadas por valor para el revenue-en-juego, Sankey para el flujo de segmentos: los recomendados según el tipo de dato.
3. **¿Por qué las barras por categoría no suman al titular de $250 M?** Las barras por-categoría excluyen `Unknown` (~32 % de eventos, sin taxonomía); el titular es el total **global incl. `Unknown`**. Está documentado en `reports/data/README.md`.
4. **¿Cómo varía la probabilidad según las características?** En la slide 8: la probabilidad de compra **sube** con `max_price_viewed` y `electronics_view_share` (PDP/SHAP dependence) — a más caro/electrónico lo que miras, más probable la compra.
5. **¿Coherencia y legibilidad?** Paleta gris/verde/rojo consistente PPT↔tablero, **color solo en el dato clave**, ejes marcados, un mensaje por slide; ensayamos proyectando.

---

## Checklist de ensayo (antes del mar 9)
- [ ] Cronometrar: el cuerpo debe caber en **~18 min** + 2 de demo. Si se pasa, recortar slides 8 y 16.
- [ ] **Ensayar el filtrado en vivo** del tablero desde el enlace público (y tener **captura de respaldo** por si falla la red).
- [ ] Proyectar de verdad: verificar ejes y contraste en pantalla grande.
- [ ] Cada quien domina **sus** slides + las 2 transiciones adyacentes (la historia es continua).
- [ ] Repasar el banco de Q&A; al responder, **apoyarse en el tablero o la slide**, no de memoria.
- [ ] No mezclar bases: 5.6 % (test) vs 0.0597 (global); no citar "2.2 min" con C3 en pantalla; no invitar a sumar las barras por-categoría.
