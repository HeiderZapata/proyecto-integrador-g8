# Prompt para crear la PPT de defensa — PI Grupo 8

> **Cómo usar:** copia este prompt completo en un chat nuevo del Proyecto de Claude (que ya tiene el knowledge del repo). Si el chat no tuviera contexto, este prompt es **autocontenido**: trae los números y decisiones clave. Pídele que itere la PPT slide por slide.

---

## ROL Y OBJETIVO

Actúa como un experto en **comunicación de datos y storytelling ejecutivo** que domina la gramática visual del curso de Visualización (EAFIT, SI7007). Tu tarea: diseñar la **presentación de defensa (PPT) de 20 minutos** del Proyecto Integrador 1 del Grupo 8, ante un jurado de tres profesores (Aprendizaje Automático, Grandes Datos, Visualización).

El entregable es un **guion slide por slide**: para cada diapositiva — (1) título, (2) **mensaje único** en una frase, (3) bullets de contenido, (4) **propuesta de visual** (qué gráfico/diagrama y por qué), (5) **notas del orador** (lo que se dice), (6) **tiempo estimado**. La PPT es el **producto** y lleva el peso narrativo; el tablero se muestra en vivo solo al final.

---

## CONTEXTO DEL PROYECTO (autocontenido)

**Tema:** Optimización de la conversión en e-commerce mediante modelado de propensión de compra, sobre el dataset abierto **REES46** (clickstream multi-categoría, Oct–Nov 2019, ~14.5 GB, **109.95 M eventos** view/cart/purchase).

**Pregunta de Oro (abrir y cerrar con ella):** *¿Dónde se concentra la fuga de conversión y qué segmento de visitantes representa la mayor oportunidad de recuperarla?*

**Frase de ascensor:** La tienda pierde **98 de cada 100 visitas** y **4 de cada 10 carritos**. No construimos un modelo para adivinar quién compra; construimos un **sistema de decisión** para entender dónde y por qué se fuga la conversión, qué visitantes la explican, y dónde —y con qué experimento— intervenir. *La predicción es el instrumento; la decisión de negocio es el producto.*

**Las 4 piezas:** (1) clasificador de propensión (sesión); (2) clustering de visitantes; (3) diagnóstico del funnel + importancia de variables; (4) diseño de un **A/B test** (no estimamos uplift causal: el dato es observacional).

**Números clave (verificados, úsalos tal cual):**
- **Datos:** Gold = 22.99 M sesiones; base limpia (cuarentena 14–17 nov) = **19.71 M**, tasa de compra **0.0597**.
- **Funnel (base limpia, producto-en-sesión):** cart 3.86 %, conversión 2.27 %, abandono 41.19 %.
- **La paradoja de la electrónica:** mayor volumen y mayor pool de carritos abandonados (**~$187 M de los $250 M** en juego); Samsung+Apple = 68.6 % de carritos de electrónica.
- **Núcleo recurrente:** 35.7 % de compradores generan **73.7 % del revenue** (ticket recurrente $1,460 vs one-time $289).
- **Insight contraintuitivo:** el precio no es el freno y la compra se decide en **~2.2 min** (mediana); los compradores navegan *menos*.
- **Modelo (cadena defendible):** Dummy 0.056 (trampa del accuracy) → logística 0.097 → familias RF/XGB/LGBM empatadas 0.116–0.119 → **LightGBM+Optuna**: CV **0.1235 ± 0.0014** (5 folds, baja varianza) → **test out-of-time 0.1172** (ex-Black-Friday **0.1115**), **Brier 0.0515** (calibrado). Test ≈ CV ⇒ sin fuga ni sobreajuste.
- **Umbral operativo (no 0.5):** F1-óptimo **0.092** → marca 15 % de sesiones, precisión ~12 %, **lift ~2.2×** sobre la base 5.6 %.
- **Qué predice la compra (importancia por permutación):** `max_price_viewed`, `electronics_view_share`, `avg_price_viewed` → *qué tan caro/electrónico es lo que miras, no cuánto navegas*.
- **Clustering (k=5 por accionabilidad; silhouette 0.30; DBSCAN = 1 masa + 3.7 % ruido):** **C3 «electrónica gama media»** (29.3 % tráfico, conversión **8.4 %** = la mayor) = **OBJETIVO del A/B**; C4 premium ($1,024, 5.8 %); C0 general bajo valor (49.8 %); C1 explorador que no cierra (7 %); C2 anómalo (calidad de datos).
- **El cruce (el insight central):** el bloque de electrónica (C3+C4) combina propensión + valor + volumen; **dentro de él, el modelo de propensión hace el targeting fino sesión a sesión** (Contrato 2: `user_session`, `prob_calibrada`, `segmento` — 19.71 M filas en Delta).
- **A/B:** target C3; aleatorización **por visitante**; métrica primaria = conversión por visitante; **guardarraíl de margen**; MDE +0.5 pp ⇒ ~99k visitantes/brazo, alcanzable en pocos días.
- **Arquitectura:** Medallion sobre **Delta Lake** (Bronze→Silver→Gold); ingesta **Kappa** (Auto Loader `cloudFiles` + `Trigger.AvailableNow`); **particionamiento por tamaño** (Bronze/Silver por fecha + ZORDER; Gold sin particionar, ~1.33 GB); **entrenamiento local + scoring distribuido en Databricks** (pandas UDFs, sin OOM); **MLflow** (params + PR-AUC/Brier + modelo calibrado + signature); **Contrato 2** en Delta. Dos arquitecturas: **referencia productiva** (Kafka→Flink→S3) vs **implementada** (Databricks Free).
- **Tablero:** Power BI, **7 páginas**, desplegado en Power BI Service (enlace público), conectado a la Gold agregada (12 CSV).

