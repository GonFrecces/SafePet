# ADR-0001 · Arquitectura base de SafePet

| Campo | Valor |
|---|---|
| Estado | **Propuesto** (nada se aplica hasta que un humano lo marque Aceptado) |
| Fecha | 2026-11-05 |
| Facilitador | Charles · Engineering Manager (GON-33) |
| Informes de entrada | GON-32 (Vision), GON-34 (Tony), GON-35 (Bruce), GON-36 (Shuri), GON-37 (Strange), GON-38 (Natasha) |
| Revisión QA | GON-39 (Domino): `requiere cambios` (B-1 a B-5). Correcciones aplicadas el 2026-11-07 (sección 10); re-revisión pendiente |
| Hoja de ruta | [`docs/roadmap/roadmap-mvp.md`](../roadmap/roadmap-mvp.md) |

Convención: **HECHO** = con evidencia citada en un informe; **HIPÓTESIS** = sin verificar. Las puntuaciones de la matriz son juicio de Charles sobre la evidencia de los informes, no mediciones.

---

## 1. Contexto

- **HECHO (GON-32, confirmado por Claudio):** el repositorio no tiene código, dependencias, pruebas, CI ni infraestructura. Solo existe `GonFrecces/SafePet`; no hay repositorio "antiguo". La decisión parte de cero.
- **HECHO:** stack declarado en `CLAUDE.md` §2.1: React + PrimeReact + FullCalendar, C#/ASP.NET Core en capas, PostgreSQL (Neon o Supabase), Stripe.
- SafePet es multi-tenant, maneja datos clínicos, tiene una agenda pública anónima y lo construye un equipo pequeño de agentes con revisión humana, con objetivo de costo $0 en desarrollo.
- Decisiones relacionadas que **este ADR no cierra**: autenticación (GON-12, se recomienda aquí), modelo de cobro Stripe (GON-25), canal de recordatorios (GON-29), despliegue (GON-31).

## 2. Cambios de requisito (respuestas humanas que prevalecen sobre `CLAUDE.md`)

Claudio respondió en los issues de informe y en la revisión de GON-39 (2026-11-07). Estas respuestas son requisitos de producto y **modifican `CLAUDE.md`**. `CLAUDE.md` §1 y §2.4 ya se actualizaron en el mismo PR que corrige este ADR (B-1), antes de cualquier tarea de esquema; la actualización es la ratificación humana del cambio de requisito.

| # | Requisito nuevo (fuente) | Contradice | Efecto |
|---|---|---|---|
| **CR-1** | **La disponibilidad se gestiona por veterinario y por sucursal**: una sucursal puede cubrir 1 o 2 boxes por turnos, y otra tener veterinarios con contrato completo. Una cita reserva veterinario + sucursal + box + rango (GON-34, resp. 1; ratificado por Claudio el 2026-11-07). | `CLAUDE.md` §1 anterior: "disponibilidad de la clínica/sucursal (no del veterinario)". Ya corregido. | Cambia el modelo de Agenda: turnos del veterinario por sucursal y box, y dos restricciones anti-doble-reserva (veterinario y box). Ver D4 y D5. |
| **CR-2** | **El cliente busca una hora por sucursal o por veterinario** y ve dónde hay horas o quién está disponible en qué sucursal de la misma compañía (GON-35, resp.). | `CLAUDE.md` §1 (agenda solo por sucursal). | Dos consultas públicas: por sucursal y por veterinario, ambas dentro de una compañía. |
| **CR-3** | **La ficha clínica es central por compañía**: cada compañía tiene N sucursales; la ficha pertenece al cliente dentro de una sola compañía y es plenamente accesible desde cualquier sucursal y veterinario de esa compañía (GON-34, resp. 3). | Precisa `CLAUDE.md` §1 ("ficha a nivel de clínica"): el tenant es la **compañía**, no la sucursal. | La ficha no se filtra por `BranchId`, solo por `TenantId`. Un cliente en dos compañías tiene dos fichas separadas (**supuesto**, pendiente de confirmación en §8.7). |
| **CR-4** | **No hay cliente real hoy** y RLS no es un requisito conocido por producto (GON-34, resp. 2). | Ninguna; es un dato para D1. | Respalda diferir RLS a fase 2, con revisión antes del primer dato clínico real. |
| **CR-6** | **Contingencia de sucursal**: una sucursal puede quedar no disponible por un periodo (desastre, evento). El personal llama a los clientes con hora tomada o los **reagenda con el mismo veterinario** en otra sucursal de la compañía con box disponible. El sistema debe ser dinámico ante estos eventos (Claudio, 2026-11-07). | Ninguna; requisito nuevo. | Nueva entidad de cierre de sucursal, estado de cita `REQUIERE_REAGENDAR` y caso de uso de reagendamiento (sección 4.11; T-30 a T-32). |
| **CR-5** | Frontend: fijar `primereact@10.9.9` (compatible con `DESIGN.md`); FullCalendar: "el más accesible y con estilos"; TypeScript 7 / Vite 8 aceptados (GON-36, resp. 1-3). | Ninguna; cierra preguntas abiertas. | Ver D6. La versión de FullCalendar se decide con spike de accesibilidad. |

