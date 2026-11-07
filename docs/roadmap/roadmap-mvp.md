# Hoja de ruta MVP · derivada de ADR-0001 (Propuesta)

Estado: **pendiente de aprobación humana**. Depende de que [ADR-0001](../adr/0001-arquitectura-base.md) pase a Aceptado. Charles no crea issues: estas tareas son **propuestas**; Pepper/humano las crea en Linear. Todas ≤ 3 puntos, una capa, un PR. Orden de entrega por capas: BD (Reed) → Backend (Bruce) → Frontend (Shuri) → Seguridad (Natasha) → QA (Domino, Vision).

Los criterios de aceptación están en formato resumido; Domino (GON-39) los reescribe en dado/cuando/entonces.

## Bloqueos de producto antes de empezar (ADR-0001 §8)

Corregido tras GON-39 (B-4). Cada línea coincide con la columna "Depende de" de las tablas.

- **Antes de aceptar el ADR:** T-07 (Sage) verifica HIPÓTESIS-2 y `btree_gist`; D1 y la decisión §8.1 (RLS) dependen de su informe.
- **Decisión §8.2 (boxes):** resuelta por Claudio el 2026-11-07. Ya no bloquea.
- **Decisiones §8.3 (asignación de veterinario al buscar por sucursal) y §8.4 (duración por tipo de atención):** bloquean T-09 y T-16.
- **GON-12 (auth):** T-12, T-13 y T-14 *son* su implementación; empiezan cuando GON-12 se cierre (el ADR recomienda la opción A).
- **GON-29 (canal de recordatorios):** bloquea T-20.
- **T-27 (RLS):** T-28 (primer despliegue) depende de T-27 o de una decisión humana registrada que lo exima.
- **GON-25 (cobros):** ningún cobro entra en este MVP.

## Fase 0 · Higiene y andamiaje

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-01** | Endurecer `.gitignore` y `.claude/settings.json`; crear `.env.example` y `appsettings.Example.json` | Peter | 1 | — |
| **T-02** | Scaffold backend: `SafePet.sln`, 4 proyectos, `global.json` (.NET 10), `Directory.Build.props`, `Directory.Packages.props`, `dotnet-tools.json` | Reed | 2 | ADR aceptado |
| **T-03** | Prueba de arquitectura de capas y módulos (NetArchTest) | Bruce | 1 | T-02 |
| **T-04** | Scaffold frontend: Vite 8 + TS 7 + ESLint + Prettier + Vitest, `primereact@10.9.9`, versiones exactas y lockfile | Shuri | 2 | ADR aceptado |
| **T-05** | Spike FullCalendar 6.1.22 vs 7.1.3 (accesibilidad, estilos, vista `listDay`) y tema Lara con overrides de `DESIGN.md` | Shuri | 2 | T-04 |
| **T-06** | CI mínimo en GitHub Actions: jobs por ruta, PostgreSQL de servicio, `gitleaks`, check de `IgnoreQueryFilters` | Forge | 2 | T-02, T-04 |
| **T-07** | Spike de verificación: pooler de Neon (modo transaction, `SET LOCAL`) y extensión `btree_gist` | Sage | 1 | — (**antes** de aceptar el ADR) |

- **T-01** Alcance: ignorar `appsettings.*.json` salvo ejemplo; denegar lectura de `.env.*` y `appsettings.*.json`; denegar `HEAD:master` y `+rama` en push. Exclusiones: protección de rama en GitHub (acción humana). Criterio: `git check-ignore` confirma los patrones; ningún archivo de ejemplo queda ignorado.
- **T-02** Exclusiones: entidades, migraciones, endpoints. Criterio: `dotnet build -warnaserror` y `dotnet test` verdes en vacío; `Nullable` y `TreatWarningsAsErrors` activos.
- **T-04** Exclusiones: pantallas. Criterio: `npm ci`, `npm run lint`, `npx tsc --noEmit`, `npm test` y `npm run build` verdes. Si TS 7/Vite 8 no resuelven con ESLint/Vitest, se documenta y se fija una combinación probada.
- **T-05** Criterio: informe con decisión, captura a 390 px y resultado de `vitest-axe`.
- **T-06** Criterio: un PR de prueba ejecuta los jobs; sin despliegue automático.
- **T-07** Criterio: informe con fuente oficial y fecha; si `btree_gist` no está disponible, se propone alternativa a D5.

