# n8n — Credentials Template (automations/n8n/credentials.example.md)

> Este archivo es un **ejemplo**. NO pegues credenciales reales aquí.
> La idea es que cualquiera pueda configurar n8n rápido sin exponer secretos.

## 1) En dónde se guardan los secretos
En n8n, los secretos se guardan como:
- **Credentials** (recomendado para tokens/keys)
- **Environment variables** (`.env` del servidor/contenedor)
- **Static Data / Variables** (solo para cosas no sensibles)

Regla:
- **Tokens/API keys → Credentials o env vars**
- **IDs o configs no sensibles → Variables**

---

## 2) Convención de nombres (para que el equipo no se pierda)
Usa estos nombres exactos en n8n:

### Credenciales
- `cred_whatsapp_cloud_api`
- `cred_supabase_service_role`
- `cred_openai_api` (si aplica)
- `cred_equation_renderer` (si aplica)

### Workflows (sugeridos)
- `wf_inbound_handler`
- `wf_daily_sender`
- `wf_weekly_pack_builder`
- `wf_weekly_review`
- `wf_duel_saturday`

---

## 3) WhatsApp Cloud API (Meta)

### 3.1 Variables / campos necesarios
- `WHATSAPP_TOKEN` → Token de acceso (Bearer)
- `PHONE_NUMBER_ID` → ID del número de WhatsApp
- `WHATSAPP_VERIFY_TOKEN` → Token para verificación del webhook (string definido por ustedes)

### 3.2 Cómo configurarlo en n8n
**Opción A — Credential (recomendado)**
Crear un credential de tipo:
- **HTTP Header Auth** (o “Generic Credential” equivalente)
Con:
- Header: `Authorization`
- Value: `Bearer {{WHATSAPP_TOKEN}}`

Y adicionalmente guardar como variables (no necesariamente secret):
- `PHONE_NUMBER_ID`
- `WHATSAPP_VERIFY_TOKEN`

**Opción B — Env vars**
Guardar en el entorno donde corre n8n:
- `WHATSAPP_TOKEN=...`
- `PHONE_NUMBER_ID=...`
- `WHATSAPP_VERIFY_TOKEN=...`

### 3.3 Endpoints que usa el sistema
- Send message:
  - `POST https://graph.facebook.com/vXX.X/{{PHONE_NUMBER_ID}}/messages`
- Media upload:
  - `POST https://graph.facebook.com/vXX.X/{{PHONE_NUMBER_ID}}/media`

*(La versión `vXX.X` depende de tu setup en Meta.)*

---

## 4) Supabase / Postgres

### 4.1 Variables / campos necesarios
- `SUPABASE_URL`
- `SUPABASE_SERVICE_ROLE_KEY` (solo servidor)
- (Opcional) `SUPABASE_ANON_KEY` (solo lectura pública si lo ocupan)
- `SUPABASE_DB_URL` (si conectan directo Postgres)
  - Ejemplo: `postgresql://USER:PASSWORD@HOST:5432/postgres`

### 4.2 Cómo configurarlo en n8n
**Opción A — Supabase REST**
- Usar HTTP Request nodes apuntando a:
  - `{{SUPABASE_URL}}/rest/v1/<table>`
- Header:
  - `apikey: {{SUPABASE_SERVICE_ROLE_KEY}}`
  - `Authorization: Bearer {{SUPABASE_SERVICE_ROLE_KEY}}`
  - `Content-Type: application/json`

**Opción B — Postgres node**
- Crear credential “Postgres” con:
  - Host, DB, User, Password, Port
- Recomendado si vas a hacer queries complejas.

---

## 5) OpenAI (opcional: generación semanal en batch)

### 5.1 Variables / campos necesarios
- `OPENAI_API_KEY`
- `OPENAI_MODEL` (ej. `gpt-5-mini`)
- (Opcional) `OPENAI_REASONING_EFFORT` si lo parametrizan

### 5.2 Cómo configurarlo en n8n
- Credential “OpenAI API” (si está disponible)
o
- HTTP Request con:
  - Header: `Authorization: Bearer {{OPENAI_API_KEY}}`

---

## 6) Renderizador de ecuaciones (opcional)
Si usas un servicio interno que recibe LaTeX y devuelve PNG:

### 6.1 Variables / campos necesarios
- `EQUATION_RENDER_URL` (ej. `https://<tu-servicio>/render`)
- `EQUATION_RENDER_TOKEN` (si está protegido)

### 6.2 Contrato esperado
**Request**
- `POST {{EQUATION_RENDER_URL}}`
- body JSON:
  - `{"latex":"\\frac{1}{2}mv^2", "width":1000}`

**Response**
- PNG bytes o JSON con `image_base64`

*(Si no tienen esto todavía, pueden empezar con texto y luego lo implementan.)*

---

## 7) Webhook inbound (n8n)
### 7.1 Requisitos
- URL pública accesible
- `WHATSAPP_VERIFY_TOKEN` debe coincidir con lo configurado en Meta

### 7.2 Campos recomendados a guardar del inbound
- `wa_message_id` (para idempotencia)
- `from` (phone_e164)
- `type` (interactive/text)
- `interactive.list_reply.id` (A/B/C/D) o `text.body`

---

## 8) Ejemplo de `.env.example` (solo plantilla)
```bash
# WhatsApp
WHATSAPP_TOKEN=REPLACE_ME
PHONE_NUMBER_ID=REPLACE_ME
WHATSAPP_VERIFY_TOKEN=REPLACE_ME

# Supabase
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=REPLACE_ME

# OpenAI (optional)
OPENAI_API_KEY=REPLACE_ME
OPENAI_MODEL=gpt-5-mini

# Equation renderer (optional)
EQUATION_RENDER_URL=https://your-render-service/render
EQUATION_RENDER_TOKEN=REPLACE_ME