## 3. Opciones consideradas

Las matrices usan los pesos fijados antes de los informes (GON-33, Paso 1): Seguridad/aislamiento 25 %, Costo $0 20 %, Simplicidad con agentes 20 %, Testabilidad 15 %, Reversibilidad 10 %, Rendimiento 10 %. Escala 1 (peor) a 5 (mejor).

### 3.1 Estilo de arquitectura

| Criterio (peso) | A. Monolito modular | B. Microservicios | C. Capas planas |
|---|---|---|---|
| Seguridad/aislamiento (25) | 4 | 2 | 3 |
| Costo $0 (20) | 5 | 1 | 5 |
| Simplicidad (20) | 4 | 1 | 5 |
| Testabilidad (15) | 4 | 3 | 3 |
| Reversibilidad (10) | 4 | 2 | 3 |
| Rendimiento (10) | 4 | 3 | 4 |
| **Total** | **4,20** | 1,85 | 3,90 |

Cálculo A: 1,00 + 1,00 + 0,80 + 0,60 + 0,40 + 0,40 = 4,20. Evidencia: GON-34 §3 (la matriz de Tony usa otros criterios; se re-puntuó con los pesos de GON-33 y el orden no cambia).

### 3.2 Estrategia multi-tenant

| Criterio (peso) | M1. `TenantId` + query filters + interceptor | M2. Schema por tenant | M3. Solo RLS | M4. M1 + RLS |
|---|---|---|---|---|
| Seguridad/aislamiento (25) | 3 | 5 | 4 | 5 |
| Costo $0 (20) | 5 | 4 | 5 | 5 |
| Simplicidad (20) | 5 | 2 | 2 | 3 |
| Testabilidad (15) | 4 | 3 | 3 | 4 |
| Reversibilidad (10) | 5 | 2 | 3 | 4 |
| Rendimiento (10) | 5 | 3 | 4 | 4 |
| **Total** | **4,35** | 3,40 | 3,55 | 4,25 |

Evidencia: GON-34 §5 (pooler en modo transaction penaliza M3 y M4, HIPÓTESIS-2), GON-38 §2 (puntuación propia: M1 4,35 / M4 4,25 / M2 3,40, coincide con la mía). M1 con sus controles obligatorios (sección 4) sube su puntaje de seguridad en la práctica; el margen sobre M4 es de 0,10 y se trata como **estrecho** (ver D1).

### 3.3 Autenticación del backoffice (insumo para GON-12)

