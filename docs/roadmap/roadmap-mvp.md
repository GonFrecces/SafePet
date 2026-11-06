# Hoja de ruta MVP · derivada de ADR-0001 (Propuesta)

Estado: **pendiente de aprobación humana**. Depende de que [ADR-0001](../adr/0001-arquitectura-base.md) pase a Aceptado. Charles no crea issues: estas tareas son **propuestas**; Pepper/humano las crea en Linear. Todas ≤ 3 puntos, una capa, un PR. Orden de entrega por capas: BD (Reed) → Backend (Bruce) → Frontend (Shuri) → Seguridad (Natasha) → QA (Domino, Vision).

Los criterios de aceptación están en formato resumido; Domino (GON-39) los reescribe en dado/cuando/entonces.

## Bloqueos de producto antes de empezar (ADR-0001 §8)

- Tareas T-09, T-10 y T-12 dependen de las decisiones abiertas 2, 3 y 4 (boxes, asignación de veterinario, duración de citas).
- T-14 depende de GON-12; T-17 de GON-29; ningún cobro (GON-25) entra en este MVP.

## Fase 0 · Higiene y andamiaje

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-01** | Endurecer `.gitignore` y `.claude/settings.json`; crear `.env.example` y `appsettings.Example.json` | Peter | 1 | — |
| **T-02** | Scaffold backend: `SafePet.sln`, 4 proyectos, `global.json` (.NET 10), `Directory.Build.props`, `Directory.Packages.props`, `dotnet-tools.json` | Reed | 2 | ADR aceptado |
| **T-03** | Prueba de arquitectura de capas y módulos (NetArchTest) | Bruce | 1 | T-02 |
| **T-04** | Scaffold frontend: Vite 8 + TS 7 + ESLint + Prettier + Vitest, `primereact@10.9.9`, versiones exactas y lockfile | Shuri | 2 | ADR aceptado |
| **T-05** | Spike FullCalendar 6.1.22 vs 7.1.3 (accesibilidad, estilos, vista `listDay`) y tema Lara con overrides de `DESIGN.md` | Shuri | 2 | T-04 |
| **T-06** | CI mínimo en GitHub Actions: jobs por ruta, PostgreSQL de servicio, `gitleaks`, check de `IgnoreQueryFilters` | Forge | 2 | T-02, T-04 |
| **T-07** | Spike de verificación: pooler de Neon (modo transaction) y extensión `btree_gist` | Sage | 1 | ADR aceptado |

- **T-01** Alcance: ignorar `appsettings.*.json` salvo ejemplo; denegar lectura de `.env.*` y `appsettings.*.json`; denegar `HEAD:master` y `+rama` en push. Exclusiones: protección de rama en GitHub (acción humana). Criterio: `git check-ignore` confirma los patrones; ningún archivo de ejemplo queda ignorado.
- **T-02** Exclusiones: entidades, migraciones, endpoints. Criterio: `dotnet build -warnaserror` y `dotnet test` verdes en vacío; `Nullable` y `TreatWarningsAsErrors` activos.
- **T-04** Exclusiones: pantallas. Criterio: `npm ci`, `npm run lint`, `npx tsc --noEmit`, `npm test` y `npm run build` verdes. Si TS 7/Vite 8 no resuelven con ESLint/Vitest, se documenta y se fija una combinación probada.
- **T-05** Criterio: informe con decisión, captura a 390 px y resultado de `vitest-axe`.
- **T-06** Criterio: un PR de prueba ejecuta los jobs; sin despliegue automático.
- **T-07** Criterio: informe con fuente oficial y fecha; si `btree_gist` no está disponible, se propone alternativa a D5.

## Fase 1 · Núcleo multi-tenant (capa 1 y 2)

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-08** | Esquema base: Company (tenant), Branch, StaffUser, Veterinarian, turnos; `TenantId` con índice; primera migración con `Down` | Reed | 3 | T-02 |
| **T-09** | Esquema de Agenda: Appointment (estado, tipo de atención), disponibilidad por veterinario y restricción `EXCLUDE` por veterinario y rango; `Down` probado | Reed | 3 | T-08, T-07, decisiones 2-4 |
| **T-10** | Tenancy: `ITenantContext`, query filter por convención con fallo cerrado, `SaveChangesInterceptor`, FKs compuestas | Bruce | 3 | T-08 |
| **T-11** | Infra de pruebas de integración (`WebApplicationFactory` + PostgreSQL real + Respawn) y prueba parametrizada de aislamiento por entidad | Bruce | 3 | T-10, T-06 |
| **T-12** | Auth 1/3 (GON-12): login, emisión de JWT ≤ 15 min con claims `tenant_id`, `role`, `branch_ids` | Bruce | 3 | T-10 |
| **T-13** | Auth 2/3: refresh rotado, lockout y políticas `OwnerOnly`/`Clinical`/`FrontDesk` con matriz rol × endpoint | Bruce | 3 | T-12 |
| **T-14** | Auth 3/3: invitación de usuarios y reset de contraseña | Bruce | 3 | T-13 |

