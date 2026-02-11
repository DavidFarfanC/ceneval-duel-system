# CENEVAL Duel System — Vision (docs/00-vision.md)

## 1) Objetivo
Construir un sistema de estudio para CENEVAL, diseñado para **dos personas (David y Kamila)**, que combine:
- **práctica diaria** con preguntas tipo examen,
- **retroalimentación inmediata**,
- **adaptación a debilidades** (personalización),
- **competencia sana** y motivación,
- y un **dashboard** para visualizar progreso y tomar decisiones semanales.

La meta es **maximizar el desempeño en el examen** mediante hábitos consistentes y aprendizaje medible, sin depender de “estudiar a ciegas”.

---

## 2) Problema que resolvemos
Estudiar sin estructura suele caer en:
- repasar lo que ya se domina,
- no medir debilidades reales por tema,
- poca constancia,
- y falta de retro semanal para ajustar.

Este sistema busca:
- **convertir el estudio en un proceso repetible**,
- **medir** con datos (no con sensaciones),
- y **optimizar el tiempo** reforzando lo que más impacta el puntaje.

---

## 3) Alcance (qué SÍ incluye)
### Funcionalidad de aprendizaje
- Envío diario de **15–20 preguntas** por persona.
- Respuestas por letra (A/B/C/D).
- **Calificación automática** con llave de respuestas.
- Feedback inmediato:
  - correcta/incorrecta,
  - respuesta correcta,
  - explicación corta (2–4 líneas).
- Registro de intentos y métricas:
  - precisión diaria,
  - precisión semanal,
  - desempeño por tema,
  - racha.

### Personalización
- Distribución de preguntas basada en desempeño:
  - **70%** debilidades personales,
  - **30%** mantenimiento (temas fuertes + núcleo común).

### Competencia (justa)
- Competencia por:
  - **precisión normalizada**,
  - **progreso semanal**,
  - **dominio por tema**,
  - y un **duelo semanal** con el mismo set para ambos.

### Operación semanal
- Generación de un **paquete semanal** de preguntas (en JSON) para reducir costos y mantener consistencia.
- Retro semanal con:
  - Top 3 debilidades por persona,
  - tipo de error (conceptual, procedimiento, cálculo, lectura),
  - plan de enfoque para la semana siguiente.

### Dashboard
- Visualización de:
  - ranking semanal,
  - racha,
  - heatmap por tema,
  - lista de preguntas falladas,
  - tendencias (mejora vs semana pasada).

---

## 4) Usuarios y contexto
- **David:** Ingeniería Mecánica (más foco en mecánica/termo/fluidos/materiales según temario).
- **Kamila:** Ingeniería Mecatrónica (más foco en electrónica/control/embebidos según temario).
- **Núcleo común:** temas que ambos comparten (p.ej., matemáticas, física, instrumentación básica; según el temario final).

El sistema debe permitir:
- preguntas diferentes por persona (personalización),
- y mantener competencia justa (normalización + duelo semanal).

---

## 5) Principios de diseño (no negociables)
- **Aprender con preguntas**, no solo con lectura.
- **Medir por tema** (porque “me siento bien” no es una métrica).
- **Reforzar errores**, no repetir lo que ya se domina.
- **Feedback corto y accionable** (evitar textos largos).
- **Constancia > intensidad** (mejor diario sostenido que maratón ocasional).

---

## 6) Qué NO vamos a hacer (anti-alcance)
Para evitar sobrecomplicar el proyecto, este repositorio NO busca:
- Ser una plataforma multiusuario pública (solo 2 usuarios).
- Reemplazar un curso completo o libros (esto es un sistema de práctica y medición).
- Generar explicaciones tipo “capítulo de libro” en cada respuesta.
- Crear un motor de IA que “califique con criterio humano” (se usa answer key).
- Construir un sistema de proctoring, anti-trampa o vigilancia.
- Soportar 20 canales distintos (inicialmente solo WhatsApp; otros canales son futuros).
- “Cubrir todo el temario perfecto”: el foco es **impacto en puntaje** y mejora medible.

---

## 7) Éxito del proyecto (cómo sabremos que funcionó)
El sistema es exitoso si:
- se usa **6/7 días** por semana de forma sostenida,
- cada persona muestra **mejora semanal** en precisión,
- las debilidades por tema **se reducen** (sube la precisión en Top 3 temas débiles),
- y el “duelo semanal” muestra tendencia positiva.

---

## 8) Roadmap (alto nivel)
- **MVP:** WhatsApp → preguntas → respuestas → scoring → resumen diario.
- **V1:** personalización 70/30 + repetición espaciada + comandos (RESUMEN/DEBILIDADES/RACHA).
- **V2:** generación semanal (paquete JSON) + retro semanal automatizada.
- **V3:** dashboard React (Next.js) con analytics y ranking.