| Criterio (peso) | A. ASP.NET Identity + JWT propio | B. Supabase Auth | C. Proveedor externo |
|---|---|---|---|
| Seguridad/aislamiento (25) | 4 | 3 | 4 |
| Costo $0 (20) | 5 | 5 | 3 |
| Simplicidad (20) | 2 | 4 | 3 |
| Testabilidad (15) | 5 | 3 | 3 |
| Reversibilidad (10) | 4 | 2 | 2 |
| Rendimiento (10) | 5 | 4 | 4 |
| **Total** | **4,05** | 3,60 | 3,25 |

Evidencia: GON-35 §5 (esfuerzo A 5-8 pts, B/C 3-5 pts; riesgo de claims editables en B), GON-38 §2 (tres opciones viables con JWT ≤ 15 min, claim emitido por servidor), GON-37 (Supabase como BD descartada, lo que deja a B sin base).

### 3.4 Hosting y base de datos

| Criterio (peso) | Render + Neon + Cloudflare Pages | Fly.io + Neon + Cloudflare | Render + Supabase + Cloudflare |
|---|---|---|---|
| Seguridad/aislamiento (25) | 3 | 3 | 3 |
| Costo $0 (20) | 4 | 2 | 3 |
| Simplicidad (20) | 4 | 3 | 3 |
| Testabilidad (15) | 4 | 4 | 3 |
| Reversibilidad (10) | 4 | 4 | 3 |
| Rendimiento (10) | 3 | 4 | 3 |
| **Total** | **3,65** | 3,15 | 3,00 |

Puntuaciones tomadas de GON-37 §9 (juicio del autor). Las cifras de free tiers provienen de resúmenes de búsqueda y de una comparativa de terceros, **no** de las páginas oficiales completas: deben reverificarse antes de contratar.

## 4. Decisión propuesta

1. **Monolito modular** (A): un despliegue, los 4 proyectos de `CLAUDE.md` §2.2 y 5 módulos con límites verificados por pruebas de arquitectura: Staff y Sucursales, Agenda, Clínica/Ficha, Pagos, Notificaciones. Referencias entre módulos solo por Id; eventos de dominio persistidos en un outbox transaccional.
2. **Multi-tenancy M1 ahora, M4 (RLS) en fase 2 antes de cargar datos clínicos reales en producción.** Controles obligatorios (los de Tony §5 y los 9 requisitos mínimos de Natasha, tratados como **bloqueantes**):
   - Tenant solo del servidor: claim del JWT o `slug` público de **compañía** (sección 4.10); el cliente nunca envía `TenantId`.
   - Query filter por convención sobre toda entidad `ITenantOwned`, con fallo cerrado (tenant nulo = 0 filas).
   - `SaveChangesInterceptor`: estampa `TenantId` en altas y rechaza modificar entidades de otro tenant.
   - FKs compuestas `(TenantId, Id)`.
   - `IgnoreQueryFilters`, `FromSql*`/`ExecuteSql` y `ExecuteUpdate/Delete` fuera de una lista permitida revisada: **falla CI**.
   - Recursos de otro tenant responden **404**.
   - Prueba de aislamiento parametrizada por cada entidad con `TenantId`, contra PostgreSQL real. Cada tarea que agrega entidades la amplía en su mismo PR. **Excepción temporal registrada (B-3):** T-08 crea solo `Company`, `Branch` y `Box`, sin consultas ni endpoints, y lleva únicamente la prueba de metadatos (filtro e índice de `TenantId`). La prueba cruzada de lectura y escritura llega en T-10 (dueño: Bruce), antes de cualquier otra tarea de esquema.
   - Agenda pública: rate limit, anti-bot y reserva `PENDIENTE` con TTL hasta confirmar.
   - Webhooks Stripe con firma verificada sobre body crudo e idempotencia por `event.id`; sin PII en logs; secretos por entorno con secret scanning.
