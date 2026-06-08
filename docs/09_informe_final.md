<!--
ESQUELETO DEL INFORME FINAL — PI Grupo 8 (2026-1)
Estructura tomada literalmente de docs/material_entrega/formatos_y_reglas_entrega_pi.docx.
Cada sección anota: [REQUISITO] qué rúbrica satisface · [FUENTE] de dónde sale el contenido · [ESTADO].
Borrar estos comentarios HTML antes de exportar a DOCX/PDF.

Reglas de entrega (recordatorio): documento consolidado DOCX/PDF + PPTX/PDF + .txt con el repo,
montados en el canal del TEAM → carpeta "Entrega Final". Exposición ≤ 20 min, todos intervienen.
-->

# Optimización de la conversión en e-commerce mediante modelado de propensión de compra
### Proyecto Integrador 1 · Maestría en Ciencia de Datos y Analítica · EAFIT · 2026-1

---

## Portada
<!-- [REQUISITO] Formato oficial. [FUENTE] doc 00 encabezado. [ESTADO] redactar -->
- **Título:** Optimización de la conversión en e-commerce mediante modelado de propensión de compra.
- **Autores:** Kelly … , Sara … , Heider Zapata, Yeison …  *(completar nombres y afiliaciones)*
- **Materias:** SI7009 Aprendizaje Automático · SI7006 Almac. y Proces. de Grandes Datos · SI7007 Visualización de Datos.
- **Fecha de entrega:** 8 de junio de 2026 · **Exposición:** 9 de junio de 2026.
- **Repositorio:** `HeiderZapata/proyecto-integrador-g8`.

---

## 1. Introducción
<!-- [REQUISITO] Estructura oficial. [FUENTE] doc 00 §1, §5. [ESTADO] redactar -->
- Contexto del problema (e-commerce, fuga de conversión) y motivación.
- Dataset REES46 (clickstream multi-categoría, Oct–Nov 2019, ~14.5 GB).
- **Frase de ascensor** y alcance del proyecto en una frase (doc 00 §1).
- Qué entrega el proyecto: propensión + clustering + diagnóstico + diseño de A/B test.

## 2. Marco teórico y referencias
<!-- [REQUISITO] Estructura oficial. [FUENTE] docs/05 (ML), doc 02 (Big Data), docs/07 (Viz). [ESTADO] redactar -->
- Clasificación supervisada y evento raro (PR-AUC, calibración). *(docs/05 §1)*
- Clustering / aprendizaje no supervisado (K-Means, silhouette, "cluster≠segmento"). *(docs/05 §1.4)*
- Anti-fuga y validación que respeta el tiempo. *(docs/05 §1.3)*
- Arquitectura de grandes datos (Medallion, Kappa, Delta Lake). *(doc 02)*
- Diseño experimental / por qué A/B y no uplift causal. *(doc 00 §5.1, docs/05 §5)*
- Referencias (REES46, papers/cursos, librerías).

## 3. Desarrollo metodológico de modelos de ML
<!-- [REQUISITO] SI7009: modelado sup+no sup, evaluación, selección, métricas, caso de uso. ESTE ES EL CORAZÓN PARA AA. -->

### 3.1 Entendimiento del problema, pregunta de negocio e hipótesis
<!-- [FUENTE] doc 00 §1, §5. [ESTADO] redactar -->
- **Pregunta de negocio** y **Pregunta de Oro** (doc 00 §1).
- Definición del target: unidad = sesión; positivo = la sesión contiene `purchase`.
- Por qué propensión y NO uplift causal (datos observacionales, sin tratamiento). *(doc 00 §5.1, §5.3)*

