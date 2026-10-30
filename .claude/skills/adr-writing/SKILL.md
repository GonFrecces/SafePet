---
name: adr-writing
description: Redacta y publica un Architecture Decision Record (ADR) en docs/adr con el formato y ciclo de estados del proyecto. Úsala cuando una decisión técnica deba quedar documentada.
---

# Redacción de ADR

## Ubicación y nombre

`docs/adr/NNNN-titulo-en-kebab.md` (NNNN correlativo, 4 dígitos). Índice en `docs/adr/README.md`.

## Estados

`Propuesto` → `Aceptado` (solo por un humano) → `Reemplazado por NNNN` | `Rechazado` | `Obsoleto`.
Un agente nunca marca un ADR como Aceptado.

## Plantilla

```markdown
# NNNN. <Título de la decisión>

- Estado: Propuesto
- Fecha: AAAA-MM-DD
- Decisores: <humano responsable>
- Facilitador: Charles (charles-engineering-manager)
- Participantes: <agentes que aportaron>
- Issue: GON-XX

## Contexto

Problema, fuerzas y restricciones. Hechos con evidencia (enlace/archivo:línea).

## Criterios de decisión

| Criterio | Peso |

## Opciones consideradas

### Opción A — <nombre>

Descripción, pros, contras, costo, riesgos.

### Opción B — ...

## Matriz de evaluación

| Opción | Criterio 1 | ... | Total |

## Decisión

Elegimos <opción> porque <criterios decisivos>.

## Consecuencias

Positivas / negativas / riesgos aceptados / deuda técnica.

## Plan de implementación y rollback

Tareas (GON-XX) y cómo revertir.

## Desacuerdos registrados

Quién, qué, cómo se resolvió.
```

## Reglas

- Un ADR = una decisión. Si hay varias, varios ADR.
- Los ADR no se reescriben tras aceptarse: se reemplazan con uno nuevo.
- Hechos e hipótesis separados explícitamente.

---

Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
