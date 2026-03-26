# Comandos habituales — SurveyApp

## 1. Carpeta del proyecto

| Qué | Dónde |
|-----|--------|
| **Repositorio Git** (`.git`) | Dentro de **`surveyapp/`**, no en la carpeta padre. |
| **Comandos `npm` del backend** | Desde **`surveyapp/`** (raíz del repo). |
| **Comandos del cliente (Vite)** | `cd client` o scripts que ya hacen `cd client` desde la raíz. |

Ejemplo de rutas típicas:

```text
.../surveyapp-main/surveyapp/     ← aquí: git, npm install, npm run dev
.../surveyapp-main/surveyapp/client/
```

Si en la terminal estás en `surveyapp-main` y Git dice “not a git repository”, haz **`cd surveyapp`**.

---

## 2. Desarrollo local (servidor + cliente)

**Terminal 1 — API (Express), desde `surveyapp/`:**

```bash
cd ruta/a/surveyapp
npm install          # la primera vez o tras cambiar dependencias
npm run dev
```

- Puerto por defecto: **3001**.
- Variables: `.env` en la raíz de `surveyapp/` y/o `server/.env`.

**Terminal 2 — Frontend (Vite), desde `surveyapp/`:**

```bash
cd ruta/a/surveyapp
npm run dev:client
```

- Puerto típico: **5173** → abre **http://localhost:5173**.
- El proxy de Vite envía `/api/*` a `http://localhost:3001`.

**Resumen:** dos terminales, ambas con cwd = **`surveyapp/`**; en el navegador usa el puerto del **cliente** (5173).

---

## 3. Git: sincronizar manualmente tu remoto con otro repo

Útil cuando tienes **dos repos** (por ejemplo el de un compañero con el código actualizado y **el tuyo** conectado a Vercel) y quieres que **tu** `main` quede **igual** al otro.

**Nomenclatura habitual:**

- **`origin`** → repo que quieres usar como **fuente de verdad** (ej. `https://github.com/JanerBus/surveyapp.git`).
- **`teammate`** (o `mi-repo`, el nombre que elijas) → **tu** repo donde harás push (ej. `https://github.com/xKQAx/surveyapp.git`).

Añade el remoto del compañero si no existe:

```bash
cd ruta/a/surveyapp
git remote -v
git remote add origin https://github.com/ALGUIEN/surveyapp.git    # si hace falta
git remote add teammate https://github.com/TU_USUARIO/surveyapp.git
```

**Forzar que tu rama `main` en `teammate` sea una copia exacta de `origin/main`:**

```bash
git fetch origin
git checkout main
git reset --hard origin/main
git push teammate main --force
```

- **`git fetch origin`** — trae commits del remoto `origin`.
- **`git checkout main`** — asegúrate de estar en `main`.
- **`git reset --hard origin/main`** — tu `main` local = `origin/main` (pierdes commits locales no subidos).
- **`git push teammate main --force`** — actualiza **tu** GitHub (`teammate`) para que Vercel despliegue ese código.

**Si tu `origin` ya es tu repo y el del compañero tiene otro nombre** (ej. `janerbus`), usa ese nombre en lugar de `origin`:

```bash
git fetch janerbus
git checkout main
git reset --hard janerbus/main
git push origin main --force
```

Tras el push, espera el deploy en Vercel o lanza un **Redeploy** manual.

**Historiales no relacionados:** si Git dice `refusing to merge unrelated histories`, o usas `--allow-unrelated-histories` en un merge, o la secuencia `fetch` + `reset --hard` + `push --force` como arriba para **reemplazar** tu `main`.

---

## 4. Instalación (primera vez)

```bash
cd surveyapp
npm install
cd client && npm install && cd ..
```

---

## 5. Build y preview del cliente

```bash
cd surveyapp
npm run build
npm run preview:client
```

## 6. Producción local (Express sirviendo el build)

```powershell
cd surveyapp
$env:NODE_ENV="production"
npm run build
node server/index.js
```

---

## 7. Comprobar la API en local

```bash
curl http://localhost:3001/api/health
curl http://localhost:3001/api/surveys
```

---

## 8. Archivos SQL (Supabase)

| Archivo | Uso |
|---------|-----|
| `supabase/schema.sql` | Tablas, columnas extra, índices y RLS. Ejecutar primero en el SQL Editor. |
| `supabase/seed_five_surveys.sql` | Las 5 encuestas alineadas con la app (borra datos previos en esas tablas). Después de `schema.sql`. |
| `supabase/seed_example.sql` | Una encuesta de demostración (alternativa). |
| `supabase/fix_rls_anon.sql` | Si fallan los INSERT con la clave `anon`. |

---

## 9. Conectar el proyecto local a Supabase (checklist)

1. Supabase → **Settings → API** → URL y claves.
2. Crea/edita **`.env`** en `surveyapp/` o `surveyapp/server/.env` (`server/.env` tiene prioridad si repites claves).
3. Ejecuta **`supabase/schema.sql`** y, si aplica, **`seed_five_surveys.sql`**.
4. Reinicia `npm run dev` y revisa `GET /api/health` (`database.configured`, `database.connected`).

---

## 10. Vercel + Supabase

- **Root Directory:** si el repo tiene la app en una subcarpeta, en Vercel pon **`surveyapp`** (donde está `vercel.json` y `api/`).
- **Variables:** `SUPABASE_URL` y clave (`SUPABASE_ANON_KEY`, `SUPABASE_SECRET_KEY`, etc.) en **Environment Variables**; **redeploy** tras cambiarlas.
- **`/api/health`** en producción debería incluir `"format": "v2"` y el objeto **`database`**. Si no, el deploy no usa este código o el root del proyecto en Vercel es incorrecto.
- **`VITE_API_URL`** puede ir vacío si front y API comparten dominio.

---

## 11. Emular Vercel en local (opcional)

```bash
cd surveyapp
npx vercel dev
```

---

## 12. Cómo encaja todo (resumen)

1. **Supabase** guarda tablas; el **backend** usa `@supabase/supabase-js` con variables de entorno.
2. El **frontend** solo llama a **`/api/...`**; en desarrollo Vite proxifica `/api` al puerto 3001.
3. Ver datos en **Supabase → Table Editor**. Crear encuestas nuevas desde la app tipo admin no está implementado en profundidad; puedes usar SQL o ampliar la API.

Si falla el guardado, revisa `database.error` en `/api/health`, RLS y que existan preguntas/opciones coherentes con el seed.