### 3.2 Análisis Exploratorio de Datos (EDA)
<!-- [REQUISITO] SI7009 + estructura oficial. [FUENTE] notebooks/analysis/eda_ecommerce + doc 00 §6, §17. -->
#### 3.2.1 Entendimiento de los datos
- Esquema, volumen (109.5M eventos; 56.76M unidades base limpia), grano. *(doc 00 §6, §17.1)*
#### 3.2.2 Preparación de los datos
- Limpieza, tipado, dedup, reconstrucción de sesión, corte anti-fuga. *(doc 00 §2.3.1, §17.5)*
- **Calidad de datos: cuarentena 14–17 nov** (criterio etiqueta-rota ∨ volumen anómalo). *(doc 00 §17.2, §17.5)*
- `"Unknown"`/taxonomía, precio acotado en fuente, sesiones-bot. *(doc 00 §17.5)*
#### 3.2.3 Análisis descriptivo e insights (correlación, causa-efecto)
- Funnel base limpia: cart 3.86%, conv 2.27%, abandono 41.19%. *(doc 00 §17.1)*
- Dos palancas de negocio (recuperar carritos electronics; retener recurrentes). *(doc 00 §17.1)*
- Concentración de revenue (electronics 76.9%), precio no es el freno, decisión <2 min. *(doc 00 §17.1)*
- Hallazgo contraintuitivo: features con correlación débil/negativa → señal no lineal. *(doc 00 §17.1)*

### 3.3 Selección de modelos, Ingeniería de Características, Entrenamiento, Evaluación
<!-- [REQUISITO] SI7009 núcleo. [FUENTE] notebooks/modeling/02,03,04 + doc 00 callout modelado + docs/05. -->
<!-- [ESTADO] ⏳ DEPENDE DE SARA: números finales tras re-correr con snapshot 14–17. Dejar placeholders [PR-AUC=…]. -->

#### 3.3.1 Características e Ingeniería de Características
- Set de 22 columnas de la Gold (anti-fuga, pre-corte). *(doc 00 §13)*
- FE: decisividad/ritmo (`revisit_intensity`…), codificación cíclica de hora, manejo de `category_id`. *(nb 02 §4)*
- Decisiones: features excluidas (IDs), `sin_navegacion_previa`, multicolinealidad. *(doc 00 §13, revisión modeling)*

#### 3.3.2 Modelos (selección)
- **Supervisado — propensión:** baseline trivial (Dummy) → logística → comparación de familias (RF/XGB/LightGBM) → LightGBM. *(nb 02)*
- **No supervisado — clustering:** K-Means k=4 (por accionabilidad) + DBSCAN como contraste. *(nb 04)*
- Justificación de cada elección (criterio defendible, no moda).

#### 3.3.3 Entrenamiento
- **Split temporal Opción C** (train Oct + Nov ≤23 / test 24–30 nov), con evidencia de drift/PSI. *(nb 03, doc 00 §17.3)*
- Cuarentena 14–17 aplicada a train y test; `StratifiedKFold` solo en CV interna; desbalanceo con `scale_pos_weight`. *(nb 02)*
- HPO con Optuna optimizando PR-AUC en CV (≥50 trials para el fit final). *(nb 02 §8)*

#### 3.3.4 Evaluación
- **Métricas:** PR-AUC + Brier (calibración), nunca accuracy. Reporte con/sin Black Friday. *(nb 02 §9–10)*
- Curva de calibración, curva PR, umbral como política operativa (no 0.5). *(nb 02 §9, §11)*
- Importancia por permutación → top features (`max_price_viewed`, `electronics_view_share`). *(nb 02 §12)*
- Clustering: perfilado, "cluster≠segmento", validación (silhouette + estabilidad). *(nb 04 §5, §9)*
- **El cruce clasificador × clustering** → segmento objetivo C0 (el insight). *(nb 04 §6, §9)*

### 3.4 Análisis y conclusiones del componente de ML
<!-- [FUENTE] nb 02/04 + doc 00. [ESTADO] depende de números finales. -->
- Lectura del resultado (qué predice la compra; nivel de precio y foco electrónica).
- Conexión con la decisión de negocio: segmento C0 → A/B test.
- Limitaciones y trabajo futuro (historia de usuario, uplift como extensión).