## Fase 1 · Núcleo multi-tenant (capa 1 y 2)

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-08** | Esquema base: Company (tenant, `companySlug`), Branch (`branchSlug`, zona horaria), Box; `TenantId` con índice; primera migración con `Down`; prueba de metadatos | Reed | 3 | T-02 |
| **T-10** | Tenancy: `ITenantContext`, query filter por convención con fallo cerrado, `SaveChangesInterceptor`, FKs compuestas; **prueba cruzada de aislamiento** sobre Branch y Box contra PostgreSQL local | Bruce | 3 | T-08 |
| **T-29** | Esquema de personal: StaffUser, Veterinarian, turnos del veterinario (veterinario × sucursal × box × rango); amplía la prueba de aislamiento | Reed | 3 | T-10 |
| **T-09** | Esquema de Agenda: Appointment (estado, tipo de atención, veterinario, sucursal, box), `EXCLUDE` por veterinario y rango y por box y rango; `Down` probado; amplía la prueba de aislamiento | Reed | 3 | T-29, T-07, decisiones §8.3 y §8.4 |
| **T-11** | Infra de pruebas de integración HTTP (`WebApplicationFactory` + PostgreSQL real + Respawn) y prueba parametrizada de aislamiento por entidad y endpoint | Bruce | 3 | T-10 (se conecta a CI cuando exista T-06) |
| **T-12** | Auth 1/3 (GON-12): login, emisión de JWT ≤ 15 min con claims `tenant_id`, `role`, `branch_ids` | Bruce | 3 | T-29, GON-12 cerrado |
| **T-13** | Auth 2/3: refresh rotado, lockout y políticas `OwnerOnly`/`Clinical`/`FrontDesk` con matriz rol × endpoint | Bruce | 3 | T-12 |
| **T-14** | Auth 3/3: invitación de usuarios y reset de contraseña | Bruce | 3 | T-13 |

- **T-08/T-29/T-09** Exclusiones: datos de clínica (mascotas, fichas), pagos. Criterio: migración `update → update <anterior> → update` probada localmente; script SQL idempotente en el PR; prueba de metadatos: toda entidad `ITenantOwned` tiene filtro e índice. **Excepción temporal (ADR §4.2, B-3):** T-08 no trae prueba cruzada porque no hay query filter ni lectura; llega en T-10. T-09 añade pruebas de 20 reservas concurrentes sobre el mismo veterinario y sobre el mismo box (1 éxito y 19 conflictos cada una) y una prueba de dos veterinarios en el mismo horario y boxes distintos (ambos éxito). Los rangos se guardan en UTC y se muestran en la zona horaria de la sucursal.
- **T-10** Criterio: tenant nulo devuelve 0 filas; alta con `TenantId` ajeno falla; lectura y escritura cruzadas entre dos compañías sobre Branch y Box fallan; `IgnoreQueryFilters` fuera de la lista permitida rompe la prueba (y CI cuando exista).
- **T-11** Criterio: dos tenants sembrados; `GET/PUT/DELETE` cruzado devuelve 404 y la fila ajena queda intacta; una entidad nueva sin cobertura hace fallar la suite.
- **T-12 a T-14** Exclusiones: MFA y proveedores externos. Criterio: sin tenant ni rol editables por el usuario; lockout verificado por prueba.

## Fase 2 · Agenda pública y ficha clínica

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-15** | Casos de uso de disponibilidad: por sucursal y por veterinario dentro de una compañía (endpoints de ADR §4.10) | Bruce | 3 | T-09, T-11 |
| **T-16** | Reserva pública: `Idempotency-Key`, 409 `slot-unavailable`, resolución por `companySlug`, rechazo de ids de otro tenant | Bruce | 3 | T-15, decisión §8.3 |
| **T-17** | Anti-abuso: rate limit por IP y sucursal, reserva `PENDIENTE` con TTL y confirmación, CAPTCHA invisible | Bruce | 3 | T-16 |
| **T-18** | Esquema de ficha clínica central por compañía: Client, Pet, MedicalRecord (sin filtro por sucursal); amplía la prueba de aislamiento | Reed | 3 | T-10 |
| **T-19** | Endpoints de ficha con política por rol (ADMINISTRATIVO sin historial, recetas ni diagnósticos) | Bruce | 3 | T-18, T-13 |
| **T-20** | Outbox transaccional + `HostedService` de recordatorios + endpoint interno para cron externo | Bruce | 3 | T-16, GON-29 cerrado |
| **T-21** | Frontend agenda pública: búsqueda por sucursal o veterinario | Shuri | 3 | T-15, spec de Wanda |
| **T-22** | Frontend agenda pública: flujo de reserva, estados vacío/error, 409 | Shuri | 3 | T-16, T-21 |
| **T-23** | Frontend backoffice: login, guards `<RequireRole>`, calendario por veterinario | Shuri | 3 | T-13, T-05 |
| **T-30** | Esquema de contingencia: `BranchClosure` y estado de cita `REQUIERE_REAGENDAR` fuera de las restricciones `EXCLUDE`; amplía la prueba de aislamiento | Reed | 2 | T-09 |
| **T-31** | Casos de uso de contingencia: cerrar sucursal por rango, lista de citas afectadas, proponer y confirmar reagendamiento con el mismo veterinario en otra sucursal con box libre | Bruce | 3 | T-30, T-16, T-13 |
| **T-32** | Frontend backoffice de contingencia: cierre de sucursal, lista de trabajo para llamar, reagendar | Shuri | 3 | T-31, T-23, spec de Wanda |

