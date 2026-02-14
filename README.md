# CENEVAL Duel System

Sistema ligero y orientado a datos para prepararse para **CENEVAL** usando:
- **Práctica diaria** (quizzes por WhatsApp)
- **Retroalimentación inmediata**
- **Aprendizaje adaptativo** (70/30 debilidades vs mantenimiento)
- **Duelo semanal** (mismo set para todos los participantes)
- **Analítica de progreso** (dashboard en Next.js)

Este repo está pensado para **cualquier número de participantes del grupo**.

Aunque cada persona reciba preguntas distintas por su especialidad, la competencia se mantiene **justa** gracias a métricas normalizadas (precisión + progreso) y un duelo semanal con el mismo set para todos.

---

## Por qué funciona (pedagogía en breve)
CENEVAL se prepara mejor con **práctica activa** (responder preguntas), **feedback rápido** y **repetición espaciada** (volver a ver errores con intervalos). Este sistema implementa eso con una rutina diaria por WhatsApp y un ciclo semanal de revisión, respaldado por una base de datos para medir y ajustar el estudio con evidencia.

Documentación completa:
- `docs/01-pedagogy.md`
- `docs/02-game-rules.md`

---

## Estructura del repositorio

```text
ceneval-duel-system/
  README.md
  docs/
    00-vision.md
    01-pedagogy.md
    02-game-rules.md
    03-architecture.md
    04-data-model.md
    05-runbook.md
    adr/
      0001-tech-stack.md
  apps/
    dashboard/              # Next.js (React)
  automations/
    n8n/
      workflows/            # exports JSON de n8n
      credentials.example.md
  infra/
    supabase/
      migrations/
      seed/
  scripts/
    question-pack-builder/  # genera JSON semanal (opcional)
  .env.example
  .gitignore
