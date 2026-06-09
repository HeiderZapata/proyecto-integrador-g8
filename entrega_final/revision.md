# Revisión de entrega final — PI Grupo 8

**Última actualización:** 2026-06-09
**Alcance:** `Informe_Final_PI_Grupo8_plantilla.docx` (consolidado) y `Arquitectura_Utilizada.jpeg`

## Estado general
- 🖼️ **Imagen (`Arquitectura_Utilizada.jpeg`): LISTA ✅** — sin inconsistencias factuales (ver §A).
- 📄 **Informe: CONSOLIDADO** en un solo `.docx` = base de **Heider** (índice/TOC + §2.6 y §4 ampliadas) **+** las correcciones revisadas. Aplicadas y verificadas (ver §B). *Pendiente manual: cerrar Word para el swap final del archivo + refrescar el TOC (F9).*

---

## A. Diagrama de arquitectura — LISTO ✅

La versión regenerada corrigió **los 4 puntos** que se habían marcado:
- Train → **`oct + nov ≤ 23 (excl. cuarentena 14–17 nov)`** (antes "≤ 13", error factual).
- Añadido **`Trigger.AvailableNow`** en Procesamiento.
- Añadidos **baselines Dummy → Logística** en Modelos de árbol.
- Añadido el paso **`12 CSV · reports/data/`** entre Gold Agregada y Power BI.

**Sin inconsistencias factuales.** Quedan solo 3 detalles **cosméticos opcionales** (no bloquean):
1. La caja "Split Temporal visible" se lee cortada ("plit…") por solape con la etiqueta "Entrenamiento".
2. La flecha punteada hacia "Gold Agregada" nace a la altura de Bronze (se deriva de Silver/Gold).
3. Texto "matriz sesión & agregados" dentro de Gold·Delta se solapa con la caja aparte "Gold Agregada".

---

## B. Informe — correcciones aplicadas

### B.1 Hallazgos de la revisión inicial (APLICADOS)
| Sección | Corrección |
|---|---|
| §1 y §3.2.1 | "109.5 millones de eventos" → **"109.95 millones"** (= 42.45M oct + 67.50M nov) |
| §2 título | "Marco teórico ~~y referencias~~" (Heider ya corrigió el encabezado; se ajustó la **entrada del TOC**) |
| §3.3.1 | "~~12~~ → **13** variables conductuales" (las 17 features son correctas; el desglose era 13+dow+finde+hora) |
| §3.3.4 | Umbral **cuantificado**: F1-óptimo ≈ 0.092, ~15 % de sesiones, precisión ~12 %, ≈2.2× la base 5.6 % |
| §3.3.4 | Aclarado que el clustering corre sobre **muestra estratificada de 300k** (resuelve el aparente 0.04 % vs 106) |
| §3.3.4 (C2) | Gramática: "106 sesiones ~~de días~~" → "106 sesiones **rotas (~14.6 días de duración)**" |
| §4 título | De-dup: "ingeniería de datos y ~~uso de tecnología~~ **arquitectura**" |

### B.2 Hallazgos sobre lo que editó Heider (APLICADOS)
| # | Problema introducido por Heider | Corrección |
|---|---|---|
| 🔴 | **§4.2 decía "69 millones de filas crudas"** vs §5.4 "23 M" (mismo concepto, 2 cifras: 69M=unidades, 23M=sesiones) | Alineado a **"los 23 millones de filas crudas"** (consistente con §5.4 y con las 22.99M sesiones) |
| 🔴 | **§2.6 (Lakehouse/Kappa): párrafos de cuerpo con estilo Heading 2** → se renderizaban como títulos y contaminarían el TOC | Re-estilados a **cuerpo/Normal** (3 párrafos) |
| 🟡 | **TOC desactualizado** en la entrada de §2 ("…y referencias", el encabezado ya no lo dice) | Entrada del TOC corregida (conviene además refrescar con F9) |
| 🟡 | Typo doble punto "código.." (§4.1) | Corregido a un punto |
| 🟡 | Punto final faltante (§4.2, "…de la visualización") | Añadido |

**Lo bueno de Heider (conservado):** generó el **índice/TOC**, amplió §2.6 (Lakehouse + estilo Kappa) y §4.1/§4.2 (ingesta, particionamiento por tamaño, ELT, Volume vs S3) — todo factualmente correcto.

### B.3 Hallazgos que CAMBIARON al verificar los notebooks fuente
- **"17 features" era CORRECTO** (el notebook lista 17); el error real era el desglose → se corrigió "12 → 13" conductuales.
- **"106 sesiones vs 0.04 %" NO era contradicción**: el clustering corre sobre **muestra de 300k** (106/300k ≈ 0.035 %). Se aclaró la muestra y se arregló la gramática.

---

## C. Pendientes (requieren tu acción / se hacen en Word)
1. **Cerrar Word** para que se complete el swap del archivo consolidado (estaba bloqueado).
2. **Refrescar el TOC** en Word (clic en la tabla → F9 → "Actualizar toda la tabla") para corregir páginas y reflejar el título de §2 y el des-titulado de §2.6.
3. **Placeholders sin rellenar:** apellidos `[apellido]` (Kelly, Sara, Yeison), los `[ POR COMPLETAR ]` (enlace del tablero, v2 Kelly) y las **referencias en APA** (§7).
4. **Opcional:** embeber el diagrama (`Arquitectura_Utilizada.jpeg`) en §4; los 3 cosméticos del diagrama (§A).

> **Formato de números (revisado):** NO es inconsistencia — el documento usa convención anglosajona de forma consistente (coma=miles, punto=decimal). No se tocó.
