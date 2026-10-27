---
name: safe-refactoring
description: Refactoriza código existente sin cambiar su comportamiento: pruebas de caracterización, pasos mecánicos pequeños, separación de refactor y cambio funcional, y rollback fácil. Úsala en modernización o limpieza de código.
---

# Refactorización segura

## Antes de tocar
1. Identifica el comportamiento actual y quién lo consume (endpoints, pantallas, jobs).
2. Si no hay pruebas, escribe **pruebas de caracterización** que fijen el comportamiento actual (incluidos casos raros) y confirma que pasan antes del cambio.
3. Define el objetivo del refactor en una frase y el criterio de "terminado".

## Durante
- Pasos mecánicos pequeños (renombrar, extraer método/clase, mover, invertir dependencia), cada uno compilando y con pruebas en verde.
- **Nunca** mezclar refactor y cambio de comportamiento en el mismo commit; idealmente en PRs distintos.
- Cambios amplios con patrón *strangler*/*branch by abstraction*: introduce la nueva implementación detrás de una interfaz, migra consumidores de a uno, elimina lo antiguo al final.
- Contratos API/BD: expand → migrate → contract en PRs separados.
- Evitar reformateos masivos que oculten el diff real.

## Después
- Pruebas afectadas + suite de la capa completa si tocaste `Domain`/`Application`.
- Comparar comportamiento (respuestas de API, consultas generadas) antes/después cuando aplique.
- PR con: objetivo, pasos, evidencia de que el comportamiento no cambió, rollback (`git revert` del PR).

## Señales para detenerse
Pruebas que cambian de resultado esperado, necesidad de modificar contratos públicos no previstos, alcance que crece: reporta y propone un issue nuevo.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
