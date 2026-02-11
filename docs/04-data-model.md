# CENEVAL Duel System — Data Model (docs/04-data-model.md)

## 1) Objetivo del modelo de datos
El modelo de datos debe permitir:
- Registrar preguntas (tema, dificultad, opciones, respuesta, explicación).
- Asignar sets diarios por usuario (incluyendo estado y avance).
- Guardar intentos (con idempotencia) y calcular métricas.
- Soportar competencia justa (precisión, progreso, trofeos por tema, duelo sábado).
- Habilitar dashboard (rankings, heatmaps, preguntas falladas, racha).

La DB es la **fuente de verdad**. WhatsApp es el canal. n8n ejecuta la lógica.

---

## 2) Entidades principales
- **users**: David, Kamila (y potencialmente más a futuro).
- **topics**: catálogo de temas (núcleo común + especialidades).
- **question_bank**: repositorio reutilizable de preguntas.
- **weekly_plan**: estrategia y referencia de la semana (70/30).
- **daily_assignment**: qué preguntas le tocaron a cada usuario cada día + estado.
- **attempts**: cada respuesta individual (timestamp, correcto/incorrecto, origen).
- **scores_daily**: resumen del día por usuario.
- **scores_weekly**: resumen semanal por usuario (incluye progreso).
- **duels**: set del sábado (mismo set para ambos).

---

## 3) Esquema propuesto (MVP)
> Tipos pensados para Postgres/Supabase. UUIDs generados en DB o en n8n.

### 3.1 users
- `id` (uuid, pk)
- `name` (text) — "David", "Kamila"
- `phone_e164` (text, unique) — ej. "+521..."
- `track` (text) — "mechanical" | "mechatronics"
- `is_active` (bool, default true)
- `created_at` (timestamptz, default now())

---

### 3.2 topics
- `id` (uuid, pk)
- `name` (text, unique) — "Matemáticas", "Control", etc.
- `category` (text) — "core" | "mechanical" | "mechatronics"
- `created_at` (timestamptz, default now())

---

### 3.3 question_bank
Banco reutilizable de preguntas. Puede incluir renderizado para ecuaciones.

- `id` (uuid, pk)
- `topic_id` (uuid, fk -> topics.id)
- `difficulty` (int) — 1 (baja) | 2 | 3 (alta)
- `question` (text)

**Opciones / respuesta**
- `options` (jsonb) — `{"A":"...", "B":"...", "C":"...", "D":"..."}`
- `correct_option` (text) — "A"|"B"|"C"|"D"
- `explanation` (text) — 2–4 líneas idealmente

**Render (ecuaciones e imágenes)**
- `render_mode` (text, default "text") — "text" | "latex_png" | "image"
- `latex` (text, nullable) — contenido LaTeX si aplica
- `media_id` (text, nullable) — cache del upload a WhatsApp (si aplica)
- `media_sha` (text, nullable) — hash del PNG para cache/dedup (opcional)

**Origen / deduplicación**
- `source_tag` (text) — "generated" | "manual" | "imported"
- `hash` (text, unique, nullable) — para deduplicar (recomendado)
- `created_at` (timestamptz, default now())

Notas:
- `media_id` permite reusar la misma imagen sin re-render / re-upload.
- `hash` puede ser un hash del texto de la pregunta+opciones+respuesta.

---

### 3.4 weekly_plan
- `id` (uuid, pk)
- `week_start_date` (date) — lunes de esa semana
- `strategy` (jsonb) — pesos 70/30, targets por persona, etc.
- `created_at` (timestamptz, default now())

Ejemplo `strategy`:
```json
{
  "ratio": {"weak": 0.7, "maint": 0.3},
  "targets": {
    "David": ["Control", "Matemáticas"],
    "Kamila": ["Electrónica", "Control"]
  },
  "notes": "Semana 3: subir precisión en Control, mantener Matemáticas"
}