3. **Stack backend:** .NET 10 LTS + EF Core 10 + Npgsql 10, fijado con `global.json`, `Directory.Build.props` y Central Package Management (HECHO, GON-35: .NET 8/9 vencen el 2026-11-10). Pruebas de integración con PostgreSQL real; prohibido InMemory/SQLite.
4. **Auth:** ASP.NET Identity + JWT propio (A), con JWT ≤ 15 min, refresh rotado y claims `tenant_id`, `role`, `branch_ids` emitidos por el servidor. La decisión final queda en GON-12.
5. **Anti-doble-reserva:** restricción en la BD, no en código. Con CR-1 y duración variable por tipo de atención se proponen dos `EXCLUDE USING gist` (`btree_gist`): por **veterinario** y rango, y por **box** y rango. Así un veterinario no atiende dos citas a la vez y un box no se usa dos veces, pero dos veterinarios distintos sí reservan el mismo horario en boxes distintos. Las citas `CANCELADA` y `REQUIERE_REAGENDAR` quedan fuera de ambas restricciones (predicado `WHERE`). Se mapean `23P01`/`23505` a **409** `slot-unavailable`. Medido por Bruce: la verificación-luego-inserción en READ COMMITTED duplicó reservas (3 filas en 2 de 5 corridas). Disponibilidad de `btree_gist` en Neon: HIPÓTESIS a verificar.
6. **Contrato API:** `/api/v1`, OpenAPI, ProblemDetails (RFC 9457), `Idempotency-Key` en reservas y pagos, fechas UTC, superficies separadas (`public`, backoffice, `webhooks/stripe`).
7. **Frontend:** una SPA Vite + TypeScript con rutas públicas (`/c/:companySlug`, `/c/:companySlug/s/:branchSlug`, `/c/:companySlug/v/:vetId`; ver 4.10) y de backoffice (`/app/*`) con `lazy()`; React Router 7, TanStack Query, react-hook-form + zod; **`primereact@10.9.9`** con tema Lara y overrides de `DESIGN.md`; Vitest + Testing Library + MSW + `vitest-axe`. Versiones exactas con lockfile. FullCalendar 6.1.22 vs 7.1.3 se decide con un spike de accesibilidad y estilos (CR-5).
8. **Hosting:** Render (Docker) + Neon + Cloudflare Pages. Free solo para dev/staging; **producción en instancia de pago mínima** (≈ USD 12/mes, HIPÓTESIS) porque el plan Free duerme el servicio y los recordatorios no corren (ver D3). GitHub Actions con jobs por ruta y PostgreSQL de servicio. Vercel Hobby, Fly.io y Supabase como BD quedan descartados.
9. **Pagos:** se registra la recomendación de Natasha (Stripe Connect por clínica) como **insumo**, sin decidir; el cierre es GON-25.
10. **Agenda pública: identificadores y endpoints (B-2).**
    - `companySlug`: único global, identifica al tenant. `branchSlug`: único dentro de la compañía. `vetId`: UUID opaco, nunca secuencial.
    - El tenant se resuelve **solo** desde `companySlug`. Sucursal, veterinario y box se buscan dentro de esa compañía; un id de otra compañía responde 404, con el mismo cuerpo que un slug inexistente.

    | Endpoint (`/api/v1/public`) | Devuelve | No devuelve |
    |---|---|---|
    | `GET /c/{companySlug}` | Nombre público y sucursales activas de esa compañía (slug, nombre, dirección, zona horaria) | Otras compañías, personal, ids internos |
    | `GET /c/{companySlug}/s/{branchSlug}/availability?from&to` | Horas libres por veterinario en esa sucursal | Citas de terceros, datos de clientes |
    | `GET /c/{companySlug}/veterinarians` | Veterinarios con agenda pública de esa compañía (vetId, nombre visible) | Correo, teléfono, turnos internos |
    | `GET /c/{companySlug}/v/{vetId}/availability?from&to` | Horas libres de ese veterinario en las sucursales de la compañía | Sucursales de otra compañía |
    | `POST /c/{companySlug}/appointments` | Reserva `PENDIENTE` (con `Idempotency-Key`) | — |

    - **Ningún endpoint lista compañías.** Rango de fechas acotado (máx. 31 días), rate limit por IP y por compañía, respuesta 404 uniforme para mitigar enumeración (riesgo Media de Natasha, GON-38).
