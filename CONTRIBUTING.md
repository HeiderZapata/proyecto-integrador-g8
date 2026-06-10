# Cómo trabajamos en GitHub — Guía del equipo

> **Para quién es esto:** para los cuatro (Yeison, Sara, Heider, Kelly). Asumimos que **nadie es experto en Git**. Esta guía explica el *porqué* y el *paso a paso*. Si algo no se entiende, pregúntalo en el chat del equipo antes de improvisar — un minuto de pregunta evita una hora de enredo.
>
> **La regla de oro (si solo recuerdas una cosa):** nunca trabajes directo sobre `main`. Siempre en tu rama, y lo que terminas entra a `main` por un *Pull Request*.

---

## 0. La idea en una frase (el modelo mental)

Imagina que `main` es **el documento final del proyecto, la versión buena que siempre debe funcionar**. Nadie escribe directo sobre el documento final mientras otros lo usan: harías un desastre.

En vez de eso:
- **Te haces una copia de trabajo** (una *rama*) donde experimentas sin miedo a romper nada.
- Cuando tu pedazo está listo, **pides meterlo al documento final** (un *Pull Request*, o *PR*), y otra persona le da una mirada rápida.
- Una vez aprobado, **se fusiona a `main`** y todos lo reciben.

Eso es todo. El resto son detalles de cómo hacerlo sin pisarnos.

**Dos momentos que se confunden — `push` NO es actualizar `main`:**
- `git push` sube tu trabajo **solo a tu rama** en GitHub. `main` todavía no se entera.
- El **merge del PR** es el único momento en que tu trabajo entra a `main`.

```
git push        →   tu trabajo llega a TU rama (feat/...)    [main sigue igual]
merge del PR    →   tu trabajo entra a MAIN                  [ahora sí main cambió]
```

En este equipo, **ese merge a `main` lo hace una sola persona: Yeison** (suplente: Heider). Tú trabajas y haces `push` a tu rama; Yeison es quien abre la puerta a `main`. Ver §5 y §6.

**El problema que estamos evitando:** que dos personas editen lo mismo a la vez y al juntarlo Git no sepa cuál versión vale (un *conflicto*), o que alguien suba algo roto a `main` y bloquee al resto. Con un deadline de 5 días, eso es lo último que queremos.

---

## 1. Glosario mínimo (una línea cada uno)

- **Repositorio (repo):** la carpeta del proyecto, con todo su historial. La nuestra: `HeiderZapata/proyecto-integrador-g8`.
- **Clonar (`clone`):** bajar el repo a tu computador por primera vez.
- **Rama (`branch`):** una línea de trabajo aislada. Tu espacio para experimentar sin tocar `main`.
- **Commit:** una "foto" guardada de tus cambios, con un mensaje que dice qué hiciste.
- **Push:** subir tus commits al repo en GitHub (la nube).
- **Pull:** bajar a tu computador los cambios que otros subieron.
- **Pull Request (PR):** la solicitud de fusionar tu rama a `main`. Es donde se revisa antes de mezclar.
- **Merge (fusionar):** juntar dos ramas. Normalmente, tu rama → `main`.
- **Conflicto:** cuando dos personas cambiaron las mismas líneas y Git pregunta cuál se queda. Se resuelve a mano (ver §7); no es grave.

---

## 2. Preparación (una sola vez por persona)

**a) Instalar Git.** Descárgalo de `https://git-scm.com/downloads`.
*Alternativa para novatos:* **GitHub Desktop** (`https://desktop.github.com`) es una app con botones en vez de comandos; hace lo mismo que esta guía pero con clics. Si prefieres botones, úsala — los conceptos (rama, commit, push, pull, PR) son idénticos.

**b) Decirle a Git quién eres** (para que tus commits lleven tu nombre):
```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu-correo@dominio.com"
```

**c) Clonar el repo** (bajarlo por primera vez):
```bash
git clone https://github.com/HeiderZapata/proyecto-integrador-g8.git
cd proyecto-integrador-g8
```

