# CENEVAL Duel System — WhatsApp UX Standard (docs/06-whatsapp-ux.md)

## 1) Propósito
Este documento define un estándar único para:
- cómo se envían preguntas por WhatsApp,
- cómo se capturan respuestas (sin teclear, cuando sea posible),
- cómo manejar ecuaciones (LaTeX → imagen),
- y cómo mantener mensajes consistentes y fáciles de operar.

Meta: **máxima velocidad + mínima fricción**, sin perder claridad.

---

## 2) Estándar de mensajes (por tipo de pregunta)

### 2.1 Opción múltiple (A/B/C/D) — estándar principal
**Formato**
- Mensaje con:
  1) Texto corto: enunciado + contexto mínimo
  2) (Opcional) imagen/diagrama
  3) Interacción: **Interactive List** con 4 opciones: A, B, C, D

**Regla**
- Siempre que exista opción A/B/C/D, se usa **Interactive List**.
- Fallback permitido: texto `A|B|C|D` (solo si falla interactive).

**Ventajas**
- 1 tap = 1 respuesta
- Menos errores de parsing
- Más constancia (fricción casi 0)

---

### 2.2 Ecuaciones complejas — imagen desde LaTeX (recomendado)
Usar imagen cuando:
- hay fracciones largas, integrales, matrices, sumatorias,
- o cuando el texto sería ambiguo.

**Formato**
- Mensaje con:
  1) Texto breve: “Resuelve y elige la opción correcta”
  2) Imagen con la ecuación (PNG)
  3) Interactive List A/B/C/D

**Regla**
- Si `render_mode = latex_png`, la ecuación debe venir en `latex` y la imagen se cachea con `media_id`.

**Fallback**
- Si falla el render/upload:
  - enviar ecuación en texto (modo degradado),
  - mantener Interactive List.

---

### 2.3 Numéricas (respuesta abierta) — texto con tolerancia
Usar solo cuando:
- el valor numérico final sea importante,
- y no sea fácil convertir a opción múltiple.

**Formato**
- Mensaje con:
  1) Enunciado
  2) Instrucción clara de formato:  
     “Escribe solo el valor final sin unidades, redondea a 2 decimales”.
  3) (Opcional) rango esperado como pista: “El resultado está entre 0 y 10”.

**Calificación**
- Tolerancia: correcto si `abs(respuesta - valor) <= tol`
- Alternativa: redondeo estricto (menos recomendado)

**Regla**
- Las numéricas deben definir:
  - redondeo (n decimales) y/o tolerancia
  - si se aceptan notaciones tipo `1e-3`

---

## 3) Formatos recomendados de contenido

### 3.1 Plantilla de pregunta (texto)
- Encabezado: `Tema • Dificultad`
- Identificador: `QID: <uuid corto o hash corto>`
- Enunciado
- (Opcional) recordatorio: “Responde tocando A/B/C/D”

Ejemplo:
- `Control • D2`
- `QID: 8f2c`
- “¿Cuál es la función de transferencia de un sistema de primer orden…?”

---

### 3.2 Plantilla de feedback (respuesta del bot)
Debe ser corto y repetible:

- Línea 1: `✅ Correcto` o `❌ Incorrecto`
- Línea 2: `Correcta: C`
- Línea 3–4: explicación (2–4 líneas máximo)
- Línea final (opcional): `Puntaje hoy: 7/10 • Racha: 3`

---

## 4) IDs y consistencia (crítico para n8n)
### 4.1 IDs de opciones
Para Interactive List:
- El `id` de cada opción debe ser exactamente:
  - `"A"`, `"B"`, `"C"`, `"D"`

No usar IDs largos ni con espacios.

### 4.2 Identificación de la pregunta activa
Dos enfoques válidos (elige uno y estandariza):

**A) Estado por assignment (recomendado)**
- n8n consulta `daily_assignment.current_index` para saber qué pregunta está activa.
- La respuesta del usuario se asume para esa pregunta activa.

**B) QID en el mensaje (más robusto)**
- El mensaje incluye `QID: xxxx`
- En el payload de cada opción, el `id` puede codificar: `A|QID`
  - ejemplo: `A:8f2c`
- n8n extrae QID y califica directo.

Recomendación práctica:
- MVP: A (por simplicidad)
- V1+: B (para evitar desalineación si el usuario responde tarde)

---

## 5) Ecuaciones: pipeline LaTeX → PNG → WhatsApp

### 5.1 Campos esperados en la DB (question_bank)
- `render_mode`: "text" | "latex_png" | "image"
- `latex`: string LaTeX (nullable)
- `media_id`: cache (nullable)
- `media_sha`: hash del PNG (opcional)

### 5.2 Flujo (operativo)
1) Si `render_mode = latex_png`:
   - Si `media_id` existe → usarlo
   - Si no existe:
     - renderizar LaTeX a PNG
     - subir PNG a WhatsApp → guardar `media_id`
2) Enviar mensaje con media + Interactive List.

### 5.3 Reglas de estilo de LaTeX (para consistencia visual)
- Fondo blanco, texto negro
- Tamaño legible (evitar letra pequeña)
- Márgenes suficientes (no recortar símbolos)
- Exportar a PNG con ancho consistente (ej. 900–1200 px)

---

## 6) Ejemplos de payload inbound (simplificados)

> Nota: Estos ejemplos no son completos; son la parte relevante que n8n debe leer.

### 6.1 Interactive List reply (A/B/C/D)
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
          "id": "C",
          "title": "C"
        }
      }
    }
  ]
}
