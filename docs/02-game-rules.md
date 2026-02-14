# CENEVAL Duel System — Game Rules (docs/02-game-rules.md)

## 1) Propósito de las reglas
Estas reglas existen para:
- Mantener la competencia **justa** aunque cada persona reciba preguntas distintas (por especialidad).
- Motivar constancia sin volverlo tóxico.
- Convertir el estudio en un juego medible con resultados semanales.

Participantes:
- Miembros del grupo que se unan al sistema.

---

## 2) Cómo se gana (marcadores oficiales)
La competencia tiene **4 marcadores**, en este orden de importancia.

### Marcador 1 — Precisión semanal (ranking principal)
**Precisión** = correctas / respondidas

- Es el ranking oficial de “quién está más sólido esta semana”.
- Es justo aunque las preguntas sean distintas.

**Regla:** solo cuentan preguntas respondidas (no enviadas).

---

### Marcador 2 — Progreso semanal (ranking de mejora)
**Progreso** = precisión de esta semana − precisión de la semana pasada

- Premia al que más mejora.
- Evita frustración si alguien parte más abajo.
- Mantiene motivación constante.

---

### Marcador 3 — Dominio por tema (trofeos por área)
Para cada tema:
- **% de aciertos por tema** (por persona)

Se otorgan “trofeos” por tema:
- 🏆 Gana quien tenga el % más alto en ese tema
- 🤝 Empate si la diferencia con el primer lugar es menor a 3 puntos porcentuales

Objetivo:
- Detectar fortalezas y debilidades específicas.
- Facilitar coaching mutuo.

---

### Marcador 4 — Duelo del sábado (Head-to-Head comparable)
Cada sábado se juega un duelo:
- **40–60 preguntas idénticas** para todos los participantes
- Tiempo recomendado: 60–90 min
- Temas: núcleo común + temas más frecuentes del examen

**Ganador del duelo:** mayor precisión en ese set.

**Desempate (si aplica):**
1) menor tiempo total (si se mide), o
2) mayor puntuación en preguntas de dificultad alta, o
3) empate oficial.

---

## 3) Puntuación y métricas
### Puntuación diaria (simple)
- +1 por respuesta correcta
- 0 por incorrecta
- (Opcional) -0.25 por “adivinar” si quieren elevar dificultad (no recomendado al inicio)

### Métrica de participación (racha)
- Se suma un día de racha si responden al menos el **70% del set diario**.

---

## 4) Reglas del set diario (asignación automática + WhatsApp sin teclear)
- 15–20 preguntas por persona (ajustable).
- El set diario es **asignado automáticamente** por el sistema (70/30 según debilidades), no se elige manualmente.
- Respuestas preferidas:
  - **Interactive List** A/B/C/D (solo tocar una opción, sin escribir).
  - Fallback: texto A/B/C/D si fuera necesario.
- Feedback inmediato por pregunta:
  - correcta/incorrecta,
  - respuesta correcta,
  - explicación corta (2–4 líneas).
- Si una persona no responde en el día, el set se marca como “incompleto” y no cuenta para ranking de precisión (para evitar sesgos).

**Regla de consistencia:** se recomienda responder siempre desde el mismo modo (interactive) para evitar errores de parsing.

---

## 5) Coach Points (opcional, recomendado)
Los coach points premian enseñar y aprender (sin reemplazar los marcadores oficiales).

- Si una persona explica una pregunta y otra demuestra que entendió (la contesta bien al repetirla) → +1 para quien explicó

**Coach Points NO cambian el ranking principal**, pero aparecen en el dashboard como medalla extra.

---

## 6) Anti-trampa (sin drama)
La idea es aprender, no vigilar.

- El duelo del sábado se hace **sin apuntes** (ideal).
- En el set diario, si usan apuntes está permitido, pero deben marcarlo con comando:
  - `APUNTES ON` / `APUNTES OFF`
Esto permite separar:
- práctica auténtica
- práctica asistida

---

## 7) Resultado semanal (lo que se anuncia)
Cada domingo:
- Ranking por precisión (oficial)
- Ranking por progreso (mejora)
- Ganador del duelo del sábado
- Top 3 temas débiles por persona
- Trofeos por tema + coach points