**d) Autenticación.** La primera vez que hagas `push`, GitHub te pedirá identificarte. **Ojo:** GitHub ya **no acepta la contraseña** de tu cuenta para esto; necesitas un **token de acceso personal (PAT)** o una **llave SSH**. Crea uno siguiendo la guía oficial de GitHub (`https://docs.github.com/authentication`). Dos reglas:
- **Nunca** pegues el token dentro de un archivo del proyecto ni en un commit. Es un secreto.
- Si usas GitHub Desktop, la autenticación es por la app y te ahorras esto.

---

## 3. Las ramas del proyecto y quién trabaja dónde

Trabajamos con **una rama por frente**, no por persona. ¿Por qué? Porque la tabla Gold tiene que llegar a `main` el jueves para que Sara y Kelly la usen, sin esperar al A/B test ni al documento; y porque dos personas (Heider y Yeison) tocan el pipeline. Ramas por frente lo manejan limpio. En la práctica, como casi cada frente tiene un dueño fijo, tu rama se sentirá "tuya" casi siempre.

| Rama | Dueño(s) | Carpeta donde trabaja | Qué contiene |
|---|---|---|---|
| `main` | **Solo Yeison fusiona** (suplente: Heider) | — | La versión buena, siempre funcional. Nadie escribe directo; todo entra por PR (§6). |
| `feat/pipeline` | Heider + Yeison | `notebooks/exploration/`, `notebooks/pipeline/` | Medallion (Bronze/Silver/Gold), streaming, la tabla Gold. |
| `feat/modelo` | Sara | `notebooks/modeling/` | Features, entrenamiento, evaluación, clustering. |
| `feat/viz` | Kelly | `notebooks/analysis/`, `reports/powerbi/` | EDA-funnel, tablero Power BI, narrativa. |
| `feat/ab-test` | Yeison | `docs/` (un `.md` de diseño) | Diseño del A/B test. |
| `feat/doc` | Yeison + equipo | `docs/`, `reports/` | Documento consolidado del PI, PPTX. |

> La regla de fondo: **cada frente toca su carpeta**. Eso, más que el nombre de la rama, es lo que evita el 90% de los conflictos.

---

## 4. El bucle diario (el corazón de todo)

Este es el ciclo que repites cada vez que vas a trabajar. Cópialo y ténlo a mano.

```bash
# 1) Párate en main y trae lo último que subió el equipo
git checkout main
git pull

# 2) Pásate a tu rama de frente
#    - Primera vez que la creas:
git checkout -b feat/modelo
#    - Si ya existe (los demás días), solo cámbiate a ella:
git checkout feat/modelo

# 3) ...trabaja en tu carpeta...
#    Guarda commits PEQUEÑOS y SEGUIDOS (no uno gigante al final del día):
git add .
git commit -m "modelo: baseline logístico con PR-AUC"

# 4) ANTES de subir, trae lo nuevo de main a tu rama
#    (aquí aparecen los conflictos, si los hay — se resuelven en TU rama, no en main)
git pull origin main

# 5) Sube tu rama a GitHub
#    OJO: esto sube a TU rama, NO a main. main se actualiza cuando Yeison fusiona tu PR (§6).
#    - Primera vez:
git push -u origin feat/modelo
#    - Los siguientes push de esa rama:
git push
```

Luego abres el PR en la web y le avisas a Yeison para el merge (§6). Y cuando otro frente fusiona algo a `main`, tú lo recibes con el paso 1 (`git checkout main && git pull`).

**Equivalente en GitHub Desktop:** "Fetch/Pull origin" (pasos 1 y 4), "Current branch → New branch" (paso 2), escribir mensaje y "Commit to feat/..." (paso 3), "Push origin" (paso 5). Mismos conceptos, con botones.

---

## 5. Reglas de oro para no chocar

1. **Nunca commitear directo a `main`.** Todo entra por PR, y **el merge del PR a `main` lo hace solo Yeison** (suplente: Heider). Tú trabajas en tu rama y haces `push` a tu rama; Yeison abre la puerta a `main`. (Es *la* regla.)
2. **Ramas de vida corta.** Vives en tu rama horas o un día y fusionas apenas una pieza esté estable. El peor escenario es que cada quien guarde su rama 4 días y haga un merge gigante el domingo.
3. **`pull` de `main` antes de empezar y antes de subir** (pasos 1 y 4). Así los conflictos los resuelves tú, en tu rama, y `main` nunca queda roto.
4. **Commits pequeños y con mensaje claro.** Formato simple: `frente: qué hiciste`. Ejemplos:
   - `pipeline: validar join Gold, corregir sesgo de sesión`
   - `viz: tablero v1 conectado a Gold agregada`
   - `docs: actualizar §15 plan con fechas`
