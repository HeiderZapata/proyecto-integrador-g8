# Gold agregada para Power BI (contrato §13.3)

CSVs pequeños que alimentan el **tablero ejecutivo** (Kelly). Los genera
`notebooks/pipeline/03_gold_agregada_bi.ipynb` sobre la **base limpia** (cuarentena 15–17 nov),
con las **mismas definiciones del EDA** → los números coinciden con `notebooks/analysis/eda_ecommerce.ipynb`.

> **Cómo se actualizan:** correr el notebook en Databricks → escribe los CSV en el Volume
> (`.../gold/bi_export`) → descargar y **commitear aquí**. Power BI los consume desde esta carpeta
> (excepción del `.gitignore`, doc 00 §12.2). **No** conectar Power BI a las 23M filas crudas.

| CSV | Pregunta de negocio | Columnas clave | Gráfico (doc 07 §5) |
|---|---|---|---|
| `agg_funnel_categoria` | ¿Dónde se concentra la fuga por categoría? **(Palanca A)** | `macro_category, units, reached_cart, purchased, cart_rate, conv_rate, cierre_pct, abandono_pct` | Treemap / barras |
| `agg_revenue_en_juego` | ¿Cuánto $ hay en carritos abandonados? **(Palanca A)** | `macro_category, carritos_abandonados, revenue_en_juego, ticket_medio` | Treemap (área = $) |
| `agg_marca_electronics` | ¿Qué marcas concentran el premio? | `brand, carritos, comprados, abandonados, abandono_pct, ticket` | Barras |
| `agg_segmentos_comprador` | ¿Qué segmento concentra el revenue? **(Palanca B)** | `segmento, n_compradores, revenue, ticket_promedio, pct_compradores, pct_revenue` | Combo doble eje |
| `agg_metricas_diarias` | ¿Cómo evoluciona conversión/revenue? | `date, views, carts, purchases, revenue, conv_x100, is_black_friday, ventana_corrupta` | Líneas / áreas |
| `agg_tipologia_visitante` | ¿Cómo se reparten browser/intender/buyer? | `tipo, n_sesiones, pct` | Barras |

**Notas para Kelly:**
- Todo en **base limpia** salvo `agg_metricas_diarias`, que **conserva todos los días** con
  `ventana_corrupta` y `is_black_friday` → úsalas para **anotar la calidad de datos** (15–17 nov) y Black Friday.
- ⚠️ **`agg_funnel_categoria` y `agg_revenue_en_juego` EXCLUYEN la categoría `Unknown`/"Sin taxonomía"**
  (~32% de las unidades, sin macro-categoría). Por eso **NO sumes sus filas para obtener el funnel global
  del titular**: el titular (cart **3.93%** / conv **2.24%** / abandono **43.1%**, **994k** carritos
  abandonados, **$283.6M** en juego) es sobre **58.6M** unidades **incl. Unknown**; sumar el per-categoría
  da ~4.40/2.54/42.35% sobre **38.6M** (sin Unknown). Para la **tarjeta KPI global** usa
  `agg_funnel_global.csv` (fila TOTAL incl. Unknown) — lo genera la celda "Funnel global" de
  `03_gold_agregada_bi.ipynb`. Los cortes **por marca/categoría/segmento** sí cuadran al céntimo con el EDA.
- Las **dos palancas** del EDA: **(A)** electrónica / carritos abandonados (**~$211M en juego**), **(B)**
  recurrentes (**35.8% compradores = 73.9% revenue**). Detalle e interpretación en `eda_ecommerce.ipynb` §4.9 y la Fase II.
- **¿Te falta un corte (filtro/cruce) que no está?** Pídelo y se añade una tabla más — es barato,
  el notebook ya tiene la base. Mejor una tabla agregada nueva que conectar el tablero a las filas crudas.
