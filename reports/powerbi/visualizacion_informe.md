# Visualización de Datos — Tablero Ejecutivo Power BI

## 1. Introducción

El componente de visualización opera en el nivel ejecutivo: está dirigido al tomador de decisiones de negocio y tiene como objetivo comunicativo convencer e informar. La pregunta que debe responder en pocos segundos de interacción es dónde se concentra la fuga de conversión y qué segmento de visitantes representa la mayor oportunidad de recuperarla.

El producto es un tablero interactivo construido en Power BI Desktop y publicado en Power BI Service mediante cuenta institucional EAFIT. No consume las filas crudas del dataset (14,5 GB); trabaja exclusivamente con doce tablas agregadas generadas desde Databricks sobre la capa Gold del proyecto, con cuarentena del 14 al 17 de noviembre de 2019 aplicada. Esto garantiza rendimiento, reproducibilidad y coherencia entre lo que muestran los visuales y lo que calculó el análisis.

El tablero se organiza en siete páginas que construyen una narrativa progresiva: arranca con una portada de contexto, plantea el problema en números concretos, profundiza en el análisis por categoría y por marca, caracteriza al cliente, contextualiza el comportamiento temporal y cierra con la respuesta directa a la pregunta de oro del proyecto. La navegación se hace mediante un panel lateral izquierdo con botones numerados, visible en todas las páginas, que permite recorrer la historia de forma controlada durante la exposición y demostrar interactividad en vivo ante los evaluadores.

---

## 2. Principios de diseño aplicados

El diseño del tablero se rige por los principios del curso de Visualización de Datos. A continuación se describen los más relevantes con indicación de cómo se materializaron en el producto.

### 2.1 Visualización exploratoria vs. aclaratoria

El curso establece una distinción central: la visualización exploratoria sirve al analista para descubrir patrones; la aclaratoria sirve al tomador de decisiones para actuar. El tablero es completamente aclaratorio. Cada visual responde una única pregunta de negocio con un mensaje predeterminado y un argumento explícito. Las visualizaciones exploratorias del EDA, realizadas en Python y PySpark sobre Databricks, sirvieron para identificar los hallazgos; el tablero los comunica ya procesados.

### 2.2 Data-to-Ink Ratio y carga cognitiva

Toda tinta que no contribuye al mensaje es ruido. En el tablero se eliminaron grillas internas, bordes decorativos, títulos de ejes redundantes y leyendas cuando las etiquetas directas sobre las barras eran suficientes. El principio rector del curso se aplicó literalmente: si el espectador necesita más de cinco segundos para entender el mensaje central, falló el argumento, no el espectador.

### 2.3 Atributos preatentivos y sistema semáforo