---

## JURADO Y SUS TENDENCIAS DE Q&A (anticipar en el diseño)

**Aprendizaje Automático (Prof. Marco Terán):** premia *criterio defendible* sobre virtuosismo. Tendencias (de defensas previas): la **pregunta de negocio clara desde el inicio**; **justificar el split** (¿por qué temporal?, ¿por qué uno solo?, ¿por qué ese corte?); **rangos de hiperparámetros** evaluados; **el split se hace ANTES de construir features** (anti-fuga); SHAP/importancia: el valor no es repetir el ranking sino **leer dirección+magnitud** para entender el negocio. Frases-ancla suyas: *"el modelo estima; el umbral decide"*, *"si el test elige el modelo, deja de ser test"*, *"nunca digas «mejor modelo» sin decir según qué métrica, validación y comparación"*, *"cluster ≠ segmento"*.

**Grandes Datos (SI7006):** quiere ver el **ciclo de vida del dato** y el pipeline. Tendencias: en el flujo debe **verse la frontera entrenar/testear**; **estrategia de particionamiento por capa** (afecta performance y **cuota**); **escenario ideal vs. desarrollado** (referencia vs implementada) — *esto lo premian explícitamente*.

**Visualización (jurado de BA/HPC + Diseño):** el **objetivo comunicativo es convencer e informar**, y **la PPT es el foco, más que el tablero**. Tendencias: **nombres de variables claros**; por el proyector, **marcar bien los ejes y explicar desde los ejes**; el tablero debe tener **un único público + objetivo comunicativo claro**; **usar los gráficos recomendados** según el tipo de problema; plantillas del argumento: *"Declaración + conector + razón"*, *"Aumentó X debido a Y + pregunta"*.

---

## CRITERIOS DE EVALUACIÓN A CUBRIR (PI = 35 %)

**Rúbrica de Visualización (la que califica la exposición):**
1. **Despliegue (10 %):** app desplegada en la nube, accesible por enlace público, sin caídas. → mostrar el tablero en vivo desde el enlace.
2. **Funcionalidad y diseño (10 %):** fluidez, filtros que responden, **diseño visual altamente profesional y coherente**.
3. **Estructura y narrativa del pitch (10 %):** discurso **enfocado en el negocio**, responde la Pregunta de Oro con **hallazgos claros + recomendaciones accionables + storytelling**. (Penaliza el "resumen técnico del código".)
4. **Defensa y Q&A (5 %):** responder con seguridad **respaldando con los datos del tablero**.

**Componentes que cada materia exige ver (deben aparecer en la PPT):**
- **ML:** modelado (sup/no-sup), evaluación y **selección de modelos**, **métricas relevantes** (PR-AUC/Brier, no accuracy), caso de uso.
- **Grandes Datos:** ciclo de vida del dato, arquitectura de referencia, ambiente tecnológico, pipeline (origen→ingesta→almacenamiento→procesamiento→despliegue), persistencia de modelos.
- **Visualización:** aplicación de los conceptos del curso (abajo).

---

## PRINCIPIOS DE DISEÑO VISUAL (OBLIGATORIO — del curso SI7007)

El diseño **debe** respetar la gramática visual del curso. Aplícalos a cada slide:

- **Aclaratorio, no exploratorio:** cada gráfica = **un solo mensaje**. Nada de "vomitar" todas las gráficas del EDA. *Si toma >5 s entender el mensaje central, falló el argumento.*
- **Anatomía del argumento (5 capas):** Dato → Contexto → Patrón → Significado → **Llamada a la acción**. Toda gráfica termina en traducción de negocio.
- **Data-to-Ink:** elimina grillas, bordes, ruido. **Menos es más.**
- **Atributos preatentivos:** **color solo para el elemento más importante** (resto en grises); **tamaño** refuerza magnitud; **posición** arriba-izquierda = insight (patrón Z).
- **Gestalt:** proximidad y espacio en blanco para agrupar; similitud de color = misma categoría.
- **Gramática visual:** **contraste** (grises = contexto, color institucional vibrante = el dato a resaltar); **jerarquía** (dile dónde mirar 1°/2°/3°); **barras ordenadas por valor** (salvo orden intrínseco); **ejes marcados y legibles** (crítico por el proyector).
- **Anotaciones integradas:** el mensaje va **dentro** de la gráfica (flecha/etiqueta), no se deduce.
- **Arco narrativo:** Inicio (contexto) → Conflicto (insight/tensión) → Resolución (acción).
- **Acto de habla:** la PPT **convence e informa** (no solo describe).
- **Paleta del tablero (mantener coherencia):** gris oscuro `#2D2D2D` estructural, **verde `#3B6D11`** = oportunidad, **rojo `#A32D2D`** = pérdida/fuga, gris `#BBBBBB` = contexto.

