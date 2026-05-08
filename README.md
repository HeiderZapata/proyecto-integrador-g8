# proyecto-integrador-g8

# Proyecto Integrador — Grupo 8
**Optimización de la conversión en E-commerce mediante Modelado de Propensión de Compra**  
Maestría en Ciencia de Datos y Analítica — Universidad EAFIT

---

## Guía de configuración para integrantes del equipo

Sigue estos pasos **en orden** para quedar sincronizado con el repositorio y poder trabajar colaborativamente desde Databricks.

---

### Prerrequisitos

- Tener cuenta en [GitHub](https://github.com) y haber aceptado la invitación al repositorio
- Tener cuenta en [Databricks Free Edition](https://www.databricks.com/try-databricks)
- Haber descargado los archivos del dataset desde [Kaggle](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store/data)

---

### Paso 1 — Aceptar la invitación al repositorio de GitHub

1. Revisa tu correo electrónico (el asociado a tu cuenta de GitHub)
2. Busca el correo de invitación de GitHub con el asunto:  
   *"You've been invited to collaborate"*
3. Haz clic en **View invitation** y luego en **Accept invitation**
4. Confirma que puedes ver el repositorio en [github.com](https://github.com)

---

### Paso 2 — Crear cuenta en Databricks Free Edition

> Si ya tienes cuenta, salta al Paso 3.

1. Ve a [databricks.com/try-databricks](https://www.databricks.com/try-databricks)
2. Selecciona **Get started for free** → **Free Edition**
3. Regístrate con tu correo de la universidad EAFIT
4. Confirma tu correo y accede al workspace

---

### Paso 3 — Generar un token de GitHub (PAT)

Necesitas un token para que Databricks pueda leer y escribir en el repositorio.

1. Entra a [github.com](https://github.com) con tu cuenta
2. Clic en tu avatar (arriba a la derecha) → **Settings**
3. Scroll hasta abajo → **Developer settings**
4. **Personal access tokens → Tokens (classic)**
5. Clic en **Generate new token (classic)**
6. Configúralo así:

   | Campo | Valor |
   |---|---|
   | **Note** | `databricks-proyecto-g8` |
   | **Expiration** | `90 days` |
   | **Scopes** | ✅ Marca solo `repo` |

7. Clic en **Generate token**
8. **⚠️ Copia el token inmediatamente** — solo se muestra una vez

---

### Paso 4 — Vincular GitHub con tu Databricks

1. Entra a tu workspace de Databricks
2. Haz clic en tu nombre de usuario (barra superior derecha) → **Settings**
3. Ve a **Linked accounts**
4. Clic en **Add Git credential**
5. Completa los campos:

   | Campo | Valor |
   |---|---|
   | **Git provider** | `GitHub` |
   | **GitHub username** | Tu usuario de GitHub |
   | **Token** | Pega el token del Paso 3 |

6. Clic en **Save** ✅

---

### Paso 5 — Clonar el repositorio en tu Workspace de Databricks

1. En el menú izquierdo de Databricks, haz clic en **Workspace**
2. Clic en **+ New → Git folder**
3. En el campo **Git repository URL**, pega:

   ```
   https://github.com/HeiderZapata/proyecto-integrador-g8
   ```

4. El nombre de la carpeta se completa automáticamente
5. Clic en **Create Git folder** 

Ahora verás la carpeta `proyecto-integrador-g8` en tu Workspace con todos los notebooks del proyecto sincronizados.

---

### Paso 6 — Subir el dataset a tu Volume

Cada integrante debe tener los datos en **su propio** Volume, ya que los Volumes no se comparten entre cuentas en la Free Edition.

1. En el menú izquierdo ve a **Catalog**
2. Expande **workspace → default**
3. Clic derecho en **default → Create → Volume**
4. Nómbralo exactamente: `ecommerce_raw`
5. Abre un nuevo notebook en tu workspace y ejecuta estas celdas para descargar el dataset:

```python
# Celda 1 — Instalar Kaggle
%pip install kaggle
```

```python
# Celda 2 — Configurar credenciales de Kaggle
# (obtén tu API key en kaggle.com → Settings → API → Create New Token)
import os
os.environ['KAGGLE_USERNAME'] = 'tu_usuario_kaggle'
os.environ['KAGGLE_KEY']      = 'tu_api_key_kaggle'
```

```python
# Celda 3 — Descargar los archivos directo al Volume
import subprocess
subprocess.run([
    'kaggle', 'datasets', 'download',
    '-d', 'mkechinov/ecommerce-behavior-data-from-multi-category-store',
    '--unzip',
    '-p', '/Volumes/workspace/default/ecommerce_raw'
], check=True)
print("✅ Descarga completa")
```

```python
# Celda 4 — Verificar que los archivos quedaron bien
for f in os.listdir('/Volumes/workspace/default/ecommerce_raw'):
    size = os.path.getsize(f'/Volumes/workspace/default/ecommerce_raw/{f}')
    print(f"📄 {f}  →  {size/1e9:.2f} GB")
```

Deberías ver:
```
📄 2019-Oct.csv  →  5.52 GB
📄 2019-Nov.csv  →  9.00 GB
```

---

### Paso 7 — Verificar que todo funciona

Abre el notebook `notebooks/01_EDA_Ecommerce.ipynb` desde tu carpeta Git en Databricks y ejecuta la primera celda. Si corre sin errores, estás listo para colaborar. ✅

---

##  Flujo de trabajo diario

Una vez configurado, el flujo para trabajar en equipo es el siguiente:

```
Antes de empezar a trabajar:
  → Abrir el notebook en Databricks
  → Clic en el ícono de Git (rama) → Pull
  → Esto descarga los últimos cambios del equipo

Al terminar tu sesión de trabajo:
  → Clic en el ícono de Git → Commit & Push
  → Escribe un mensaje descriptivo, por ejemplo:
    "feat: agrego análisis de precios por categoría"
  → Clic en Commit & Push
```

>  **Buena práctica:** haz Pull siempre antes de empezar y Push siempre al terminar. Esto evita conflictos entre los cambios del equipo.

---

##  Estructura del repositorio

```
proyecto-integrador-g8/
├── README.md                          ← Este archivo
├── notebooks/
│   ├── 01_EDA_Ecommerce.ipynb         ← Análisis exploratorio (Semana 1)
│   ├── 02_Feature_Engineering.ipynb   ← Preparación de datos (Semana 2)
│   ├── 03_Modeling.ipynb              ← Modelos ML (Semana 3)
│   └── 04_Dashboard.ipynb             ← Visualizaciones (Semana 4)
└── docs/
    └── arquitectura.md                ← Decisiones técnicas del proyecto
```

---

##  Integrantes

| Nombre | Usuario GitHub | Rol |
|---|---|---|
| Sara | [@usuario] | Visualización |
| Kelly | [@usuario] | Data Architect |
| Yeison | [@usuario] | ML Engineer |
| Heider | [@usuario] | Data Analyst / 


---

##  Dataset

**Fuente:** REES46 Marketing Platform  
**Referencia:** Kechinov, M. (2020). *eCommerce behavior data from multi category store*. Kaggle.  
**Licencia:** CC BY 4.0  
**Volumen:** ~14.5 GB (Oct 2019: 5.5 GB + Nov 2019: 9 GB)