5. **Cada quien en su carpeta.** Si necesitas tocar la carpeta de otro frente, avísale primero en el chat.
6. **Archivos compartidos = avisar antes.** La documentación consolidada y los contratos (esquema Gold, salida del modelo, datos del dashboard) los tocamos todos. Antes de editarlos: avisa en el chat, haz commits chiquitos, y `pull` justo antes.
7. **Archivos binarios no se fusionan.** El `.pbix` de Power BI, imágenes (`.png`), Excel (`.xlsx`) y el PPTX **no se pueden mezclar automáticamente**: si dos personas editan el mismo `.pbix`, una pierde su trabajo. Regla: **un solo editor a la vez** para esos archivos; coordínalo en el chat.
8. **Nunca uses `git push --force`** sobre una rama que comparten dos personas (como `feat/pipeline`). Borra el trabajo del otro.

---

## 6. Abrir y fusionar un Pull Request (desde la web de GitHub)

1. Después de tu `push`, entra a `https://github.com/HeiderZapata/proyecto-integrador-g8`.
2. GitHub muestra un botón **"Compare & pull request"** para tu rama. Haz clic.
3. Verifica que sea **`feat/tu-rama` → `main`**. Pon un título claro y, en la descripción, una o dos líneas de qué cambiaste.
4. Crea el PR.
5. **Antes de pedir el merge, sincroniza tu rama con `main`** (paso 4 del bucle): `git pull origin main`, resuelve cualquier conflicto en *tu* rama (§7) y vuelve a hacer `push`. El PR debe quedar **sin conflictos / "Able to merge"** (GitHub lo muestra en verde). Esto es responsabilidad tuya, no de Yeison: así él recibe un PR limpio.
6. **El merge a `main` lo hace una sola persona: Yeison** (suplente: Heider si no está disponible, para que nadie quede bloqueado). Cuando tu PR esté en verde, avísale en el chat ("PR de modelo listo para merge"). **Nadie más toca el botón de merge.**
   - Esto centraliza el control de qué entra a `main` y evita que cuatro novatos fusionen a la vez. A cambio, Yeison se compromete a **no dejar PRs represados**.
   - Para archivos **compartidos** (documentación consolidada, contratos) o la **Gold** (afecta a todos), Yeison revisa con más cuidado antes de fusionar; para cambios en tu propia carpeta, el merge es casi un trámite.
7. Yeison da clic en **"Merge pull request"** —**este clic es el único momento en que `main` se actualiza**—, luego **"Delete branch"**, y avisa en el chat: "fusioné X a main, hagan `pull`".

---

## 7. Conflictos: qué son y cómo resolverlos sin pánico

Un conflicto pasa cuando tú y otra persona cambiaron **las mismas líneas** de un mismo archivo. Git no adivina cuál vale, así que te pregunta. **No es un error tuyo ni es grave.**

Aparece típicamente en el paso 4 (`git pull origin main`). Git te dirá algo como `CONFLICT in notebooks/...`. Pasos:

1. Abre el archivo en conflicto. Verás marcas así:
   ```
   <<<<<<< HEAD
   (tu versión)
   =======
   (la versión que venía de main)
   >>>>>>> main
   ```
2. **Decide a mano** qué quieres dejar: tu versión, la de main, o una mezcla de ambas. Borra las líneas de marca (`<<<<<<<`, `=======`, `>>>>>>>`) y deja el archivo como debe quedar.
3. Guarda, y dile a Git que ya resolviste:
   ```bash
   git add archivo-en-conflicto
   git commit -m "resolver conflicto en X"
   ```
4. Sigue con tu `push` normal.

**Si te sientes perdido:** no hagas más comandos. Avisa en el chat y resuélvanlo entre dos. Es mejor parar que enredar más.

---

## 8. Secretos y datos: lo que NUNCA va al repo

Esto es crítico y ya nos costó una vez (la credencial de Kaggle expuesta).

