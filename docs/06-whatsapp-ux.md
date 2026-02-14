# CENEVAL Duel System — WhatsApp UX Standard
`docs/06-whatsapp-ux.md`

## 1) Propósito
Este documento define un **estándar único y obligatorio** para:
- cómo se envían preguntas por WhatsApp,
- cómo se capturan respuestas con mínima fricción,
- cómo manejar ecuaciones (LaTeX → imagen),
- y cómo mantener consistencia operativa entre WhatsApp, n8n y la base de datos.

**Objetivo:** máxima velocidad, cero ambigüedad y cero desalineación al calificar.

---

## 2) Decisiones de diseño (clave)
Este sistema adopta una sola convención oficial:

- Toda pregunta tiene un **QID obligatorio**.
- Toda respuesta enviada por WhatsApp **incluye el QID**.
- **No se confía en estado implícito** (índices, “pregunta actual”, orden de envío).

> Regla de oro  
> **Cada respuesta debe identificar inequívocamente la pregunta que responde.**

---

## 3) Estándar de mensajes (por tipo de pregunta)

### 3.1 Opción múltiple (A/B/C/D) — estándar principal
**Formato**
1) Texto corto: enunciado + contexto mínimo  
2) Identificador visible: `QID: xxxx`  
3) (Opcional) imagen o diagrama  
4) Interacción: **Interactive List** con 4 opciones (A, B, C, D)

**Regla**
- Siempre que exista A/B/C/D → usar **Interactive List**.
- Fallback permitido: texto plano (`A`, `B`, `C`, `D`) solo si falla interactive.

**Ventajas**
- 1 tap = 1 respuesta  
- Parsing simple y robusto  
- Mayor constancia diaria (fricción casi cero)

---

### 3.2 Ecuaciones complejas — LaTeX renderizado a imagen (recomendado)
Usar imagen cuando:
- hay integrales, fracciones largas, matrices, sumatorias,
- o cuando el texto puede inducir error visual.

**Formato**
1) Texto breve: “Resuelve y elige la opción correcta”  
2) Imagen PNG con la ecuación  
3) Interactive List A/B/C/D

**Regla**
- Si `render_mode = latex_png`, la ecuación vive en el campo `latex`.
- El `media_id` debe cachearse para reutilización.

**Fallback**
- Si falla render o upload:
  - enviar ecuación en texto (modo degradado),
  - **mantener Interactive List**.

---

### 3.3 Numéricas (respuesta abierta) — uso restringido
Usar solo cuando **no sea viable** convertir a opción múltiple.

**Formato**
1) Enunciado  
2) Instrucción explícita de formato  
   - “Escribe solo el valor final, sin unidades, redondea a 2 decimales”
3) (Opcional) rango esperado como pista

**Calificación**
- Correcto si:  
  `abs(respuesta - valor) <= tolerancia`

**Regla**
Toda pregunta numérica debe definir explícitamente:
- número de decimales **o**
- tolerancia numérica,
- aceptación o no de notación científica (`1e-3`).

---

## 4) Convención de IDs (CRÍTICO)

### 4.1 QID (Question ID)
- Obligatorio en **todas** las preguntas.
- Visible en el mensaje.
- Estable y único.

**Formato recomendado**
- Hash corto o UUID truncado (4–6 caracteres).
- Ejemplos: `8f2c`, `a91e`, `c7d4`.

**Ejemplo en mensaje**
```

Control • D2
QID: 8f2c
¿Cuál es la función de transferencia de un sistema de primer orden…?

````

---

### 4.2 IDs de opciones (Interactive List)
Cada opción debe llevar **opción + QID** en el campo `id`.

| Opción | ID enviado |
|------|-----------|
| A | `A|8f2c` |
| B | `B|8f2c` |
| C | `C|8f2c` |
| D | `D|8f2c` |

**Reglas**
- No usar espacios.
- No usar textos largos.
- El separador oficial es `|`.

---

## 5) Identificación de la pregunta (decisión final)
Se adopta **un solo enfoque oficial**:

### ✅ QID embebido en la respuesta
- El mensaje muestra `QID`.
- El `list_reply.id` viene como `OPCION|QID`.
- n8n **no depende** de:
  - orden de envío,
  - índices,
  - estado previo,
  - timing del usuario.

**Motivo**
WhatsApp es asíncrono:
- respuestas tardías,
- reintentos de webhook,
- mensajes duplicados.

Este enfoque elimina esos riesgos por diseño.

---

## 6) Lógica esperada en n8n (resumen)
1) Recibir payload inbound.
2) Extraer `interactive.list_reply.id`.
3) Separar por `|` → `opcion`, `qid`.
4) Buscar pregunta por `qid`.
5) Calificar contra `correct_option`.
6) Guardar intento (idempotente por `wa_message_id`).
7) Enviar feedback inmediato.

---

## 7) Ecuaciones: pipeline LaTeX → PNG → WhatsApp

### 7.1 Campos esperados en `question_bank`
- `render_mode`: `"text"` | `"latex_png"` | `"image"`
- `latex`: string (nullable)
- `media_id`: cache de WhatsApp (nullable)
- `media_sha`: hash del PNG (opcional)

---

### 7.2 Flujo operativo
1) Si `render_mode = latex_png`:
   - si `media_id` existe → reutilizar,
   - si no:
     - renderizar LaTeX → PNG,
     - subir a WhatsApp,
     - guardar `media_id`.
2) Enviar mensaje con imagen + Interactive List.

---

### 7.3 Reglas visuales para LaTeX
- Fondo blanco, texto negro.
- Tamaño legible (evitar letra pequeña).
- Márgenes amplios (no recortar símbolos).
- Ancho consistente: **900–1200 px**.

---

## 8) Ejemplo de payload inbound (simplificado)
```json
{
  "messages": [
    {
      "from": "+521XXXXXXXXXX",
      "id": "wamid.HBgLM...",
      "type": "interactive",
      "interactive": {
        "type": "list_reply",
        "list_reply": {
          "id": "C|8f2c",
          "title": "C"
        }
      }
    }
  ]
}
````

---

## 9) Regla final (no negociable)

> **Si una respuesta no trae QID, no se califica.**

Esto mantiene el sistema:

* robusto,
* auditable,
* escalable,
* y libre de errores silenciosos.

```

---
