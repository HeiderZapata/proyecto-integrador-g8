# Visualización de Datos — Tablero Ejecutivo Power BI

## 1. Introducción

El componente de visualización del proyecto opera en el **nivel ejecutivo**: su audiencia es el tomador de decisiones de negocio y su objetivo comunicativo es **convencer e informar** — responder, en pocos segundos de interacción, dónde se concentra la fuga de conversión y qué segmento de visitantes representa la mayor oportunidad de recuperarla.

El producto es un **tablero interactivo en Power BI Desktop**, publicado en Power BI Service, conectado a la capa Gold agregada del proyecto (doce CSVs generados desde Databricks sobre la base limpia con cuarentena 14–17 de noviembre de 2019). El tablero no consume las filas crudas del dataset (~14.5 GB); consume exclusivamente las tablas agregadas que encapsulan los hallazgos del EDA. Esto garantiza rendimiento, reproducibilidad y alineación entre lo que muestra el visual y lo que calculó el análisis.

El tablero se organiza en **seis páginas** que construyen una narrativa progresiva: parte del problema global, profundiza en la categoría más crítica, caracteriza al cliente, contextualiza temporalmente y cierra con los hallazgos y la propuesta de acción. La navegación entre páginas es explícita mediante botones con acciones de navegación configuradas en Power BI, lo que permite recorrer la historia de forma controlada durante la exposición y demostrar interactividad en vivo ante el jurado.

---

## 2. Principios de diseño aplicados

El diseño del tablero se rige por los principios del curso de Visualización de Datos. Se describen a continuación los más relevantes, con indicación de cómo se materializaron en el producto.

### 2.1 Visualización exploratoria vs. aclaratoria

El curso establece una distinción central: la visualización exploratoria sirve al analista para descubrir patrones; la aclaratoria sirve al tomador de decisiones para actuar. El tablero es completamente aclaratorio: cada visual responde una única pregunta de negocio, con un mensaje predeterminado y un argumento explícito. Las visualizaciones exploratorias del EDA (realizadas en Python/PySpark sobre Databricks) sirvieron para identificar los hallazgos; el tablero los comunica ya procesados.

### 2.2 Data-to-Ink Ratio y carga cognitiva

Toda "tinta" que no contribuye al mensaje es ruido. En el tablero se eliminaron grillas internas, bordes decorativos, títulos de ejes redundantes y leyendas cuando las etiquetas directas en las barras eran suficientes. El principio rector del curso se aplicó literalmente: si el espectador necesita más de cinco segundos para entender el mensaje central, falló el argumento, no el espectador.

### 2.3 Atributos preatentivos

El color es el atributo preatentivo más poderoso: se detecta antes de que el ojo lo procese conscientemente. En el tablero se utilizó una paleta semáforo con significado consistente en todas las páginas:

- **Verde (#3B6D11)** → lo positivo: categorías con alta conversión, el segmento recurrente, la oportunidad de recuperación.
- **Rojo (#A32D2D)** → la alerta: abandono de carrito, ganancias perdidas, categorías críticas.
- **Gris (#BBBBBB)** → el contexto: categorías de referencia, valores neutros que no son el insight.

Este sistema semáforo permite al espectador identificar el dato crítico sin leer ninguna etiqueta. El color se reservó exclusivamente para el elemento más importante de cada gráfica; el resto va en gris.

### 2.4 Principios Gestalt

- **Proximidad:** los KPIs relacionados se agrupan en fila en la parte superior de cada página; las gráficas que responden la misma pregunta se disponen en la misma fila.
- **Similitud:** el mismo tipo de gráfico y los mismos colores se usan para el mismo tipo de dato en todas las páginas (barras verdes siempre = volumen positivo, barras rojas siempre = pérdida).

### 2.5 Jerarquía visual y patrón Z

El patrón de lectura occidental va de arriba-izquierda hacia abajo-derecha. En cada página los KPIs más impactantes se ubican arriba a la izquierda o arriba al centro, y los detalles van bajando. Los títulos de las gráficas usan fuente Segoe UI negrita en color oscuro (#2D2D2D); las etiquetas de ejes y valores van en gris (#555555) para que no compitan con el dato principal.

### 2.6 Storytelling integrado y argumento visual

El curso propone el patrón obligatorio: **Contexto → Hallazgo técnico → Traducción de negocio → Acción**. Cada página del tablero cierra con una frase-insight en la parte inferior que sigue este patrón, convirtiendo el análisis en una recomendación accionable. Adicionalmente, las anotaciones de eventos clave (ventana corrupta 14–17 nov, Black Friday) se integran directamente en los gráficos temporales, siguiendo la recomendación del curso de no obligar al espectador a deducir el contexto.

### 2.7 Interactividad y revelación progresiva

El tablero incorpora un filtro dinámico de Top N categorías (valores 5, 10, 15) que controla simultáneamente dos gráficas mediante medidas DAX sincronizadas, demostrando filtrado en vivo durante la defensa. La navegación entre páginas usa botones con acción de navegación configurada, lo que permite recorrer la narrativa de forma controlada. Se evitó el "tablero de control de avión": cada página tiene como máximo un control de filtro visible y el espacio principal se reserva para la narrativa visual.

---

## 3. Páginas del tablero

### 3.1 Portada

**Pregunta que responde:** ¿De qué trata este análisis y cuál es la magnitud del problema?

**Acto de habla:** motivar — crear tensión narrativa antes de mostrar los datos.

**Estructura visual:**
La portada aplica un layout asimétrico de dos columnas: el título y la pregunta de investigación a la izquierda; cuatro tarjetas KPI apiladas a la derecha. Esta disposición sigue el principio del patrón Z: el ojo lee primero el contexto (izquierda) y simultáneamente procesa los números de impacto (derecha), completando el diagnóstico en menos de cinco segundos.

Las tarjetas KPI usan fondo rojo suave (#FCEBEB) con texto rojo oscuro (#A32D2D) — todos los indicadores son alertas del problema: tasa de conversión de 2.27%, abandono de carrito de 41.19%, 903 mil carritos abandonados y $250.4 millones en ganancias perdidas. El uso sistemático del rojo aplica el principio de similitud de Gestalt: el espectador entiende de inmediato que todos estos números son un problema, no un logro.

Los botones de navegación en la parte inferior derecha permiten ir directamente a cualquier sección del análisis. El botón activo (la página actual) tiene fondo negro (#2D2D2D) y los inactivos tienen fondo blanco con borde gris — consistente con la jerarquía visual del tablero.

> **Nota:** los visuales descritos pueden estar sujetos a ajustes menores antes de la versión final.

**Argumento visual:** *Una tienda online con 23 millones de sesiones pierde el 97.73% de sus visitas sin compra. $250.4M están atrapados en carritos que nadie cerró.*

---

### 3.2 Análisis por categoría

**Pregunta que responde:** ¿Dónde se concentra la fuga de conversión por categoría de producto?

**Acto de habla:** convencer — demostrar que el problema no está distribuido uniformemente sino concentrado en un punto específico.

**Estructura visual:**
Cuatro gráficas de barras horizontales organizadas en cuadrícula 2×2, más un filtro dinámico de Top N categorías (5, 10, 15) y una frase-insight de cierre.

Las barras horizontales se eligieron sobre las verticales porque los nombres de las categorías son largos y se leen mejor en horizontal; además, permiten alinear visualmente las cuatro gráficas en columna para comparar la posición de electronics en todas ellas de un vistazo. El orden de las barras es descendente en todas las gráficas (mayor a menor), siguiendo la recomendación del curso de reducir la fricción visual ordenando por valor.

Las cuatro gráficas son:
- **Valor promedio de compra por categoría (USD):** barras con semáforo verde/gris según el ticket.
- **Ganancias por categoría (%):** barras verdes — electronics con 85.5% del volumen de compras.
- **Tasa de conversión por categoría (%):** barras con electronics en verde oscuro, el resto en gris.
- **Ganancias perdidas por categoría (USD):** barras con electronics en rojo intenso — 83.3% de las pérdidas.

La tensión narrativa emerge de leer las cuatro gráficas en secuencia: electronics lidera en ticket, en compras y en conversión (verde) pero también concentra la mayoría de las ganancias perdidas (rojo). Esa paradoja es el insight central de la página.

El filtro Top N permite demostrar interactividad en vivo durante la defensa: el jurado puede ver cómo cambia la distribución al pasar de Top 5 a Top 15 categorías, sin que el mensaje central cambie.

> **Nota:** los visuales descritos pueden estar sujetos a ajustes menores antes de la versión final.

**Argumento visual:** *Electronics lidera en ventas (85.5%), ticket promedio de $412 y mayor tasa de conversión (3.6%), pero concentra el 83.3% de las ganancias perdidas. Es la categoría con mayor oportunidad de recuperación.*

---

### 3.3 Detalle Electronics

**Pregunta que responde:** Dentro de electronics, ¿qué marcas concentran el abandono de carrito?

**Acto de habla:** convencer — localizar el problema al nivel de marca para hacer accionable la intervención.

**Estructura visual:**
Cuatro tarjetas KPI en la parte superior (ticket promedio $412, abandono 41.5%, 511.884 carritos abandonados, $211.1M en ganancias perdidas) más dos gráficas en la mitad inferior.

Las tarjetas KPI de alerta (abandono y ganancias perdidas) tienen fondo rojo suave, mientras que el ticket promedio tiene color verde — aplicando el sistema semáforo de forma coherente con el resto del tablero.

Las dos gráficas son:
- **Barras horizontales Top N marcas por carritos abandonados:** Samsung (34.5%) y Apple (31.6%) en rojo oscuro, el resto en gris. La elección de barras horizontales sigue el mismo criterio que en la página anterior. El filtro Top N de marcas (5, 10, 20) controla esta gráfica dinámicamente.
- **Scatter plot ticket vs. tasa de abandono (tamaño = carritos perdidos):** cada burbuja es una marca; el tamaño de la burbuja codifica el volumen de carritos abandonados (atributo preatentivo de tamaño). El scatter permite identificar el cuadrante crítico: marcas con alto ticket y alto abandono son la mayor oportunidad de recuperación. Apple aparece en la zona de alto ticket (~$800) con abandono elevado; Samsung en ticket medio-alto (~$200) con mayor volumen de pérdida.

El scatter responde una pregunta que las barras no pueden responder: no basta con saber quién pierde más carritos (volumen), sino también cuánto vale cada carrito perdido (ticket). La combinación de los dos gráficos da la imagen completa.

El filtro Top N de marcas está sincronizado con las barras; el scatter mantiene Top 10 fijo para no saturar el visual con burbujas ilegibles.

> **Nota:** los visuales descritos pueden estar sujetos a ajustes menores antes de la versión final.

**Argumento visual:** *Samsung y Apple concentran el 66% de los carritos abandonados en Electronics — con tickets promedio de $200-$800, representan la mayor oportunidad de recuperación.*

---

### 3.4 Análisis de cliente

**Pregunta que responde:** ¿Quién compra y cuánto vale? ¿Qué impacto tendría convertir compradores únicos en recurrentes?

**Acto de habla:** motivar — mostrar la oportunidad económica de intervenir en el segmento correcto.

**Estructura visual:**
Dos visuales principales más un espacio reservado para los resultados del modelo de clustering.

El primero es un conjunto de **tres tarjetas** para la tipología de visitante (browser 90.12%, intender 3.92%, buyer 5.97%). La tarjeta central (intender) se destaca con fondo verde suave (#EAF3DE) y texto verde oscuro (#3B6D11) — es el segmento objetivo del modelo de propensión: tiene intención de compra pero no la cierra. Las otras dos tarjetas tienen fondo blanco y texto gris oscuro. Esta jerarquía visual dirige la atención sin necesidad de texto explicativo adicional.

El segundo es un **diagrama Sankey** que muestra el flujo de compradores hacia revenue. El Sankey se eligió porque el curso lo identifica explícitamente como el gráfico adecuado para representar flujos entre etapas con ancho de banda proporcional a la cantidad. La historia del Sankey es: el 64.3% de los compradores son de única vez pero generan solo el 26.3% de las ganancias; el 35.7% recurrente genera el 73.7%. La franja verde claro muestra el impacto proyectado de convertir el 10% de compradores únicos en recurrentes: +13.3% al revenue recurrente. Los datos hipotéticos de la proyección se construyeron a partir de los datos reales de ticket promedio por segmento y se presentan explícitamente como proyección, no como resultado observado.

> **Nota:** esta página está **pendiente de terminar**. Falta integrar los resultados del modelo de clustering (segmentos C0–C3) y la matriz de oportunidad (tamaño × conversión × valor por segmento) que constituyen el insight central del proyecto: el cruce clasificador × clustering.

**Argumento visual:** *El 3.92% con intención de compra y el comprador recurrente — que representa el 35.7% pero genera el 73.7% de las ganancias — son los dos segmentos con mayor potencial de recuperación.*

---

### 3.5 Contexto temporal

**Pregunta que responde:** ¿Cómo evolucionó la conversión en el tiempo y qué eventos la afectaron?

**Acto de habla:** informar — establecer que el problema de conversión es estructural, no estacional.

> **Nota:** esta página está **pendiente de terminar**. Se construirá con las siguientes tablas agregadas nuevas: `agg_funnel_embudo` (embudo Vistas→Carrito→Compra), `agg_metricas_dia_hora` (toggle día↔hora con field parameter), `agg_hora_dow` (heatmap hora × día de semana) y `agg_metricas_diarias_categoria` (serie temporal por categoría con slicer). Los gráficos planeados son: gráfico combo doble eje (barras de tráfico + línea de conversión) con anotaciones de la ventana corrupta (14–17 nov) y Black Friday (29 nov), y un heatmap de conversión por hora y día de semana.

---

### 3.6 Cierre

**Pregunta que responde:** ¿Qué encontramos y qué recomendamos hacer?

**Acto de habla:** motivar — cerrar la narrativa con la llamada a la acción.

> **Nota:** esta página está **pendiente de terminar**. Contendrá los tres hallazgos principales del proyecto (concentración en electronics, paradoja del comprador recurrente, perfil del visitante con intención) y los próximos pasos (diseño del A/B test sobre el segmento C0 del clustering, intervención con incentivo sobre visitantes con alta propensión de compra).

---

## 4. Interactividad y navegación

El tablero incorpora los siguientes elementos interactivos demostrados en vivo durante la defensa:

- **Filtro Top N categorías** (valores 5, 10, 15): controla simultáneamente las cuatro gráficas de la página de análisis por categoría mediante medidas sincronizadas.
- **Filtro Top N marcas** (valores 5, 10, 20): controla la gráfica de barras en la página de detalle Electronics.
- **Botones de navegación entre páginas**: presentes en todas las páginas, permiten recorrer la narrativa sin usar las pestañas de Power BI, manteniendo el control del presentador durante la exposición.

La interactividad en vivo es uno de los tres pilares de evaluación del curso (exploración, mensaje, producto). El diseño del tablero privilegió la demostración de filtrado en vivo sobre la cantidad de visuales estáticos.

---

## 5. Herramientas y conexión a datos

**Herramienta principal:** Power BI Desktop (versión de escritorio) con publicación en Power BI Service mediante cuenta institucional EAFIT.

**Fuente de datos:** doce archivos CSV generados por el notebook `03_gold_agregada_bi_pyspark.ipynb` sobre la capa Gold del proyecto (cuarentena 14–17 de noviembre, base limpia). Los CSV residen en `reports/data/` del repositorio y están versionados en Git. Power BI los consume desde la ruta local del repositorio clonado, sin conexión directa a las tablas Delta de Databricks ni a los 23 millones de filas crudas de la Gold.

**Modelo de datos en Power BI:** se construyó una tabla de dimensión `dim_categoria` que actúa como puente entre las tablas de funnel por categoría y revenue en juego, permitiendo que el filtro Top N controle ambas con una sola selección. Las demás tablas son independientes entre sí y se conectan directamente a sus visuales correspondientes.
