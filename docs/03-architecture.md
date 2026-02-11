# CENEVAL Duel System — Architecture (docs/03-architecture.md)

## 1) Visión general
El sistema tiene 4 piezas principales:

1) **WhatsApp Cloud API (Meta)**  
Canal de envío de preguntas y recepción de respuestas. Las respuestas se capturan preferentemente con **mensajes interactivos** (lista/botones) para que el usuario solo “toque” una opción.

2) **n8n (orquestador)**  
Automatiza envíos, recibe webhooks, interpreta respuestas (payload interactivo), valida, calcula puntaje y guarda datos. También ejecuta la lógica 70/30, repetición espaciada y reportes.

3) **Base de datos (Postgres / Supabase recomendado)**  
Fuente de verdad: banco de preguntas, planes semanales, asignaciones diarias, estado de avance (`current_index`), intentos (`attempts`) y métricas (scores y desempeño por tema).

4) **Dashboard (Next.js/React)**  
Visualización de progreso, ranking (precisión/progreso), heatmap por tema, historial, preguntas falladas y resultados del duelo.

---

## 2) Diagrama de arquitectura (alto nivel)
```mermaid
flowchart LR
  WA[WhatsApp Cloud API] -->|Inbound Webhook| N8N[n8n]
  N8N -->|Reply / Feedback| WA
  N8N --> DB[(Supabase/Postgres)]
  N8N -->|Weekly pack generation| LLM[OpenAI API]
  DASH[Dashboard Next.js] --> DB
