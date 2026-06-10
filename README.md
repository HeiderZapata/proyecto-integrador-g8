# Proyecto Integrador — Grupo 8
**Optimización de la conversión en E-commerce mediante Modelado de Propensión de Compra**
Maestría en Ciencia de Datos y Analítica — Universidad EAFIT

> **Qué es este archivo:** la **guía de puesta en marcha** del entorno (cómo configurarte y correr el proyecto).
> - Para *qué hace* el proyecto y las decisiones → el **informe final** en `entrega_final/`.
> - Para *cómo colaborar en Git* (ramas, PRs, merges) → `CONTRIBUTING.md` (en la raíz).

---

## Guía de configuración para integrantes del equipo

Sigue estos pasos **en orden** para quedar sincronizado y poder trabajar desde Databricks.

### Prerrequisitos
- Cuenta en [GitHub](https://github.com) y haber aceptado la invitación al repositorio.
- Cuenta en [Databricks Free Edition](https://www.databricks.com/try-databricks).
- Cuenta en [Kaggle](https://www.kaggle.com). **Cada integrante usa su propia API key** (ver Paso 6).

---

### Paso 1 — Aceptar la invitación al repositorio de GitHub
1. Revisa el correo asociado a tu cuenta de GitHub.
2. Busca el correo con asunto *"You've been invited to collaborate"*.
3. Clic en **View invitation** → **Accept invitation**.
4. Confirma que ves el repositorio en [github.com](https://github.com).

### Paso 2 — Crear cuenta en Databricks Free Edition
> Si ya tienes cuenta, salta al Paso 3.
1. Ve a [databricks.com/try-databricks](https://www.databricks.com/try-databricks).
2. **Get started for free** → **Free Edition**.
3. Regístrate con tu correo y accede al workspace.

### Paso 3 — Generar un token de GitHub (PAT)
Databricks lo necesita para leer/escribir en el repo.
1. En GitHub: avatar → **Settings** → **Developer settings**.
2. **Personal access tokens → Tokens (classic)** → **Generate new token (classic)**.
3. Configúralo: **Note** `databricks-proyecto-g8` · **Expiration** `90 days` · **Scopes** solo `repo`.
4. **Generate token** y **cópialo de inmediato** (solo se muestra una vez).
5. **No lo pegues en ningún archivo del repo.** Es un secreto; solo va en el campo de Databricks (Paso 4).

### Paso 4 — Vincular GitHub con tu Databricks
1. En Databricks: tu nombre (arriba a la derecha) → **Settings** → bajo **User**, **Linked accounts**.
2. **Add Git credential** y completa:
   - **Git provider:** GitHub
   - **Git provider username:** tu usuario de GitHub
   - **Token / App password:** el PAT de GitHub del Paso 3 (no un token de Databricks).
   - **Git credential nickname:** una etiqueta interna tuya, única, p. ej. `github-proyecto-g8`. Es solo para tu referencia; no afecta la conexión.
3. **Save**. Debe quedar marcado como *GitHub (Linked)*.

### Paso 5 — Clonar el repositorio como Git folder
> *Forma nueva de Databricks: ya no se usa la carpeta "Repos".*
1. Requisito: haber vinculado tu credencial de GitHub (Paso 4) — el repo es privado.
2. Menú izquierdo → **Workspace** → entra a tu **carpeta personal**: **Users → tu-correo**.
3. Botón **Create** (arriba a la derecha) → **Git folder**. *(No uses "Repo": ese es el modo viejo.)*
4. **Git repository URL:** `https://github.com/HeiderZapata/proyecto-integrador-g8`.
5. **Sparse checkout mode:** déjalo **DESACTIVADO** (queremos todo el repo; es pequeño).
6. **Create Git folder.** Verás la carpeta `proyecto-integrador-g8` con `docs/`, `notebooks/` (con `pipeline/`, etc.) y `reports/`.
7. **Importante:** antes de trabajar, en el diálogo de Git **crea/cámbiate a TU rama de frente** (`feat/pipeline`, `feat/modelo`, `feat/viz`, `feat/ab-test`, `feat/doc`). **Nunca trabajes sobre `main`.** El detalle está en `CONTRIBUTING.md`.

### Paso 6 — Tus credenciales de Kaggle (cada quien la suya — método simple)
**Ninguna llave se commitea, nunca.** Cada integrante usa la suya.

1. **Genera tu credencial *Legacy*:** kaggle.com → avatar → **Settings** → sección **API**. Hay dos tipos de credencial; usa **"Create Legacy API Key"** (bajo *Legacy API Credentials*), **NO** el token nuevo `KGAT_`. Descarga el `kaggle.json` (trae `username` y `key`). Si ya generaste un `KGAT_`, expíralo.
2. **Crea tu Volume:** Catalog → workspace → default → **Create → Volume**. **Volume type: Managed.** Nómbralo `ecommerce_raw` (en minúsculas, igual que en el código).
3. **Sube tu `kaggle.json` al Volume:** dentro del Volume, **Upload to this volume** → selecciona el archivo. Queda en `/Volumes/workspace/default/ecommerce_raw/kaggle.json`.
4. El notebook `notebooks/pipeline/01_ingesta_kaggle.ipynb` **lee la llave de ese archivo** (sin ningún valor escrito en el código). Esta celda corre **antes** de la descarga:

```python
%pip install kaggle
```
```python
import json, os
with open('/Volumes/workspace/default/ecommerce_raw/kaggle.json') as f:
    creds = json.load(f)
os.environ['KAGGLE_USERNAME'] = creds['username']
os.environ['KAGGLE_KEY']      = creds['key']
```
```python
import subprocess
subprocess.run([
    'kaggle','datasets','download',
    '-d','mkechinov/ecommerce-behavior-data-from-multi-category-store',
    '--unzip','-p','/Volumes/workspace/default/ecommerce_raw'
], check=True)
```
```python
for f in os.listdir('/Volumes/workspace/default/ecommerce_raw'):
    size = os.path.getsize(f'/Volumes/workspace/default/ecommerce_raw/{f}')
    print(f"{f}  ->  {size/1e9:.2f} GB")
# Esperado: 2019-Oct.csv ~5.52 GB · 2019-Nov.csv ~9.00 GB
```

> **Por qué así (lección aprendida):** una llave escrita dentro de un notebook queda en el historial de Git y se asume comprometida. Por eso **ninguna credencial va en el código**: cada quien sube su `kaggle.json` a su Volume (no al repo) y el notebook lo lee de ahí. En Databricks serverless **no** sirve el comando `~/.kaggle/...` que sugiere Kaggle (ese `~` es efímero). Usamos la llave *Legacy* porque el notebook trabaja con `username`+`key`. El `.gitignore` bloquea `kaggle.json` por si acaso.

### Paso 7 — Verificar que todo funciona
Ejecuta la primera celda de `notebooks/pipeline/01_ingesta_kaggle.ipynb`. Si corre sin errores, estás listo.

---

## Flujo de trabajo diario — cómo guardar y subir tus cambios

> **⚠️ Lo más importante (léelo): ejecutar un notebook NO actualiza GitHub.** Correr las celdas solo genera resultados en *tu* Databricks. Para que tu código llegue a GitHub tienes que hacer **Commit & Push a mano** (pasos abajo), y eso sube **solo a tu rama** — `main` cambia únicamente cuando Yeison fusiona tu PR. Los **datos** del Volume (los CSV) **nunca** van a GitHub.

Trabajamos con **una rama por frente** (no por persona). Todo esto se hace desde el **diálogo de Git** del Git folder en Databricks (el ícono de Git / el nombre de la rama que aparece arriba del notebook).

### A) Una sola vez — crea tu rama de frente
1. Abre el **diálogo de Git** del Git folder `proyecto-integrador-g8`.
2. En el selector de rama (arriba), elige **Create Branch**.
3. Nómbrala según tu frente: `feat/pipeline`, `feat/modelo`, `feat/viz`, `feat/ab-test` o `feat/doc`. Créala **a partir de `main`**.
4. Quedas parado en tu rama. **Nunca trabajes sobre `main`.**

### B) Cada vez que vas a trabajar — trae lo último
1. Abre el diálogo de Git y confirma que estás en **TU rama** (no en `main`).
2. Clic en **Pull** → baja los cambios que ya estén en `main`/tu rama.
3. Trabaja en tu notebook, dentro de tu carpeta (`notebooks/...`).

### C) Al terminar una pieza estable — Commit & Push (esto SÍ sube a GitHub)
1. Abre el diálogo de Git. Verás la lista de **archivos cambiados**.
2. Escribe un **mensaje de commit** claro, p. ej. `viz: funnel por categoría v1`.
3. Clic en **Commit & Push**.
4. Esto sube tus cambios a **tu rama** en GitHub (no a `main`). Si es la primera vez en esa rama, Databricks la crea en GitHub.
5. *Si te sale un conflicto:* normalmente es porque `main` cambió; haz **Pull** primero, resuelve y vuelve a **Commit & Push**. (Detalle en `CONTRIBUTING.md` §7.)

### D) Para integrar tu trabajo a `main` — el Pull Request
1. Entra a GitHub: `https://github.com/HeiderZapata/proyecto-integrador-g8`.
2. Abre un **Pull Request**: tu rama → `main`.
3. Asegúrate de que el PR quede **"Able to merge" / en verde** (sin conflictos); si no, sincroniza tu rama con `main` (Pull) y vuelve a empujar.
4. **Avisa a Yeison** en el chat: él hace el **merge** a `main` (suplente: Heider). **Este es el único momento en que `main` cambia.**
5. Cuando Yeison fusione, los demás hacen **Pull** de `main` para recibir tu cambio.

> **Regla de oro:** nunca trabajes ni hagas push directo a `main`. Todo entra por PR; el merge lo centraliza Yeison (suplente: Heider). Si editas y **no** haces Commit & Push, tu trabajo se queda solo en tu Databricks y nadie lo ve (ni queda respaldado). Conflictos, recetas de rescate y `.gitignore`: ver `CONTRIBUTING.md`.

---

## Estructura del repositorio

```
proyecto-integrador-g8/
├── README.md                ← este archivo (puesta en marcha)
├── CONTRIBUTING.md          ← cómo colaborar en Git
├── .gitignore
├── docs/                    ← documentación (00 maestro, 02 arquitectura, packs, propuesta...)
├── notebooks/
│   ├── exploration/         ← EDA de descubrimiento (informa el diseño)
│   ├── pipeline/            ← ingesta + Medallion (Bronze/Silver/Gold)
│   ├── modeling/            ← features, entrenamiento, evaluación, clustering
│   └── analysis/            ← EDA-funnel entregable + clustering exploratorio
└── reports/
    ├── powerbi/             ← tablero
    └── data/                ← Gold agregada (pequeña) para Power BI
```

El principio detrás de la estructura: **exploración ≠ producción** (el pipeline reproducible vive separado del análisis exploratorio).

> **Nota:** el pipeline está separado del análisis (principio *exploración ≠ producción*): `02_medallion.ipynb` (Bronze/Silver/Gold) y `03_gold_agregada_bi.ipynb` viven en `pipeline/`; el EDA entregable (`eda_ecommerce.ipynb`) en `analysis/`. La separación se completó el 4-jun.

---

## Integrantes y frentes

| Nombre | Usuario GitHub | Frente |
|---|---|---|
| Heider | [@usuario] | Ingeniería de Datos (pipeline, Medallion, repo) |
| Sara | [@usuario] | ML / Modelado (features, entrenamiento, clustering) |
| Kelly | [@usuario] | Visualización + narrativa (EDA-funnel, tablero) |
| Yeison | [@usuario] | Integración + Gold (con Heider) + A/B + documento |

*(Reparto **firme** — aceptado en la reunión del 3-jun.)*

---

## Dataset
**Fuente:** REES46 Marketing Platform · **Referencia:** Kechinov, M. (2020). *eCommerce behavior data from multi category store*. Kaggle. · **Licencia:** CC BY 4.0 · **Volumen:** ~14.5 GB (Oct 2019: 5.5 GB + Nov 2019: 9 GB).