**Nunca subas al repo:**
- **Credenciales / tokens / llaves** (el `kaggle.json`, tokens de GitHub, contraseñas). Van en variables de entorno o en *Databricks secrets*, jamás en el código.
- **Datos crudos pesados** (los CSV de 14 GB, parquets grandes). El repo es para código y documentos, no para datos.
- **El material del profe** (`material_cursos/`): por peso y por propiedad intelectual.

Para esto existe el archivo **`.gitignore`** (ver apéndice): le dice a Git qué ignorar para que ni por accidente lo subas.

**Si subiste un secreto por error:** borrarlo del archivo **no basta** — queda en el historial y se asume comprometido. Hay que: **(1) rotar/revocar el secreto en su servicio** (p. ej. generar una llave nueva en Kaggle e invalidar la vieja), y **(2) limpiarlo del historial de Git** (esto es avanzado; avisa al equipo y háganlo con Claude Code o quien lleve la salud del repo). Primero rotar, siempre.

---

## 9. Si algo sale mal (recetas de rescate)

Guarda esta sección. Son los enredos más comunes de novatos y cómo salir.

**"Hice commit en `main` por error" (y no he hecho push):**
```bash
git branch feat/mi-trabajo     # 1) salva tu commit en una rama nueva
git reset --hard origin/main   # 2) regresa main a como está en GitHub (¡descarta cambios sin guardar!)
git checkout feat/mi-trabajo   # 3) sigue trabajando en tu rama
```

**"Mi `push` fue rechazado" (rejected / non-fast-forward):**
Casi siempre es porque alguien subió algo y a ti te falta traerlo. Solución:
```bash
git pull origin feat/tu-rama   # trae lo que falta (resuelve conflicto si aparece, §7)
git push                       # ahora sí
```

**"Quiero deshacer mi último commit" (aún no subido):**
```bash
git reset --soft HEAD~1   # deshace el commit pero CONSERVA tus cambios para rehacerlo
```
Evita `git reset --hard` salvo que quieras **borrar** esos cambios para siempre.

**"Subí un archivo pesado o un secreto por error":** para. No sigas con más comandos. Avisa al equipo de inmediato; si era un secreto, rótalo ya (§8). La limpieza del historial se hace en grupo.

**"Estoy enredado y no sé en qué estado quedó todo":**
```bash
git status     # te dice en qué rama estás y qué cambios tienes
git log --oneline -5   # muestra los últimos 5 commits
```
Con eso, pega la salida en el chat y lo vemos juntos.

---

## 10. Chuleta rápida (cheat sheet)

```bash
# Empezar el día
git checkout main && git pull
git checkout feat/tu-rama        # o:  git checkout -b feat/tu-rama  (primera vez)

# Guardar avances
git add .
git commit -m "frente: qué hiciste"

# Subir (antes, traer main)
git pull origin main
git push                          # o:  git push -u origin feat/tu-rama  (primera vez)

# Ver en qué estás
git status
git log --oneline -5
```

Luego: sincronizar (`git pull origin main`) → abrir PR en GitHub → avisar a Yeison → (Yeison) fusiona y borra la rama → todos hacen `git pull` de `main`.

---

## Apéndice — `.gitignore` sugerido

Crear este archivo en la raíz del repo. Ignora lo que nunca debe subirse:

```gitignore
# --- Datos crudos y pesados (el repo NO es un data lake) ---
/data/
*.csv
*.parquet
*.tsv
# Excepción: la Gold AGREGADA pequeña sí la versionamos para Power BI.
# Si la guardas, ponla en reports/data/ y fuérzala con:  git add -f reports/data/gold_agg.parquet

# --- Material del curso (peso + IP del profe) ---
material_cursos/

# --- Secretos y credenciales (NUNCA al repo) ---
.env
*.env
kaggle.json
*secret*
*credential*
*.key

# --- Checkpoints de streaming / Spark ---
checkpoint/
_checkpoints/
spark-warehouse/

# --- Power BI: decidir en equipo si versionar el .pbix (es binario y pesado) ---
# *.pbix

# --- Basura del sistema y editores ---
.DS_Store
.ipynb_checkpoints/
__pycache__/
.vscode/
.idea/
```

> Nota: ignorar `*.csv`/`*.parquet` evita subir datos pesados por accidente. Si necesitas versionar un archivo pequeño concreto (como la Gold agregada), usa la excepción comentada arriba (`git add -f ...`).
