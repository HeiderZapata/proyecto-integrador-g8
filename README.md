# Proyecto Integrador — Grupo 8
**Optimización de la conversión en E-commerce mediante Modelado de Propensión de Compra**
Maestría en Ciencia de Datos y Analítica — Universidad EAFIT

> **Qué es este archivo:** la **guía de puesta en marcha** del entorno (cómo configurarte y correr el proyecto).
> - Para *qué hace* el proyecto y las decisiones → `docs/00_estado_del_proyecto.md` (documento maestro).
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
1. En Databricks: tu usuario → **Settings → Linked accounts**.
2. **Add Git credential** → **Git provider:** GitHub · **Username:** tu usuario · **Token:** el del Paso 3 → **Save**.

### Paso 5 — Clonar el repositorio como Git folder
> *Pasos PROVISIONALES (forma nueva de Databricks; ya no se usa la carpeta "Repos"). Yeison los valida al configurarse y se finalizan después.*
1. Requisito: haber vinculado tu token de GitHub (Paso 4) — el repo es privado.
2. Menú izquierdo → **Workspace** → entra a tu **carpeta personal**: **Users → tu-correo**.
3. Botón **Create** (arriba a la derecha) → **Git folder**. *(No uses "Repo": ese es el modo viejo.)*
4. **Git repository URL:** `https://github.com/HeiderZapata/proyecto-integrador-g8` → **Create Git folder**.
5. **Importante:** antes de trabajar, en el diálogo de Git **crea/cámbiate a TU rama de frente** (`feat/pipeline`, `feat/modelo`, `feat/viz`, `feat/ab-test`, `feat/doc`). **Nunca trabajes sobre `main`.** El detalle está en `CONTRIBUTING.md`.

### Paso 6 — Tus credenciales de Kaggle (cada quien la suya — método simple)
> *Pasos PROVISIONALES: Yeison los valida y se finalizan después.*
**Ninguna llave se commitea, nunca.** Cada integrante usa la suya.

1. **Genera tu credencial *Legacy*:** kaggle.com → avatar → **Settings** → sección **API**. Hay dos tipos de credencial; usa **"Create Legacy API Key"** (bajo *Legacy API Credentials*), **NO** el token nuevo `KGAT_`. Descarga el `kaggle.json` (trae `username` y `key`). Si ya generaste un `KGAT_`, expíralo.
2. **Crea tu Volume:** Catalog → workspace → default → **Create → Volume**, nómbralo `ecommerce_raw`.
3. **Sube tu `kaggle.json` al Volume:** dentro del Volume, **Upload to this volume** → selecciona el archivo. Queda en `/Volumes/workspace/default/ecommerce_raw/kaggle.json`.
4. El notebook `notebooks/pipeline/01_sube_datos_kaggle_Databricks.ipynb` **lee la llave de ese archivo** (sin ningún valor escrito en el código). Esta celda corre **antes** de la descarga:

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
Ejecuta la primera celda de `notebooks/pipeline/01_sube_datos_kaggle_Databricks.ipynb`. Si corre sin errores, estás listo.

---

## Flujo de trabajo diario (resumen — el detalle está en `CONTRIBUTING.md`)

Trabajamos con **una rama por frente** (no por persona) y `main` se actualiza **solo cuando Yeison fusiona un PR**. Desde Databricks:

```
Antes de trabajar:
  → Abre el ícono de Git del Git folder
  → Asegúrate de estar en TU rama (feat/...), no en main
  → Pull  (trae lo último)

Al terminar una pieza estable:
  → Commit & Push  → SUBE A TU RAMA, no a main
  → Mensaje claro, p. ej. "viz: funnel por categoría v1"

Para integrar a main:
  → Abre un Pull Request en GitHub (tu rama → main)
  → Sincroniza tu rama con main y deja el PR "en verde" (sin conflictos)
  → Avisa a Yeison: él hace el merge a main
  → Los demás hacen Pull de main
```

> **Regla de oro:** nunca trabajes ni hagas push directo a `main`. Todo entra por PR; el merge lo centraliza Yeison (suplente: Heider). Conflictos, recetas de rescate y `.gitignore`: ver `CONTRIBUTING.md`.

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

Detalle y el principio **exploración ≠ producción**: `docs/00_estado_del_proyecto.md` §12.

> **Nota (jun):** el notebook `02_Medallion_Y_EDA_Ecommerce` hoy combina Medallion + EDA. Está provisionalmente en `notebooks/pipeline/`; se separará en Fase 4 (Medallion → `pipeline/`, funnel → `analysis/`).

---

## Integrantes y frentes

| Nombre | Usuario GitHub | Frente |
|---|---|---|
| Heider | [@usuario] | Ingeniería de Datos (pipeline, Medallion, repo) |
| Sara | [@usuario] | ML / Modelado (features, entrenamiento, clustering) |
| Kelly | [@usuario] | Visualización + narrativa (EDA-funnel, tablero) |
| Yeison | [@usuario] | Integración + Gold (con Heider) + A/B + documento |

*(Reparto propuesto; se confirma en la reunión que cierra Fase 3 — doc 00 §11.)*

---

## Dataset
**Fuente:** REES46 Marketing Platform · **Referencia:** Kechinov, M. (2020). *eCommerce behavior data from multi category store*. Kaggle. · **Licencia:** CC BY 4.0 · **Volumen:** ~14.5 GB (Oct 2019: 5.5 GB + Nov 2019: 9 GB).