11. **Contingencia de sucursal (CR-6).**
    - `BranchClosure` (`TenantId`, `BranchId`, rango, motivo, creado por). Crearla marca las citas de esa sucursal en el rango como `REQUIERE_REAGENDAR` y las saca de las restricciones de 4.5; la sucursal deja de ofrecer horas en ese rango.
    - Lista de trabajo para el personal: citas afectadas con datos de contacto del cliente y estado de gestión (pendiente, contactado, reagendada, cancelada por el cliente).
    - Reagendar: propone horas del **mismo veterinario** en otras sucursales de la misma compañía con box libre; si no hay, otras horas del mismo veterinario o de otro de la misma sucursal. Siempre lo confirma una persona (`OWNER` o `ADMINISTRATIVO`); el sistema no mueve citas solo.
    - Notificar al cliente del cambio usa el canal de GON-29 cuando exista; mientras tanto, el contacto es telefónico y queda registrado.

## 5. Desacuerdos entre informes y su resolución

| ID | Desacuerdo | Resolución (por matriz, no por mayoría) |
|---|---|---|
| **D1** | RLS: Natasha lo recomienda ya (4,25) y Tony/Bruce lo difieren; Natasha reconoce que solo filtros puntúa 4,35. | M1 gana la matriz (4,35 vs 4,25). Se adoptan sus controles como bloqueantes, que era la condición de Natasha si se elegía solo filtros. **Condicional a T-07:** la penalización de M4 en simplicidad depende de HIPÓTESIS-2 (el pooler de Neon en modo transaction obliga a `SET LOCAL` por transacción), que nadie verificó. T-07 se ejecuta **antes** de aceptar el ADR; si refuta la hipótesis, M4 se re-puntúa. CR-4 no es una decisión sobre RLS (Claudio no conocía el término): la decisión se pide explícitamente en §8.1. |
| **D2** | Auth: Bruce recomienda Identity; Supabase Auth solo cabe si la BD es Supabase. | Identity + JWT (4,05). Strange descarta Supabase como BD (pausa a 7 días), lo que deja B sin base. Costo: 5-8 pts de esfuerzo, dividido en la hoja de ruta. |
| **D3** | "Costo $0" vs. recordatorios fiables: el plan Free de Render duerme a los 15 min (cold start ~1 min) y no corre jobs. | $0 vale para dev/staging; producción requiere pago mínimo o un cron externo que despierte el servicio. Pierde puntos el criterio de costo, no se oculta. Decisión de gasto: humano. |
| **D4** | Disponibilidad por sucursal (`CLAUDE.md`, Tony/Bruce) vs. por veterinario (Claudio). | Prevalece el requisito humano (CR-1/CR-2): por veterinario **y** por sucursal, con box como recurso limitante (Claudio, 2026-11-07). Restricción por veterinario y por box (4.5). |
| **D5** | Restricción anti-doble-reserva: A (índice único) vs. B (`EXCLUDE`). | B, porque la duración varía por tipo de atención y la agenda es por veterinario. Verificar `btree_gist` en Neon. |
| **D6** | Versiones frontend: v11 de PrimeReact (`latest` en npm) vs. v10 que asume `DESIGN.md`; FullCalendar 6 vs 7; TS 7/Vite 8 sin verificar. | `primereact@10.9.9` por decisión humana. TS 7/Vite 8 aceptados por Claudio, pero su compatibilidad con ESLint/Vitest/plugins es HIPÓTESIS: el scaffold fija versiones exactas y falla si no resuelven. FullCalendar: spike. |
| **D7** | Acceso de ADMINISTRATIVO a datos clínicos. Natasha: sin acceso; CR-3: ficha accesible "desde cualquier veterinario". | Compatibles. Se propone ADMINISTRATIVO sin historial, recetas ni diagnósticos; PROFESIONAL y OWNER con acceso a toda la ficha de la compañía. Cierre en GON-22. |
| **D8** | Tamaño: varios bloques superan 3 pts (Bruce §8). | Se dividen en la hoja de ruta (todas las tareas ≤ 3 pts). |

