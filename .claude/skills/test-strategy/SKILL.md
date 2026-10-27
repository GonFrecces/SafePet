---
name: test-strategy
description: Define la estrategia de pruebas de un cambio o proyecto: niveles, qué cubre cada uno, datos, entorno y criterios de salida, incluida la prueba de aislamiento multi-tenant. Úsala al planificar trabajo o revisar propuestas.
---

# Estrategia de pruebas

## Pirámide SafePet
| Nivel | Herramienta | Cubre |
|---|---|---|
| Unitario dominio | xUnit | Invariantes: estados de cita, tipos de atención, cálculo de montos |
| Unitario aplicación | xUnit + fakes | Casos de uso, autorización de caso de uso |
| Integración API | WebApplicationFactory + PostgreSQL | Endpoints, roles, aislamiento tenant, concurrencia, webhooks |
| Migraciones | EF Core + PostgreSQL local | `Up`, `Down`, `Up` |
| Front unitario/componentes | Vitest + Testing Library + MSW | Pantallas, formularios, guards |
| E2E (selectivo) | Playwright | Reserva pública completa, login + abrir/cerrar cita |
| Arquitectura | NetArchTest | Dependencias entre capas |

## Para cada cambio
1. Lista los criterios de aceptación.
2. Matriz `criterio → prueba(s) → nivel → archivo`.
3. Identifica riesgos sin prueba y decide: agregar o aceptar explícitamente.
4. Define datos (builders, nunca reales) y entorno (PostgreSQL local/servicio CI).
5. Criterios de salida: suites afectadas en verde, sin pruebas omitidas nuevas, prueba de aislamiento si toca datos de clínica.

## Reglas
CLAUDE.md §7: ejecutar las afectadas y reportar comando + resultado. Prohibido desactivar pruebas.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
