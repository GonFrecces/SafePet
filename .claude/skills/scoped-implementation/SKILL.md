---
name: scoped-implementation
description: Implementa estrictamente dentro del alcance de un issue aprobado: precondiciones, pasos pequeños, sin trabajo extra, PR con evidencia. Úsala siempre que vayas a escribir código para un issue.
---

# Implementación acotada

## Precondiciones (si falta una, detente y repórtalo)
- Issue con alcance, exclusiones y criterios de aceptación.
- Decisión/ADR referenciado en estado **Aceptado** (para Modernization Developer).
- Rama asignada; árbol limpio (`git status`).

## Procedimiento
1. Relee alcance y exclusiones; escribe un plan de 3–7 pasos y los archivos que tocarás.
2. Por cada paso: prueba → cambio mínimo → pruebas afectadas → commit (Conventional Commits, en español).
3. Si descubres trabajo necesario fuera de alcance: **no lo hagas**; anótalo como propuesta de issue nuevo en el PR.
4. Antes del PR: auto-revisión del diff (`git diff origin/master...HEAD`): ¿todo cambio se justifica por un criterio? ¿hay cambios de formato masivos o archivos ajenos?
5. Solicita revisión a `natasha-security-engineer` (labels `multi-tenant`/`Seguridad`) y a `domino-qa-automation`.

## Plantilla de PR
```
## Alcance
## Exclusiones
## Criterios de aceptación (checklist con prueba asociada)
## Pruebas ejecutadas (comando + resultado)
## Riesgos
## Rollback
## Trabajo fuera de alcance detectado (propuestas)
```

## Prohibido
Merge, despliegue, migraciones fuera de BD local, secretos, desactivar pruebas, actualizar dependencias no pedidas.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
