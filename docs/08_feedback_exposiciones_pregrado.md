# Feedback de exposiciones de pregrado — anticipación para la defensa G8

> **Qué es esto:** notas tomadas observando las sustentaciones de **pregrado**. No es
> feedback a nuestro proyecto, sino lo que los profes **preguntaron, criticaron y
> premiaron**. Sirve como **predictor del Q&A** y como **checklist** para nuestra propia
> exposición (9-jun). Está organizado por las tres materias del PI.
>
> Las secciones 1–4 son las notas (limpiadas y agrupadas). La **§5 es interpretación de
> G8** (acciones e implicaciones cruzadas con el doc 00 y el doc 02); es separable.

---

## 1. Aprendizaje Automático (SI7009)

**Preguntas / críticas:**
- La **pregunta de negocio** debe quedar clara **desde el inicio**, tanto en la PPT como en el documento.
- **Justificar la partición:** ¿por qué esa proporción de split?, ¿por qué **un solo** split?, ¿por qué **ese** en particular?
- **Rangos de hiperparámetros** en modelos de árboles: ¿sobre qué rangos se evaluaron?
- **SHAP vs. importancia nativa del árbol:** ¿por qué usar SHAP para *feature importance* si los modelos de árboles ya dan la importancia? El valor de SHAP no es repetir el *ranking*, sino el **gráfico de impacto** (dirección + magnitud por variable). Hacerle **lectura estadística al gráfico de impacto** (no al de importancia) para ir **entendiendo el negocio**.
- Observación a pregrado: **hicieron el split en la última etapa** del pipeline → señal de posible fuga / orden incorrecto.

---

## 2. Visualización de Datos (SI7007)

**Preguntas / críticas:**
- El **objetivo comunicativo es convencer e informar**. La **PPT es el foco** para lograrlo y debe ser el objetivo principal, **más que el tablero**.
- **Nombres de variables claros e intuitivos** en la PPT y en el tablero.
- Problema del **proyector**: **marcar bien los ejes** y explicar la gráfica **a partir de los ejes** (ejes → contexto).
- El **tablero** debe tener **un único público** y un **objetivo comunicativo claro** (¿informar, convencer, motivar?).
- **Usar los gráficos recomendados en el curso** según el tipo de problema a visualizar.

**Plantillas del argumento visual (úsalas en la PPT y el tablero):**
- **Declaración + conector + razón.**
- **"Aumentó X debido a Y" + pregunta.**
- **Dashboard → público objetivo + objetivo comunicativo.**

---

## 3. Almacenamiento y Procesamiento de Grandes Datos (SI7006)

**Preguntas / críticas:**
- En el **flujo de datos** debe verse la **diferencia entre el momento de entrenar y el de testear**.
- ¿Qué **estrategia de particionamiento** siguieron para **cada capa** de la arquitectura Medallion? El particionamiento **afecta el performance** (tiempo de respuesta de queries) → **optimiza presupuesto/cuota**.

---

## 4. Lo que premiaron / ideas a considerar

- **Diagrama de flujo** claro.
- **Capas de seguridad** (p. ej. correo para reportar inconsistencias).
- **Asignador/controlador de permisos y accesos.**
- **SHAP** para *feature importance*.
- Para Big Data: mostrar el **escenario ideal vs. el desarrollado** (adaptado/acotado al problema), demostrando **cómo se desarrollaría** en el caso aplicado.
- Visualización: mostrar **cómo varía la probabilidad según las características**.
- **Ensayar la proyección antes**, para garantizar que todo se vea bien al proyectar.

---

## 5. Implicaciones y acciones para G8 *(interpretación; cruzada con doc 00 / 02)*

**Aprendizaje Automático**
- Pregunta clara desde el inicio → ya tenemos la **Pregunta de Oro** (doc 00 §2); abrir PPT y documento con ella, no enterrarla.
- Split → ya usamos **split temporal** (Oct entrena / Nov prueba) + **corte anti-fuga** (doc 00 §4). Preparar la justificación: por qué **temporal** (replica producción, evita fuga de futuro) y por qué eso en vez de CV aleatoria. **Clave:** dejar claro que el corte/split se hace **antes** de construir features —justo el error que les marcaron a pregrado—.
- Hiperparámetros → documentar la **grilla/rangos** evaluados, aunque sea simple.
- SHAP → ya está en plan (doc 00 §3). Usar el **beeswarm/impacto** y leer **dirección del efecto** para el negocio, no solo el ranking de importancia.

**Big Data**
- **GAP (ya cubierto):** la estrategia de particionamiento por capa Medallion —pregunta probable que, además, **ata directo a la cuota de Databricks** (particionar bien = menos cómputo escaneado = no agotar la cuota)— quedó **documentada en el doc 02 §3** y firmada en la propuesta corregida (Curso 2). Repasarla para la defensa.
- Entrenar vs. testear visible en el flujo → reflejar el **split temporal** en el diagrama de pipeline/arquitectura.
- "Escenario ideal vs. desarrollado" → **es exactamente nuestra narrativa de dos arquitecturas** (referencia vs. implementada, doc 02). Reforzarla: es justo lo que premian.

**Visualización**
- PPT como foco / tablero con único público → para Kelly: definir **público y objetivo comunicativo** del tablero, y que la **PPT lleve el peso narrativo**.
- "Cómo varía la probabilidad según las características" → encaja con **SHAP dependence** y con un visual del tablero. Conecta el frente de ML con el de Visualización.
- Plantillas del argumento visual → aplicarlas a la **Pregunta de Oro** ("se concentra la fuga en X debido a Y → ¿dónde intervenir?").

**Transversal**
- **Ensayar la proyección** antes del 9-jun (el doc 00 §6 ya pide ensayar el filtrado en vivo). Agendarlo.
- Las "capas de seguridad" y el "controlador de permisos/accesos" que premiaron → mencionables en la **arquitectura de referencia** (gobernanza con Unity Catalog) sin tener que implementarlos.
