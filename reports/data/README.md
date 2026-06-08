# Gold agregada para Power BI (contrato §13.3)

CSVs pequeños que alimentan el **tablero ejecutivo** (Kelly). Los genera
`notebooks/pipeline/03_gold_agregada_bi_pyspark.ipynb` sobre la **base limpia** (cuarentena **14–17 nov**),
con las **mismas definiciones del EDA** → los números coinciden con `notebooks/analysis/eda_ecommerce.ipynb`.

> **⏳ Pendiente de regenerar (7-jun):** la cuarentena se amplió de 15–17 a **14–17 nov** (el 14 tenía un
> volcado de eventos que se escapó del criterio original; ver doc 00 §17, callout 7-jun). Hay que
> **re-correr el notebook** para regenerar estos CSV; los números de abajo aún excluyen solo 15–17.

> **Cómo se actualizan:** correr el notebook en Databricks → escribe los CSV en el Volume
> (`.../gold/bi_export`) → descargar y **commitear aquí**. Power BI los consume desde esta carpeta
> (excepción del `.gitignore`, doc 00 §12.2). **No** conectar Power BI a las 23M filas crudas.

| CSV | Pregunta de negocio | Columnas clave | Gráfico (doc 07 §5) |
|---|---|---|---|
| `agg_funnel_categoria` | ¿Dónde se concentra la fuga por categoría? **(Palanca A)** | `macro_category, units, reached_cart, purchased, revenue, revenue_en_juego, cart_rate, conv_rate, cierre_pct, abandono_pct, ticket_medio` | Treemap / barras |
| `agg_revenue_en_juego` | ¿Cuánto $ hay en carritos abandonados? **(Palanca A)** | `macro_category, carritos_abandonados, revenue_en_juego, ticket_medio` | Treemap (área = $) |
| `agg_marca_electronics` | ¿Qué marcas concentran el premio? | `brand, carritos, comprados, abandonados, abandono_pct, ticket` | Barras |
| `agg_segmentos_comprador` | ¿Qué segmento concentra el revenue? **(Palanca B)** | `segmento, n_compradores, revenue, ticket_promedio, pct_compradores, pct_revenue` | Combo doble eje |
| `agg_metricas_diarias` | ¿Cómo evoluciona conversión/revenue? **(global)** | `date, views, carts, purchases, revenue, conv_x100, is_black_friday, ventana_corrupta` | Líneas / áreas |
| `agg_metricas_diarias_categoria` | Evolución temporal **filtrable por categoría** (NUEVA, 7-jun) | `date, macro_category, views, carts, purchases, revenue, is_black_friday, ventana_corrupta` | Líneas con slicer de categoría |
| `agg_metricas_dia_hora` | Evolución temporal con **toggle día ↔ hora** (NUEVA, 8-jun) | `date, hour, views, carts, purchases, revenue, is_black_friday, ventana_corrupta` | Línea + *field parameter* (eje X: `date` o `hour`) |
| `agg_tipologia_visitante` | ¿Cómo se reparten browser/intender/buyer? | `tipo, n_sesiones, pct` | Barras |
| `agg_funnel_embudo` | Embudo global vistas→carrito→compra (NUEVA, 7-jun) | `etapa, n, pct_del_total, pct_paso_anterior, perdidos_vs_anterior` | **Funnel / waterfall** |
| `agg_hora_dow` | ¿Cuándo intervenir? hora × día de la semana (NUEVA, 7-jun) | `day_of_week, day_name, session_hour, is_weekend, n_sesiones, n_compras, conv_x100` | **Matriz / heatmap** |
| `agg_electronics_marca_diaria` | Top marcas electronics en el tiempo (NUEVA, 7-jun) | `date, brand, views, carts, purchases, revenue, is_black_friday, ventana_corrupta` | Líneas con slicer de marca |

**Notas para Kelly:**
- Todo en **base limpia** salvo `agg_metricas_diarias`, `agg_metricas_diarias_categoria` **y `agg_metricas_dia_hora`**, que
  **conservan todos los días** con `ventana_corrupta` y `is_black_friday` → úsalas para **anotar la
  calidad de datos** (**14–17 nov**) y Black Friday.
- **`agg_metricas_diarias_categoria` (NUEVA):** misma serie temporal pero al grano `date × macro_category`
  (incluye `Unknown`/"Sin taxonomía"). **Sumando todas las categorías reproduce exactamente
  `agg_metricas_diarias`** → úsala con un *slicer* de categoría para gráficos temporales interactivos.
  La **conversión** no viene como columna: créala como **medida** en Power BI (`SUM(purchases)/SUM(views)`),
  así respeta el filtro de categoría/fecha activo (es la práctica correcta en BI, no un % pre-agregado).
  *(El 14-nov ya quedó marcado `ventana_corrupta=1` en el CSV; este día es el que producía el pico de vistas atípico del gráfico temporal.)*
