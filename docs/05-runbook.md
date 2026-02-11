# CENEVAL Duel System — Runbook (docs/05-runbook.md)

## 1) Propósito
Este runbook explica:
- cómo correr el sistema (MVP → completo),
- cómo verificar que todo está conectado,
- y qué hacer si algo falla.

Arquitectura: WhatsApp ↔ n8n ↔ DB ↔ Dashboard

---

## 2) Requisitos

### Servicios
- WhatsApp Cloud API (Meta)
- n8n (self-hosted o cloud)
- Postgres (recomendado Supabase)
- Dashboard (Next.js/React) — opcional al inicio
- (Opcional) Render de ecuaciones: servicio/script LaTeX → PNG (KaTeX/MathJax)

### Variables sensibles (NO subir a Git)
- Token WhatsApp
- Verify token del webhook
- Supabase URL + keys (service role o key de servidor)
- OpenAI API key (si usan generación semanal)
- (Opcional) URL/credenciales del servicio de render de ecuaciones

---

## 3) Checklist de “arranque rápido” (MVP)

### Paso A — DB lista (Supabase/Postgres)
1) Crear proyecto en Supabase
2) Crear tablas del modelo (ver `docs/04-data-model.md`)
3) Insertar `users` (David y Kamila) con `phone_e164`
4) Insertar `topics`

**Validación**
- Puedo hacer SELECT de `users` y `topics` sin error.
- Los `phone_e164` coinciden con el número que WhatsApp entrega (formato E.164).

---

### Paso B — WhatsApp Cloud API (Meta)
1) Crear app en Meta Developers
2) Activar WhatsApp Cloud API
3) Configurar número de prueba (sandbox)
4) Obtener:
   - `PHONE_NUMBER_ID`
   - `WHATSAPP_TOKEN`
5) Configurar Webhook:
   - URL pública del Inbound Handler (n8n)
   - `WHATSAPP_VERIFY_TOKEN`
   - Suscribir eventos necesarios (mensajes entrantes)

**Validación**
- Con un request manual (Postman/curl) puedo enviar un mensaje de texto a mi número.
- Meta muestra el webhook como “verificado” (si aplica).

---

### Paso C — n8n (webhooks + workflows)
Workflows mínimos:
1) **Inbound Handler**
   - Webhook público para recibir mensajes
   - Lookup user por `phone_e164`
   - Interpretación de respuesta:
     - preferente: payload **interactive list/button**
     - fallback: texto `A|B|C|D` (si llega)
   - Calificar vs `question_bank.correct_option`
   - Insert en `attempts` (idempotente)
   - Avanzar `current_index` solo si el intento se guardó
   - Responder feedback por WhatsApp

2) **Daily Sender**
   - Cron (09:00 y/o 19:00)
   - Lee `daily_assignment`
   - Envía siguiente pregunta/bloque
   - Actualiza `status` (pending → in_progress → completed)

**Validación**
- Si selecciono una opción en WhatsApp (lista/botón), el sistema contesta “✅/❌ + explicación”.
- `attempts` se llena en la DB.
- `current_index` avanza correctamente.

---

## 4) Operación diaria (cómo se usa)
- El sistema manda preguntas por WhatsApp (idealmente con **Interactive List A/B/C/D**).
- Ustedes responden tocando una opción (cero escritura).
- Feedback inmediato por pregunta.
- Al final del día puede llegar resumen (si está habilitado).
- Sábado: se manda el duelo (mismo set a ambos).
- Domingo: retro semanal (precisión, progreso, debilidades, plan).

---

## 5) Health checks (cuando algo “se siente raro”)

### 5.1 WhatsApp no manda mensajes (Outbound falla)
Verificar:
- Token vigente (expiró / se revocó)
- `PHONE_NUMBER_ID` correcto
- Rate limit / plantilla requerida (si ya no están en sandbox o si el mensaje viola ventana)
- Logs de n8n: ¿falló el nodo HTTP?

Acción rápida:
- Enviar un mensaje “PING” desde n8n con un workflow manual.
- Si falla, revisar respuesta HTTP (401/403/429/5xx) y el body.

---

### 5.2 n8n no recibe mensajes (Inbound / webhook muerto)
Verificar:
- URL pública accesible (sin auth extra)
- Verify token correcto
- Meta app webhook suscrito a eventos correctos
- Workflow en n8n activo

Acción rápida:
- Probar el endpoint con un POST simple (o usando “Webhook test” de Meta Developers).
- Revisar en n8n si el webhook registró “hit”.

---

### 5.3 Mensajes interactivos no aparecen (no veo lista/botones)
Verificar:
- El payload que envías corresponde a **interactive list** o **reply buttons** válido
- Longitudes y formato (IDs únicos y cortos)
- Ventana de mensajería / tipo de mensaje permitido según el estado (sandbox vs producción)

Acción rápida:
- Probar enviando un interactive mínimo (lista con 2 opciones).
- Si eso sí funciona, el problema es de estructura/longitud en el mensaje real.

---

### 5.4 Llega respuesta, pero no la entiende (payload raro)
Verificar:
- El Inbound Handler está leyendo el campo correcto:
  - `interactive.list_reply.id` (lista)
  - `interactive.button_reply.id` (botón)
  - fallback: texto plano

Acción:
- Loggear (en n8n) el “tipo de mensaje” y el campo extraído.
- Asegurar que los IDs de opciones sean estables (ej. `"A"`, `"B"`, `"C"`, `"D"`).