- Exclusiones comunes: cobros y Stripe (GON-25), recetas y tratamientos (siguientes MVP).
- **T-15/T-16** Criterio: la API nunca acepta `TenantId` del cliente; slug de A con `serviceId` de B devuelve 404/422; 20 reservas concurrentes dan 1 × 201 y 19 × 409.
- **T-17** Criterio: exceder el límite devuelve 429; reserva sin confirmar expira.
- **T-18/T-19** Criterio (CR-3): un veterinario de la sucursal 2 lee la ficha creada en la sucursal 1 de la misma compañía; otra compañía recibe 404.
- **T-20** Criterio: un fallo al guardar la cita no genera recordatorio; procesar el outbox de A no toca datos de B.
- **T-30/T-31** Criterio (CR-6, ADR §4.11): cerrar la sucursal S de 10:00 a 14:00 marca solo las citas de S en ese rango; ninguna cita cambia de sucursal sin confirmación de `OWNER` o `ADMINISTRATIVO`; la propuesta solo ofrece horas del mismo veterinario en sucursales de la misma compañía con box libre; un usuario de otra compañía recibe 404; repetir el cierre no duplica la lista.
- **T-32** Criterio: validado a 390 px y 1440 px; `vitest-axe` sin violaciones; `PROFESIONAL` no ve la acción de cerrar sucursal.
- **T-21 a T-23** Criterio: validado a 390 px y 1440 px; `vitest-axe` sin violaciones; el frontend no envía `tenantId` libre; el cambio de sucursal cambia la `queryKey`.

## Fase 3 · Verificación y puertas

| ID | Tarea | Agente | Pts | Depende de |
|---|---|---|---|---|
| **T-24** | Revisión de seguridad del MVP (capa 4): aislamiento, agenda pública, contingencia, secretos | Natasha | 2 | T-15 a T-23, T-31, T-32 |
| **T-25** | QA del MVP: criterios, regresión, matriz criterio → prueba (capa 5) | Domino | 2 | T-24 |
| **T-26** | Auditoría de dependencias y cobertura; pasar `CLAUDE.md` §2.2 y §3 de **[Propuesto]** a **[Hecho]** según lo construido (los requisitos CR-1 a CR-6 ya están en `CLAUDE.md`) | Vision | 2 | T-25 |
| **T-27** | RLS de PostgreSQL (M4) como migración aditiva, con rol sin `BYPASSRLS` y `SET LOCAL` por transacción | Reed | 3 | T-11, T-07; **antes del primer dato clínico real** |
| **T-28** | Despliegue a staging (GON-31): Render, Neon, Cloudflare Pages | Forge | 2 | T-06, T-25, T-27 o decisión humana registrada; **autorización humana explícita** |

- Exclusiones: ningún despliegue, migración en staging/producción ni recurso de Stripe live sin autorización humana (`CLAUDE.md` §8).
- **T-27** Criterio: la prueba de aislamiento de T-11 pasa también con una consulta cruda sin tenant fijado (0 filas).
- **T-28** Criterio: plan de rollback documentado y branch de Neon previo a cada migración.

## Resumen

32 tareas, 81 puntos (T-29 a T-32 se agregaron tras GON-39: separar el esquema de personal de T-08 y la contingencia CR-6). Ruta crítica: T-02 → T-08 → T-10 → T-29 → T-09 → T-15 (con T-11) → T-16 → T-22. Los ids no se renumeran para no romper referencias. Las tareas que superaban 3 puntos en los informes (auth 5-8, tenancy, infra de pruebas) ya están divididas. El orden de capas se respeta salvo T-03, T-05 y T-06, que son transversales.
