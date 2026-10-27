---
name: technical-decision-facilitation
description: Facilita una decisión técnica con varios especialistas: preguntas por rol, informes independientes, matriz de criterios ponderados y resolución de desacuerdos. Úsala al abrir o conducir un issue de decisión (p. ej. Architecture Decision Meeting).
---

# Facilitación de decisiones técnicas

## Objetivo
Llegar a una decisión justificada por criterios y evidencia, no por mayoría ni por la opinión más insistente.

## Procedimiento
1. **Encuadre.** Escribe en una frase la pregunta a decidir, el alcance y lo que queda fuera. Lista restricciones duras (CLAUDE.md, presupuesto free tier, plazo, equipo).
2. **Criterios.** Define 4–7 criterios con peso (suma 100). Base SafePet: mantenibilidad, complejidad operativa, costo, riesgo multi-tenant/seguridad, esfuerzo, reversibilidad, experiencia de desarrollo.
3. **Preguntas por especialista.** Una lista concreta por rol; cada pregunta debe poder responderse con evidencia.
4. **Informes independientes.** Encarga los informes sin compartir los de otros. Exige formato: hechos (con evidencia) / hipótesis / opciones / riesgos / recomendación.
5. **Verificación de participación.** Revisa el historial: quién respondió, quién no, y si algún informe es genérico o sin evidencia. Repregunta o decláralo.
6. **Cruce.** Tabla `afirmación | agente | evidencia | coincide/contradice`. Marca contradicciones.
7. **Resolución de desacuerdos.** Para cada desacuerdo: identifica el criterio en disputa, busca el dato que lo zanjaría, pídelo; si no existe, decide por el criterio de mayor peso y regístralo como riesgo aceptado.
8. **Puntuación.** Matriz `opción × criterio` (1–5) × peso. Revisa sensibilidad: ¿cambia el ganador si un peso varía ±10?
9. **Revisión QA.** Envía síntesis a QA antes de publicar.
10. **Cierre.** Publica con `adr-writing`; el ADR queda *Propuesto* hasta aprobación humana.

## Antipatrones
- Votar. Promediar opiniones sin evidencia.
- Aceptar "es mejor práctica" sin fuente.
- Ocultar desacuerdos no resueltos.

## Salida
Plan de reunión, matriz de criterios, registro de desacuerdos y resolución, registro de participación.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
