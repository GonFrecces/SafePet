---
name: agile-delivery
description: Opera Scrum + Kanban en Linear para SafePet: planning, standup con despacho por capas, review/retro, Definition of Ready/Done y límites WIP.
---

# agile-delivery

## Cuándo usarla
Planning del lunes, standup diario, review/retro del viernes y cualquier despacho de issues.

## Procedimiento
1. Lee el ciclo actual del equipo GON. Sin ciclo activo: "Sin sprint activo" y termina.
2. Ojo: Linear pone en Todo todo issue que entra a un ciclo. La señal de despacho es el label `ready`, no el estado.
3. Agrupa los issues del ciclo por MVP (milestone) y por capa: 0 Fundación, 1 BD (Reed), 2 Backend (Bruce), 3 Frontend (Shuri), 4 Seguridad (Natasha), 5 QA (Domino/Vision).
4. Capa liberada = la más baja con issues no Done, siempre que todas las anteriores del MVP estén Done. Issues sin código (investigación, diseño, ADR) no esperan capas.
5. Definition of Ready: 1 capa, 1 PR, ≤ 3 puntos, alcance + exclusiones + criterios dado/cuando/entonces + comando de pruebas, blockedBy en Done. Peter: solo ≤ 1 punto.
6. Despacho: agrega el label `ready` (y mueve a Todo si estaba en Backlog) hasta WIP (1 por agente, 3 en total), por capa más baja, prioridad y número de issues que desbloquea.
7. Comenta cada despacho: `Despachado · Sprint N · MVP Mx · capa X · DoR ok · pruebas: <comando>`.
8. No cumple DoR: label `needs-split` + propuesta de división en comentario. No crees issues sin aprobación humana.

## Plantillas
- **Standup:** Ayer (Done / In Review) · Hoy (despachados) · Bloqueos · Riesgos.
- **Retro:** Qué funcionó · Qué no · Acción para el próximo sprint (una, con responsable).

## Definition of Done
PR con `GON-xx`, pruebas afectadas con salida, revisión del dueño de capa o QA, merge humano. Ningún agente marca Done.