## 6. Consecuencias

**Positivas**
- Un despliegue y una BD: costo y superficie operativa mínimos.
- El aislamiento tiene un único punto de resolución y pruebas automáticas.
- Reversibilidad alta: contenedor Docker, PostgreSQL estándar y SPA estática; M1 → M4 es una migración aditiva.

**Riesgos y mitigaciones**
- Sin RLS, el aislamiento depende del código. Mitigación: controles bloqueantes de la decisión 2 y revisión de M4 antes del primer dato clínico real.
- Un módulo puede afectar a todos. Mitigación: límites por módulo, pruebas de arquitectura y health checks.
- Free tiers cambian de política (HIPÓTESIS). Mitigación: reverificar en páginas oficiales antes de contratar.
- La disponibilidad por veterinario y box añade una segunda restricción y un modelo de turnos (veterinario × sucursal × box). Mitigación: ambas restricciones en BD y prueba de concurrencia por cada una.
- La contingencia de sucursal (CR-6) puede afectar muchas citas a la vez. Mitigación: nada se mueve sin confirmación humana y la lista de trabajo es idempotente.
- La auth propia aumenta la superficie a mantener (reset, lockout, rotación de claves).

**Efectos documentales**
- `CLAUDE.md` §1 y §2.4 ya reflejan CR-1, CR-2, CR-3 y CR-6 (B-1). Al aceptar el ADR, GON-48 cambia las marcas **[Propuesto]** a **[Decidido — ADR-0001]**.
- Higiene del repo pendiente (GON-32/38): endurecer `.gitignore` y `.claude/settings.json` (tarea T-01).

## 7. Reversibilidad y rollback

Mientras este ADR esté **Propuesto** no se aplica nada. Descartar = cerrar GON-33 y marcarlo Rechazado. Si se acepta y luego se cambia: M4 se agrega o retira con una migración nueva; un módulo se extrae sin cambiar `/api/v1`; el hosting se mueve con `pg_dump`/`pg_restore` y redeploy de la misma imagen.

## 8. Decisiones abiertas para el humano

1. **RLS (D1).** *Qué es:* Row-Level Security es una regla dentro de PostgreSQL que oculta las filas de otra compañía aunque el código de la aplicación olvide filtrar; es una segunda cerradura detrás de los filtros de EF Core. *Costo:* cada transacción debe fijar la compañía (`SET LOCAL`), y un error ahí devuelve 0 filas en vez de datos ajenos. *Pregunta:* ¿se acepta diferir RLS a fase 2 (T-27) con la condición de que **ningún despliegue con datos clínicos reales** ocurra antes (T-28 depende de T-27), o se exige desde la primera migración? Recomendación de Charles: diferir con esa puerta. La decisión se toma después del informe de T-07.
2. ~~**Capacidad y boxes**~~ **Resuelta (Claudio, 2026-11-07):** el box limita las citas simultáneas; se modela `Box` con su propia restricción (4.5).
3. **Cita sin veterinario fijo:** al buscar por sucursal, ¿el sistema asigna un veterinario concreto al reservar (recomendado) o la cita queda "sin asignar" hasta el día? Cambia el esquema.
4. **Duración de citas:** ¿fija o variable por tipo de atención? Se asume variable (D5).
5. **Costo de producción (D3):** ¿se aprueba ≈ USD 12/mes (Render de pago + Neon Launch) o se acepta cron externo sobre el plan Free?
6. **FullCalendar 6 o 7:** se resolverá con el spike T-05; ¿se acepta ese criterio?
7. **Cliente en dos compañías:** dos fichas separadas (supuesto actual). ¿Producto lo confirma?
8. **ADMINISTRATIVO sin acceso clínico (D7):** ¿se acepta como política por defecto?
9. **Stripe Connect (GON-25)** y **canal de recordatorios (GON-29):** siguen abiertos; este ADR no los cierra.