- **T-08/T-09** Exclusiones: datos de clínica (mascotas, fichas), pagos. Criterio: migración `update → update <anterior> → update` probada localmente; script SQL idempotente en el PR; prueba de metadatos: toda entidad `ITenantOwned` tiene filtro e índice. T-09 añade una prueba de 20 reservas concurrentes sobre el mismo veterinario y horario: exactamente 1 éxito y 19 conflictos.
- **T-10** Criterio: tenant nulo devuelve 0 filas; alta con `TenantId` ajeno falla; `IgnoreQueryFilters` fuera de la lista permitida rompe CI.
- **T-11** Criterio: dos tenants sembrados; `GET/PUT/DELETE` cruzado devuelve 404 y la fila ajena queda intacta; una entidad nueva sin cobertura hace fallar la suite.
- **T-12 a T-14** Exclusiones: MFA y proveedores externos. Criterio: sin tenant ni rol editables por el usuario; lockout verificado por prueba.

## Fase 2 · Agenda pública y ficha clínica

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-15** | Casos de uso de disponibilidad: por sucursal y por veterinario dentro de una compañía (`/api/v1/public/...`) | Bruce | 3 | T-09, T-11 |
| **T-16** | Reserva pública: `Idempotency-Key`, 409 `slot-unavailable`, resolución por `slug`, rechazo de ids de otro tenant | Bruce | 3 | T-15 |
| **T-17** | Anti-abuso: rate limit por IP y sucursal, reserva `PENDIENTE` con TTL y confirmación, CAPTCHA invisible | Bruce | 3 | T-16 |
| **T-18** | Esquema de ficha clínica central por compañía: Client, Pet, MedicalRecord (sin filtro por sucursal) | Reed | 3 | T-08 |
| **T-19** | Endpoints de ficha con política por rol (ADMINISTRATIVO sin historial, recetas ni diagnósticos) | Bruce | 3 | T-18, T-13 |
| **T-20** | Outbox transaccional + `HostedService` de recordatorios + endpoint interno para cron externo (GON-29) | Bruce | 3 | T-16 |
| **T-21** | Frontend agenda pública: búsqueda por sucursal o veterinario | Shuri | 3 | T-15, spec de Wanda |
| **T-22** | Frontend agenda pública: flujo de reserva, estados vacío/error, 409 | Shuri | 3 | T-16, T-21 |
| **T-23** | Frontend backoffice: login, guards `<RequireRole>`, calendario por veterinario | Shuri | 3 | T-13, T-05 |

- Exclusiones comunes: cobros y Stripe (GON-25), recetas y tratamientos (siguientes MVP).
- **T-15/T-16** Criterio: la API nunca acepta `TenantId` del cliente; slug de A con `serviceId` de B devuelve 404/422; 20 reservas concurrentes dan 1 × 201 y 19 × 409.
- **T-17** Criterio: exceder el límite devuelve 429; reserva sin confirmar expira.
- **T-18/T-19** Criterio (CR-3): un veterinario de la sucursal 2 lee la ficha creada en la sucursal 1 de la misma compañía; otra compañía recibe 404.
- **T-20** Criterio: un fallo al guardar la cita no genera recordatorio; procesar el outbox de A no toca datos de B.
- **T-21 a T-23** Criterio: validado a 390 px y 1440 px; `vitest-axe` sin violaciones; el frontend no envía `tenantId` libre; el cambio de sucursal cambia la `queryKey`.

## Fase 3 · Verificación y puertas

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-24** | Revisión de seguridad del MVP (capa 4): aislamiento, agenda pública, secretos | Natasha | 2 | T-15 a T-23 |
| **T-25** | QA del MVP: criterios, regresión, matriz criterio → prueba (capa 5) | Domino | 2 | T-24 |
| **T-26** | Auditoría de dependencias y cobertura; actualizar `CLAUDE.md` (§1, §2.2, §3, §2.4) de **[Propuesto]** a **[Hecho]** y por CR-1 a CR-3 | Vision | 2 | T-25 |
| **T-27** | RLS de PostgreSQL (M4) como migración aditiva, con rol sin `BYPASSRLS` y `SET LOCAL` por transacción | Reed | 3 | T-11, T-07; **antes del primer dato clínico real** |
| **T-28** | Despliegue a staging (GON-31): Render, Neon, Cloudflare Pages | Forge | 2 | T-06, T-25; **autorización humana explícita** |

- Exclusiones: ningún despliegue, migración en staging/producción ni recurso de Stripe live sin autorización humana (`CLAUDE.md` §8).
- **T-27** Criterio: la prueba de aislamiento de T-11 pasa también con una consulta cruda sin tenant fijado (0 filas).
- **T-28** Criterio: plan de rollback documentado y branch de Neon previo a cada migración.

## Resumen

28 tareas, 70 puntos. Ruta crítica: T-02 → T-08 → T-10 → T-11 → T-12/T-13 → T-15 → T-16 → T-22. Las tareas que superaban 3 puntos en los informes (auth 5-8, tenancy, infra de pruebas) ya están divididas. El orden de capas se respeta salvo T-03, T-05 y T-06, que son transversales.