## 4. Tecnología: Ingeniería de Datos y uso de tecnología
<!-- [REQUISITO] SI7006 (obligatorio): ciclo de vida + arquitectura. [FUENTE] doc 02 + doc 00 §7. -->
### 4.1 Desarrollo del proyecto (arquitectura implementada)
- **Fuentes de datos y naturaleza:** REES46, clickstream, batch → replay streaming. *(doc 00 §7)*
- **Ingesta:** Auto Loader (`cloudFiles`) + `Trigger.AvailableNow` + checkpoint (Kappa). *(doc 00 §7, doc 02 §4)*
- **Almacenamiento:** Volume (object storage) → Medallion Bronze/Silver/Gold en **Delta Lake**. *(doc 02)*
- **Particionamiento por capa** (decisión por tamaño; Gold sin particionar). *(doc 00 §7, doc 02 §3)*
- **Framework de procesamiento:** Apache Spark / Spark SQL. *(doc 02)*
- **Persistencia de modelos:** MLflow (tracking + modelo calibrado + signature). *(doc 00 §18.4)*
### 4.2 Despliegue (escenario hipotético de implementación real)
- **Arquitectura de referencia productiva:** Kafka/Kinesis → streaming → Delta/S3 → serving tiempo real. *(doc 00 §7)*
- Scoring batch → Contrato 2 en Delta → consumo del tablero. *(doc 00 §13)*
- Por qué Volume y no S3 externo (decisión y Q&A). *(doc 00 §7)*

## 5. Visualización y comunicación de datos
<!-- [REQUISITO] SI7007 (rúbrica 35%: despliegue/funcionalidad/narrativa/defensa). [FUENTE] reports/powerbi + docs/07 + doc 00 §8. -->
- **Requerimientos de comunicación:** Pregunta de Oro, audiencia (jurado BA/Diseño). *(doc 00 §8)*
- **Análisis y diseño:** mapa pregunta→gráfico, paleta, narrativa. *(doc 00 §8, reports/data/README)*
- **Implementación:** Tablero Power BI (4 páginas) publicado en Power BI Service. *(doc 00 §2.3, reports/powerbi/)*
  - Análisis global · Detalle Electronics · Contexto temporal · (v2) matriz de oportunidad + scores. *(⏳ Contrato 2)*
- **Validación:** coherencia de cifras vs EDA (12 CSV verificados). *(doc 00 §17.1)*
- Enlace público del tablero desplegado (requisito de despliegue 10%).

## 6. Conclusiones generales del proyecto
<!-- [FUENTE] doc 00 §1, §5, §17 + resultados finales. [ESTADO] redactar al cierre. -->
- Respuesta a la Pregunta de Oro: dónde se concentra la fuga y qué segmento es la oportunidad.
- Recomendaciones accionables (las dos palancas + segmento C0 + A/B test).
- Aporte por materia (ML / Grandes Datos / Visualización).

## 7. Diseño del A/B test (cierre del alcance)
<!-- [REQUISITO] responde al revisor de la propuesta; alcance doc 00 §5.2(4), §18.7. [FUENTE] por redactar (Yeison). [ESTADO] PENDIENTE. -->
- Hipótesis, unidad de aleatorización, métrica primaria, MDE, tamaño de muestra/poder, duración.
- Segmento objetivo (C0) y cómo el modelo hace el targeting fino dentro del segmento.

## 8. Referencias
<!-- [FUENTE] consolidar. [ESTADO] redactar -->

---

<!--
MAPA DE COBERTURA DE LA RÚBRICA (verificación — borrar antes de exportar)
- SI7009 Aprendizaje Automático → §3 completo (sup §3.3.2, no sup §3.3.2, evaluación §3.3.4, métricas §3.3.4, caso de uso §3.1).
- SI7006 Grandes Datos → §4 completo (ciclo de vida, arquitectura ref §4.2, pipeline/ingesta/almacenamiento/Spark §4.1, despliegue/persistencia §4.2, visualización §5).
- SI7007 Visualización → §5 (despliegue + funcionalidad + narrativa) + la defensa se cubre en el ensayo/PPTX.

DEPENDENCIAS ABIERTAS
- §3.3.3/§3.3.4/§3.4: números finales del modelo (Sara, tras snapshot 14–17). Placeholders [..].
- §5: tablero v2 con scores (Kelly, tras Contrato 2).
- §7: diseño del A/B (Yeison) — se puede redactar ya.
-->