El color es el atributo preatentivo más poderoso; se detecta antes de que el ojo lo procese conscientemente. En el tablero se usó una paleta semáforo con significado consistente en todas las páginas: verde (#3B6D11) para lo positivo como alta conversión, oportunidad y segmento recurrente; rojo (#A32D2D) para la alerta como abandono de carrito, ganancias perdidas y categorías críticas; gris (#BBBBBB) para el contexto, es decir, categorías de referencia y valores neutros que no son el insight.

Este sistema permite al espectador identificar el dato crítico sin leer ninguna etiqueta. El color se reservó exclusivamente para el elemento más importante de cada gráfica y el resto va en gris, siguiendo la regla del curso: gris para el contexto, color para el insight.

### 2.4 Principios Gestalt

Se aplicaron principalmente dos principios. Por proximidad, los KPIs relacionados se agrupan en la misma área de cada página y las gráficas que responden la misma pregunta se disponen en la misma fila. Por similitud, el mismo tipo de gráfico y los mismos colores se usan para el mismo tipo de dato en todas las páginas, de modo que barras verdes siempre indican volumen positivo y barras rojas siempre indican pérdida.

### 2.5 Jerarquía visual y patrón Z

El patrón de lectura occidental va de arriba izquierda hacia abajo derecha. En cada página los datos más impactantes se ubican arriba y los detalles van bajando. Los títulos usan fuente Segoe UI negrita en color oscuro (#2D2D2D); las etiquetas de ejes y valores secundarios van en gris (#555555) para no competir con el dato principal.

### 2.6 Storytelling integrado y argumento visual

El curso propone el patrón obligatorio: contexto, hallazgo técnico, traducción de negocio, acción. Cada página del tablero cierra con una frase que sigue este patrón y convierte el análisis en una afirmación accionable. La ventana corrupta del 14 al 17 de noviembre se gestionó filtrando esos registros antes de construir las visualizaciones, de modo que no aparecen en ningún gráfico del tablero y no distorsionan los patrones mostrados.

### 2.7 Interactividad y revelación progresiva

El tablero incorpora filtros dinámicos de Top N que controlan simultáneamente múltiples gráficas mediante medidas sincronizadas, lo que permite demostrar filtrado en vivo durante la defensa. El panel de navegación lateral está visible en todas las páginas y permite ir directamente a cualquier sección sin perder el hilo de la presentación. Se evitó el tablero de control de avión: cada página tiene como máximo un control de filtro visible y el espacio principal se reserva para la narrativa visual.

---

## 3. Sistema de color y tipografía

Toda la identidad visual del tablero sigue una paleta fija aplicada de forma consistente en las siete páginas.

**Colores estructurales:** el gris oscuro #2D2D2D se usa en títulos de página, texto principal y el botón de navegación activo. El gris claro #DDDDDD es el header de todas las páginas. El blanco #FFFFFF es el fondo de tarjetas y gráficas. El gris muy claro #F8F8F8 es el fondo general de cada lienzo.

**Semáforo de datos:** verde #3B6D11 para oportunidades y métricas positivas; verde claro #97C459 para proyecciones o datos secundarios positivos; rojo #A32D2D para alertas y pérdidas; rojo claro #FCEBEB para fondos de tarjetas de alerta; gris #BBBBBB para datos de contexto que no son el insight.

**Tipografía:** Segoe UI en todo el tablero. Títulos de página en 20px negrita #2D2D2D. Títulos de gráficas en 14px negrita #2D2D2D. Etiquetas de ejes y subtítulos en 12px regular #555555. Frases de cierre en 13px negrita #2D2D2D.

---

## 4. Descripción de páginas

### 4.1 Inicio

**Pregunta que responde:** ¿de qué trata este análisis?

**Acto de habla:** motivar; crear contexto narrativo antes de mostrar los datos.

**Estructura**

La portada aplica un layout minimalista con el título del proyecto, la pregunta de investigación y los botones de navegación hacia cada sección. No incluye datos ni KPIs; su función es orientar al espectador y establecer el tono del análisis. La tipografía grande en negro sobre fondo gris sigue el principio de jerarquía visual del curso: lo primero que lee el espectador es el nombre del proyecto, lo segundo es la pregunta que va a responderse.

---

### 4.2 El problema

**Pregunta que responde:** ¿cuánto se pierde en el funnel de conversión?

**Acto de habla:** informar; establecer la magnitud del problema antes de profundizar en sus causas.

**Estructura**

La página se divide en dos columnas. A la izquierda, un gráfico de embudo muestra el recorrido desde vistas únicas (56,8 mill.) hacia carritos (2,2 mill.) y compras (1,3 mill.), con cada etapa en un tono progresivo hasta llegar al verde en las compras. A la derecha, cuatro tarjetas KPI con fondo rojo suave muestran los números del problema: carritos abandonados, tasa de abandono, tasa de conversión y ganancias perdidas.

El uso sistemático del rojo en todas las tarjetas aplica el principio de similitud de Gestalt: el espectador entiende de inmediato que todos esos números son una alerta. El embudo usa el atributo preatentivo de tamaño para hacer visible la caída de escala entre etapas, sin necesidad de texto explicativo.

---

### 4.3 Análisis por categoría

**Pregunta que responde:** ¿dónde se concentra la fuga de conversión por categoría de producto?

**Acto de habla:** convencer; demostrar que el problema no está distribuido uniformemente sino concentrado en un punto específico.

**Estructura**

La página organiza cuatro gráficas de barras horizontales en cuadrícula 2x2, más un filtro dinámico de Top N categorías con valores 5, 10 y 15. Las barras horizontales se eligieron sobre las verticales porque los nombres de las categorías son largos y se leen mejor en horizontal; además permiten alinear visualmente las cuatro gráficas para comparar la posición de electronics en todas ellas de un solo vistazo. El orden de las barras es descendente, siguiendo la recomendación del curso de reducir la fricción visual ordenando por valor.

Las cuatro gráficas son: valor promedio de compra por categoría en USD, ganancias por categoría en porcentaje, tasa de conversión por categoría en porcentaje y ganancias perdidas por categoría en USD. En las tres primeras, electronics aparece en verde oscuro como líder. En la cuarta aparece en rojo como la categoría que más dinero pierde. La tensión narrativa emerge de leer las cuatro gráficas en secuencia: electronics lidera en todo lo positivo pero también concentra la mayoría de las pérdidas. Esa paradoja es el insight central de la página.

---

### 4.4 Detalle Electronics

**Pregunta que responde:** dentro de electronics, ¿qué marcas concentran el abandono de carrito?

**Acto de habla:** convencer; localizar el problema al nivel de marca para hacer accionable la intervención.

**Estructura**

La página tiene dos KPIs en la parte superior derecha, valor promedio de compra y ganancias perdidas, que dan el contexto de escala sin saturar la página. Debajo, dos gráficas se complementan: a la izquierda, barras horizontales con el Top N de marcas por porcentaje de carritos abandonados, con Samsung y Apple en rojo dominando el ranking; a la derecha, un scatter plot que cruza ticket promedio en el eje Y, tasa de abandono en el eje X y tamaño de burbuja proporcional al volumen de carritos perdidos.

El scatter plot responde una pregunta que las barras no pueden responder: no basta con saber quién pierde más carritos en volumen, sino también cuánto vale cada carrito perdido. Apple aparece en la zona de alto ticket con abandono elevado; Samsung en ticket medio alto con mayor volumen absoluto de pérdida. La combinación de los dos gráficos da la imagen completa y aplica el principio del curso de usar el scatter de cuadrantes para identificar zonas de riesgo.

El filtro Top N de marcas con valores 5, 10 y 20 está sincronizado con las barras y el scatter, permitiendo demostrar interactividad en vivo durante la defensa.

---

### 4.5 Análisis de cliente

**Pregunta que responde:** ¿quién compra y cuánto vale?

**Acto de habla:** motivar; mostrar la oportunidad económica de intervenir en el segmento correcto.

**Estructura**

La página se divide en dos visuales. A la izquierda, un gráfico de donut muestra la tipología de visitante en tres segmentos: navegadores (85,0%), compradores con intención (9,3%) y compradores activos (5,6%). Los colores diferencian los tres grupos usando tonos de gris y verde oscuro para los compradores, siguiendo el sistema semáforo del tablero.

A la derecha, un diagrama Sankey muestra el flujo de compradores hacia revenue. El Sankey se eligió porque el curso lo identifica explícitamente como el gráfico adecuado para representar flujos entre etapas con ancho de banda proporcional a la cantidad. La historia del Sankey es que el 64,3% de los compradores son de única vez pero generan solo el 26,3% de las ganancias, mientras que el 35,7% recurrente genera el 73,7%. Los dos tonos de verde distinguen visualmente los dos segmentos: verde claro (#97C459) para única vez y verde oscuro (#3B6D11) para recurrente.

Los resultados del modelo de clustering (segmentos C0 a C3) se integrarán en esta página una vez estén disponibles, completando el insight central del proyecto: el cruce clasificador por clustering.

---

### 4.6 Contexto temporal

**Pregunta que responde:** ¿cuándo se convierte y qué nos dice el tiempo?

**Acto de habla:** informar; establecer que el problema de conversión es estructural y que el timing de la intervención importa.

**Estructura**

La página muestra un gráfico combo de doble eje con barras grises para el volumen de vistas y una línea verde para la tasa de conversión, con el eje X en formato de hora (00:00 a 23:00). La visualización se construyó sobre la tabla agregada por hora del día, filtrando la ventana corrupta, lo que garantiza que los patrones mostrados corresponden al comportamiento real de los usuarios.

El insight más poderoso de esta página emerge de la distribución por hora: el tráfico aumenta entre las 14h y las 17h pero la conversión es más alta entre las 5h y las 11h. Las barras altas de la tarde no coinciden con la línea verde de la mañana; esa disociación entre volumen y conversión es el argumento visual que justifica intervenir en el momento correcto y no solo en el segmento correcto.

---

### 4.7 Cierre

**Pregunta que responde:** ¿cuál es la respuesta a la pregunta de oro?

**Acto de habla:** motivar; cerrar la narrativa con la respuesta directa y accionable.

**Estructura**

La página aplica un layout de dos columnas. A la izquierda, la pregunta de investigación del proyecto en texto grande. A la derecha, tres tarjetas con las respuestas en palabras clave: Electronics (tarjeta roja, señala el problema), Comprador único convertible en recurrente (tarjeta verde, señala el segmento objetivo) y Franja 5:00 a 11:00h (tarjeta verde, señala el momento de intervención).

El diseño aplica el principio de jerarquía visual del curso: el espectador lee la pregunta a la izquierda y los ojos van naturalmente hacia las respuestas a la derecha. En cinco segundos el evaluador tiene la síntesis completa del proyecto sin necesidad de leer un párrafo. El uso del rojo para Electronics y el verde para las dos respuestas de acción refuerza el sistema semáforo del tablero: rojo señala dónde está el problema, verde señala qué hacer.

---

## 5. Interactividad

El tablero incorpora los siguientes elementos interactivos demostrados en vivo durante la defensa.

El filtro Top N de categorías con valores 5, 10 y 15 controla simultáneamente las cuatro gráficas de la página de análisis por categoría mediante medidas sincronizadas. El filtro Top N de marcas con valores 5, 10 y 20 controla las barras y el scatter de la página de Electronics. El panel de navegación lateral está presente en todas las páginas y permite saltar directamente a cualquier sección sin usar las pestañas nativas de Power BI.

La interactividad en vivo es uno de los tres pilares de evaluación del curso (exploración, mensaje, producto). El diseño del tablero privilegió la demostración de filtrado en vivo y navegación controlada sobre la cantidad de visuales estáticos.

---

## 6. Herramientas y conexión a datos

La herramienta principal es Power BI Desktop con publicación en Power BI Service mediante cuenta institucional EAFIT. La fuente de datos son los CSVs generados por el pipeline de Databricks sobre la capa Gold del proyecto, versionados en Git en la carpeta `reports/data/` del repositorio. Power BI los consume desde la ruta local del repositorio clonado.

El modelo de datos incluye una tabla de dimensión `dim_categoria` que actúa como puente entre las tablas de funnel por categoría y revenue en juego, permitiendo que el filtro Top N controle ambas con una sola selección. Las demás tablas son independientes entre sí y se conectan directamente a sus visuales correspondientes.