## 9. Registro de agentes que participaron

| Agente | Issue | Aporte | Observación |
|---|---|---|---|
| Vision · Code Auditor | GON-32 | Estado real del repo; brechas en `.gitignore` y `settings.json` | Pregunta sobre un repo "antiguo" respondida por Claudio: no existe. |
| Tony · Software Architect | GON-34 | Estilo, módulos, tenancy, contrato API | Declaró haber leído GON-32 y no GON-35 a 38. Firma como "Code Software Arquitect". |
| Bruce · Backend | GON-35 | .NET 10, auth, concurrencia (prueba medida en Postgres 17.11 efímero) | Independiente. El script de la prueba está adjunto en Multica, no en el repo; no se re-ejecutó aquí. |
| Shuri · Frontend | GON-36 | Versiones npm, estructura, accesibilidad, pruebas | Independiente. Compatibilidad solo por peerDependencies, sin prueba en runtime. |
| Strange · Cloud Architect | GON-37 | Hosting, BD, CI, costos | Fuentes oficiales vía resumen de búsqueda; cifras por reverificar. |
| Natasha · Security | GON-38 | Amenazas, aislamiento, secretos, Stripe | Independiente. No verificó protección de rama de GitHub. |
| Domino · QA | GON-39 | Verificó la aritmética de las matrices; 5 objeciones bloqueantes y 7 no bloqueantes; criterios dado/cuando/entonces. | Veredicto `requiere cambios`. Correcciones en sección 10; re-revisión pendiente. |
| Charles · Engineering Manager | GON-33 | Preguntas, criterios, matrices, síntesis, ADR y hoja de ruta | Las puntuaciones son juicio propio; Domino debe verificar su trazabilidad. |
| Claudio (humano) | GON-34, 35, 36 | Cinco cambios de requisito (sección 2) | Prevalecen sobre `CLAUDE.md`. |

No participaron Pepper, Reed, Wanda, Sage, Forge ni Peter. Sage no investigó HIPÓTESIS-2 (pooler) ni `btree_gist` en Neon, aunque Tony lo sugirió: queda como tarea.

## 10. Correcciones tras la revisión QA (GON-39, 2026-11-07)

| Objeción | Respuesta humana | Corrección |
|---|---|---|
| **B-1** `CLAUDE.md` se actualizaba tarde (T-26) | Agendar por veterinario y por sucursal; contingencia ante cierre de sucursal | `CLAUDE.md` §1 y §2.4 actualizados en este PR. CR-1 reescrito, CR-6 nuevo, 4.11 y tareas T-30 a T-32. T-26 ya no edita requisitos. |
| **B-2** Slug público ambiguo | Entendido | 4.10: `companySlug` resuelve el tenant; sucursal y veterinario dentro de la compañía; tabla de endpoints; ningún endpoint lista compañías. |
| **B-3** Prueba de aislamiento tardía | Ok | Excepción temporal y acotada en 4.2 (solo T-08, dueño Bruce en T-10). T-10 trae la prueba cruzada con PostgreSQL local, sin esperar CI. Toda tarea de esquema posterior depende de T-10 y amplía la prueba. |
| **B-4** Dependencias mal referenciadas | Arreglarlas | Encabezado del roadmap alineado con la tabla; §8.6 apunta a T-05. |
| **B-5** RLS sin decisión informada; D1 sobre HIPÓTESIS-2 | Pregunta qué hace falta | §8.1 explica RLS y pide decisión explícita. D1 condicional a T-07, que pasa a ejecutarse antes de aceptar el ADR. T-28 depende de T-27 o de una decisión humana registrada (N-3). |

No bloqueantes atendidos: N-3 (puerta de T-27), N-5 (dos veterinarios en el mismo horario y zona horaria de sucursal en criterios de T-08/T-09), N-6 (CR-3 queda como supuesto). Pendientes para la re-revisión: N-1, N-2, N-4, N-7.