---

## RESTRICCIONES

- **20 minutos en total**, todos los integrantes intervienen (4: Yeison, Heider, Kelly, Sara).
- **Reserva ~2 min al final para el tablero en vivo** (desde el enlace público, filtrando en vivo) → así se cubre el criterio de "despliegue + defensa con datos del dashboard".
- Apunta a **~15–18 slides** (ritmo ~1 min/slide + demo). No saturar.
- **Legibilidad de proyector:** fuentes grandes, alto contraste, ejes claros, un mensaje por slide.
- **Repartir la narración por frente** según roles (modelado: Sara/Yeison; datos/arquitectura: Heider; visualización/tablero: Kelly; orquestación/A-B: Yeison) — pero la **historia es una sola y continua**.

---

## ESTRUCTURA NARRATIVA PROPUESTA (ajústala, no la copies ciega)

1. **Portada** — título, autores, las 3 materias, fecha. (15 s)
2. **El problema** — la fuga: 98/100 visitas, 4/10 carritos; **Pregunta de Oro** en grande. Acto: crear tensión. (1.5 min)
3. **Los datos** — REES46, escala (109.95 M eventos / 14.5 GB), unidad = sesión. Encuadre Big Data. (1 min)
4. **Arquitectura de datos** — Medallion en Delta + Kappa; **referencia vs implementada**; particionamiento por tamaño; frontera train/test visible. *(Cubre Grandes Datos.)* (2 min)
5. **EDA / hallazgos** — funnel + **paradoja electrónica** (rojo solo en electronics) + recurrentes (Sankey) + decisión rápida. Cada uno con traducción de negocio. (2.5 min)
6. **Estrategia de modelado** — la cadena: target → baseline (Dummy = trampa del accuracy) → **PR-AUC** → **split temporal anti-fuga (antes de features)**. (2 min)
7. **Resultados del modelo** — tabla de comparación + test ≈ CV (sin fuga) + **calibración (Brier)** + **umbral operativo** (lift 2.2×). (2 min)
8. **Qué predice la compra** — importancia por permutación; **leer dirección** (precio/electrónica manda) → insight de negocio. (1.5 min)
9. **Segmentos (clustering)** — los 5 clusters; **C3 = objetivo** (mayor conversión + volumen). "cluster ≠ segmento". (1.5 min)
10. **El cruce** — clasificador × clustering: el segmento dice *dónde*, el modelo dice *a quién*. (1.5 min)
11. **El experimento (A/B)** — hipótesis, target C3, aleatorización por visitante, guardarraíl, tamaño/duración. Cierra el alcance (uplift→A/B). (1.5 min)
12. **Despliegue tecnológico** — scoring distribuido, MLflow, **Contrato 2** en Delta que alimenta tablero y A/B. (1 min)
13. **Conclusiones** — responder la **Pregunta de Oro** + recomendaciones accionables (palancas A y B) + aporte por materia. (1.5 min)
14. **Demo del tablero en vivo** (~2 min) — abrir el enlace, filtrar en vivo, responder la Pregunta de Oro desde el dashboard.
15. **Cierre / Q&A.**

---

## QUÉ **NO** HACER

- **No** prometer uplift causal / estimación de incentivos (el dato es observacional → entregamos el **diseño** del A/B).
- **No** reportar accuracy ni "mejor score" sin métrica+validación+comparación.
- **No** enterrar la pregunta de negocio ni volver la PPT un "resumen técnico del código".
- **No** saturar slides ni usar color decorativo (color = solo el dato clave).
- **No** llamar "segmento" a cualquier cluster sin nombrarlo y accionarlo.
- **No** presentar SHAP/importancia como "lo visto en clase" sin la lectura de dirección/negocio.

---

## FORMATO DEL ENTREGABLE

Para **cada slide**, entrega:

```
### Slide N — [Título]
- **Mensaje único:** (una frase, lo que deben recordar)
- **Contenido:** (bullets concisos para la slide)
- **Visual:** (qué gráfico/diagrama, ejes, qué va en color vs gris, anotación clave)
- **Notas del orador:** (qué se dice; incluir la traducción de negocio y, si aplica, la frase-ancla del profe)
- **Tiempo:** (mm:ss) — **Quién presenta:** (rol)
```

Al final: un **mini-banco de Q&A** (3–5 preguntas probables por profesor) con respuesta de 2–3 frases.

**Material de apoyo en el repo** (para profundizar): `docs/05` (estándar ML/Terán), `docs/07` (gramática visual), `docs/02` (arquitectura/particionamiento/cuota), `docs/08` (Q&A por profesor), `entrega_final/Informe_Final_PI_Grupo8_plantilla.docx` (contenido completo), `reports/powerbi/` (capturas del tablero).