---

### 5.5 Califica mal (marca incorrecta cuando era correcta)
Verificar:
- La pregunta “activa” que se está calificando
- `current_index` en `daily_assignment`
- Si el usuario respondió rápido y el sistema avanzó de más (race condition)

Acciones:
- Bloquear doble avance: solo avanzar índice cuando el insert de attempt fue exitoso.
- Incluir en feedback: “Pregunta #X del día” para diagnóstico rápido.
- En caso de desalineación, reconstruir el índice (ver sección 6).

---

### 5.6 Se duplican attempts
Causa típica:
- Webhook reintenta o llega duplicado.

Mitigación:
- Idempotencia:
  - guardar `message_id` (de WhatsApp) y hacer `UNIQUE(message_id)`
  - o clave única por (user_id + question_id + date)
- Si llega duplicado, responder “recibido” sin avanzar.

Acción:
- Crear índice único y manejar el conflicto (upsert/no-op).

---

### 5.7 Media upload falla (ecuación/imagen no se envía)
Verificar:
- Token y endpoint correctos
- Tamaño/formato permitido (PNG/JPG)
- Si el `media_id` se guardó bien y no está vacío

Acciones:
- Reintentar upload 1–2 veces.
- Si falla, fallback: enviar la ecuación en texto (modo degradado) y registrar incidente.

---

### 5.8 Scores no cuadran
Verificar:
- ¿scores se calculan incremental o por batch?

Recomendación:
- En MVP, calcular scores por batch (query a `attempts`) al final del día/semana.

Acción:
- Recalcular `scores_daily` y `scores_weekly` desde `attempts`.

---

## 6) Recuperación (cuando “se rompió” un día)

### Caso A: No se enviaron preguntas hoy
- Cambiar `status` del día a `pending`
- Re-ejecutar workflow Daily Sender manual
- O enviar un “set compactado” (menos preguntas) para no romper la racha

### Caso B: Se perdió o corrompió el estado de pregunta actual
Objetivo: reconstruir `current_index`.

Pasos:
1) Contar intentos del usuario para ese día (y assignment)
2) Setear `current_index = attempts_count`
3) Continuar desde ahí

Si hay dudas:
- Bloquear el set del día (status = paused) y reiniciar con un set nuevo de menor tamaño.

### Caso C: WhatsApp caído / bloqueado
- Enviar el set por un canal alterno temporal (Telegram/email)
- Registrar el día como “maintenance”
- Opcional: pausar racha para ese día (no castigar)

---

## 7) Backups y mantenimiento (mínimo)
- Supabase: habilitar backups automáticos (si está disponible)
- Export semanal del `question_pack` JSON (para reproducibilidad):
  - guardar en `infra/supabase/seed/` o en `scripts/question-pack-builder/outputs/`
- Export de workflows n8n en `automations/n8n/workflows/`
- Logs de n8n: retención 7–14 días

---

## 8) Procedimiento antes del duelo del sábado
- Confirmar que `duels.question_ids` está creado
- Enviar a ambos el mismo set (ideal: interactive list A/B/C/D)
- Validar que se registran attempts con esos `question_ids`
- Cerrar duelo con un resumen:
  - precisión del duelo
  - ganador
  - 3 preguntas más falladas del set (opcional)

---

## 9) Modo ecuaciones e imágenes (operación)

### 9.1 Cuándo usar imagen
- Integrales, fracciones complejas, matrices, diagramas, o notación ambigua en texto.

### 9.2 Flujo operativo (con cache)
1) Pregunta tiene `render_mode = latex_png` y `latex`.
2) Si `media_id` existe en DB → reutilizar.
3) Si no existe:
   - renderizar LaTeX → PNG
   - subir a WhatsApp → obtener `media_id`
   - guardar `media_id` en DB
4) Enviar pregunta con imagen + interactive list.

### 9.3 Si falla el render o upload
- Fallback: enviar la ecuación en texto, marcar la pregunta como “degradada” y continuar (no romper el flujo).

---

## 10) Pruebas recomendadas (antes de usarlo en serio)
### Prueba 1 — Inbound básico
- Enviar un mensaje cualquiera y confirmar que n8n recibe el webhook.

### Prueba 2 — Interactive list
- Enviar una pregunta con lista A/B/C/D y confirmar que el payload recibido trae el `id` correcto.

### Prueba 3 — Persistencia
- Confirmar que el attempt aparece en DB y que `current_index` avanza.

### Prueba 4 — Media upload
- Enviar una pregunta con imagen, verificar `media_id` y que la imagen se ve en WhatsApp.

### Prueba 5 — Resumen diario
- Completar un set y confirmar que el resumen coincide con el conteo de attempts.

---

## 11) “Modo emergencia” (si no hay tiempo)
Si falla todo el sistema:
- Enviar 10–15 preguntas por WhatsApp manualmente (texto)
- Responder con A/B/C/D
- Registrar solo el conteo en una nota o sheet
Lo importante es NO romper el hábito.

---

## 12) Definición de “Done” del sistema
- Envío diario automático (o manual con fallback) funcionando.
- Respuestas por interactive list (preferente) y calificación correcta.
- Registro de `attempts` sin duplicados (idempotencia).
- Resumen diario confiable.
- Precisión semanal y progreso calculados.
- Duelo del sábado ejecutado y reportado.
- Dashboard mostrando rankings y debilidades (cuando se active).
