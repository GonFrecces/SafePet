---
name: project-planning
description: Convierte una decisión aprobada en hoja de ruta y tareas pequeñas, ordenadas por dependencia, con alcance, exclusiones, criterios de aceptación y agente responsable. Úsala tras un ADR o al planificar un milestone.
---

# Planificación de proyecto

## Procedimiento
1. **Entradas.** ADR (estado), issues existentes en Linear (evitar duplicados), milestones.
2. **Descomposición.** Divide en tareas que un agente pueda terminar y revisar en un PR (≈ ≤ 400 líneas de diff, un objetivo).
3. **Dependencias.** Construye un DAG: qué bloquea a qué. Identifica la ruta crítica.
4. **Orden.** Primero fundaciones y riesgo alto (multi-tenant, auth, datos), luego features, luego pulido.
5. **Asignación.** Un label `agent:*` por tarea según `docs/equipo-agentes.md`.
6. **Riesgos.** Por tarea: riesgo principal + mitigación + rollback.

## Plantilla de tarea
```
Título: <verbo> <objeto> (<capa>)
Contexto: por qué existe; referencia al ADR.
Alcance: viñetas concretas.
Exclusiones: lo que NO entra.
Criterios de aceptación: dado/cuando/entonces, verificables.
Pruebas: qué suite y qué casos (incl. aislamiento multi-tenant si aplica).
Rollback: cómo revertir.
Depende de: GON-XX. Agente: agent:<nombre>.
```

## Hoja de ruta
Tabla `milestone | objetivo | tareas | criterio de salida`. Sin fechas comprometidas si el equipo no las validó: usa orden relativo.

## Reglas
- Las tareas quedan *pendientes de aprobación*; no se marcan como listas para ejecutar.
- No crear tareas que mezclen refactor y cambio de comportamiento.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