- **`agg_metricas_dia_hora` (NUEVA · toggle día↔hora):** misma serie pero al grano `date × hour`. Para el
  *toggle* crea un **field parameter** en Power BI con `{Día del mes = date, Hora del día = hour}` y ponlo
  en el eje X de la línea; las medidas (views/purchases/conversión-medida) se mantienen al cambiar el eje.
  Sumar sobre horas reproduce `agg_metricas_diarias`; para el **patrón intradía limpio** filtra `ventana_corrupta = 0`.
- ⚠️ **`agg_funnel_categoria` y `agg_revenue_en_juego` EXCLUYEN la categoría `Unknown`/"Sin taxonomía"**
  (~32% de las unidades, sin macro-categoría). Por eso **NO sumes sus filas para obtener el funnel global
  del titular**: el titular (cart **3.86%** / conv **2.27%** / abandono **41.19%**, **903k** carritos
  abandonados, **$250.4M** en juego) es sobre **56.76M** unidades **incl. Unknown**; sumar el per-categoría
  da ~4.33/2.58/40.3% sobre **37.3M** (sin Unknown). Para la **tarjeta KPI global** usa
  `agg_funnel_global.csv` (fila TOTAL incl. Unknown) — lo genera la celda "Funnel global" de
  `03_gold_agregada_bi_pyspark.ipynb`. Los cortes **por marca/categoría/segmento** sí cuadran al céntimo con el EDA.
  *(Números a la corrida 8-jun, cuarentena 14–17.)*
- Las **dos palancas** del EDA: **(A)** electrónica / carritos abandonados (**~$187.1M en juego**), **(B)**
  recurrentes (**35.7% compradores = 73.7% revenue**). Detalle e interpretación en `eda_ecommerce.ipynb` §4.9 y la Fase II.
- **¿Te falta un corte (filtro/cruce) que no está?** Pídelo y se añade una tabla más — es barato,
  el notebook ya tiene la base. Mejor una tabla agregada nueva que conectar el tablero a las filas crudas.

---

## Recomendaciones para el tablero (visualización · propósito + criterios de evaluación)

El jurado pesa **narrativa, diseño e interactividad** (defender filtrando en vivo). Ideas concretas con las tablas disponibles:

- **Slide temporal — toggle día ↔ hora (`agg_metricas_dia_hora`).** Un **único** visual (barras de tráfico + línea de conversión, doble eje) con un **field parameter** `{Día del mes = date, Hora del día = hour}` en el eje X y un slicer *"Ver por: día / hora"*. Conversión como **medida** (`SUM(purchases)/SUM(views)`) para que respete el eje activo. En modo hora, filtra `ventana_corrupta = 0`. *Por qué puntúa:* interactividad en vivo + diseño limpio (un visual) + técnica avanzada de PBI.
- **Narrativa "volumen ≠ conversión".** La conversión **cae en noviembre** porque el **tráfico pre-Black-Friday** sube más rápido que las compras (no es un bug; ver doc 00 §17.2). Anota la rampa: *"rampa pre-BF: el tráfico sube, la conversión baja"*. Con `agg_metricas_diarias_categoria` muestra que esa rampa la **lidera electronics**.
- **Embudo (`agg_funnel_embudo`).** Visual **funnel/waterfall** Vistas→Carrito→Compra → hace tangible la frase *"de 100 vistas perdemos 98, y 4 de cada 10 carritos"*. Pieza narrativa fuerte y legible.
- **Heatmap "cuándo intervenir" (`agg_hora_dow`).** Matriz día-de-semana × hora con color = conversión → da el *timing* accionable del A/B test (a qué hora pica la intención).
- **Pendiente (cuando Sara entregue clusters):** **matriz de oportunidad** (segmento: tamaño × abandono × valor) que aterriza el *"qué segmento = mayor oportunidad"* de la Pregunta de Oro. Es el cruce clasificador×clustering, el insight del proyecto.
- **Detalle/forma:** unificar separador decimal (hoy mezcla `2.24%` y `3,5%`); rotular "Ganancias en juego" como *"$ en riesgo / recuperable"*; añadir una frase-insight por página (*"electronics = 83% del $ en juego → objetivo del A/B"*).
