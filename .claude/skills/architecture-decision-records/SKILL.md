---
name: architecture-decision-records
description: Analiza alternativas de arquitectura y prepara el contenido técnico de un ADR: opciones realistas, trade-offs, matriz de criterios y consecuencias. Úsala cuando el Solution Architect deba proponer una arquitectura.
---

# Análisis para ADR de arquitectura

## Procedimiento

1. **Drivers.** Requisitos funcionales clave (CLAUDE.md §1), atributos de calidad (disponibilidad, seguridad multi-tenant, costo, mantenibilidad), restricciones (stack fijado, free tier, equipo pequeño).
2. **Opciones (≥2, realistas).** Ejemplo SafePet:
   - A. Monolito modular en capas (Api/Application/Domain/Infrastructure) con módulos por bounded context.
   - B. Monolito + worker separado para notificaciones/webhooks.
   - C. Microservicios por contexto.
3. **Para cada opción:** diagrama textual C4 (contexto y contenedores), flujo de una reserva pública, flujo de pago, despliegue, costo free tier, complejidad operativa, impacto en pruebas, reversibilidad.
4. **Matriz.** Criterios ponderados (alinear con Charles). Justifica cada puntuación en una línea.
5. **Riesgos y mitigaciones** por opción.
6. **Fitness functions.** Reglas verificables que protegen la decisión (p. ej. prueba de arquitectura: `Domain` no referencia `Infrastructure`; prueba de aislamiento de tenant en CI).

## Salida

Borrador para `adr-writing` con secciones Contexto, Opciones, Matriz, Recomendación, Consecuencias, Fitness functions.

## Reglas

- No elegir microservicios sin un driver concreto que lo exija (escala independiente, equipos separados).
- Toda afirmación sobre un proveedor o framework con enlace a documentación oficial.

---

Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
