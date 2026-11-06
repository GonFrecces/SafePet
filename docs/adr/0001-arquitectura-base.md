# ADR-0001 · Arquitectura base de SafePet

| Campo | Valor |
|---|---|
| Estado | **Propuesto** (nada se aplica hasta que un humano lo marque Aceptado) |
| Fecha | 2026-11-05 |
| Facilitador | Charles · Engineering Manager (GON-33) |
| Informes de entrada | GON-32 (Vision), GON-34 (Tony), GON-35 (Bruce), GON-36 (Shuri), GON-37 (Strange), GON-38 (Natasha) |
| Revisión QA | GON-39 (Domino): **pendiente**; se ejecuta sobre este PR |
| Hoja de ruta | [`docs/roadmap/roadmap-mvp.md`](../roadmap/roadmap-mvp.md) |

Convención: **HECHO** = con evidencia citada en un informe; **HIPÓTESIS** = sin verificar. Las puntuaciones de la matriz son juicio de Charles sobre la evidencia de los informes, no mediciones.

---

## 1. Contexto

- **HECHO (GON-32, confirmado por Claudio):** el repositorio no tiene código, dependencias, pruebas, CI ni infraestructura. Solo existe `GonFrecces/SafePet`; no hay repositorio "antiguo". La decisión parte de cero.
- **HECHO:** stack declarado en `CLAUDE.md` §2.1: React + PrimeReact + FullCalendar, C#/ASP.NET Core en capas, PostgreSQL (Neon o Supabase), Stripe.
- SafePet es multi-tenant, maneja datos clínicos, tiene una agenda pública anónima y lo construye un equipo pequeño de agentes con revisión humana, con objetivo de costo $0 en desarrollo.
- Decisiones relacionadas que **este ADR no cierra**: autenticación (GON-12, se recomienda aquí), modelo de cobro Stripe (GON-25), canal de recordatorios (GON-29), despliegue (GON-31).

## 2. Cambios de requisito (respuestas humanas que prevalecen sobre `CLAUDE.md`)

Claudio respondió en los issues de informe. Según las instrucciones de esta reunión, estas respuestas son requisitos de producto y **modifican `CLAUDE.md`**, que debe actualizarse en el PR del scaffold (no en este).

| # | Requisito nuevo (fuente) | Contradice | Efecto |
|---|---|---|---|
| **CR-1** | **La disponibilidad se gestiona por veterinario**, no solo por sucursal: una sucursal puede cubrir 1 o 2 boxes por turnos, y otra tener veterinarios con contrato completo. La agenda muestra disponibilidad por veterinario (GON-34, resp. 1). | `CLAUDE.md` §1: "disponibilidad de la clínica/sucursal (no del veterinario)". | Cambia el modelo de Agenda, la restricción anti-doble-reserva y la capacidad (sección 5, D4). |
| **CR-2** | **El cliente busca una hora por sucursal o por veterinario** y ve dónde hay horas o quién está disponible en qué sucursal de la misma compañía (GON-35, resp.). | `CLAUDE.md` §1 (agenda solo por sucursal). | Dos consultas públicas: por sucursal y por veterinario, ambas dentro de una compañía. |
| **CR-3** | **La ficha clínica es central por compañía**: cada compañía tiene N sucursales; la ficha pertenece al cliente dentro de una sola compañía y es plenamente accesible desde cualquier sucursal y veterinario de esa compañía (GON-34, resp. 3). | Precisa `CLAUDE.md` §1 ("ficha a nivel de clínica"): el tenant es la **compañía**, no la sucursal. | La ficha no se filtra por `BranchId`, solo por `TenantId`. Un cliente en dos compañías tiene dos fichas separadas (aceptado implícitamente). |
| **CR-4** | **No hay cliente real hoy** y RLS no es un requisito conocido por producto (GON-34, resp. 2). | Ninguna; es un dato para D1. | Respalda diferir RLS a fase 2, con revisión antes del primer dato clínico real. |
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
   - Tenant solo del servidor: claim del JWT o `slug` público de sucursal; el cliente nunca envía `TenantId`.
   - Query filter por convención sobre toda entidad `ITenantOwned`, con fallo cerrado (tenant nulo = 0 filas).
   - `SaveChangesInterceptor`: estampa `TenantId` en altas y rechaza modificar entidades de otro tenant.
   - FKs compuestas `(TenantId, Id)`.
   - `IgnoreQueryFilters`, `FromSql*`/`ExecuteSql` y `ExecuteUpdate/Delete` fuera de una lista permitida revisada: **falla CI**.
   - Recursos de otro tenant responden **404**.
   - Prueba de aislamiento parametrizada por cada entidad con `TenantId`, contra PostgreSQL real.
   - Agenda pública: rate limit, anti-bot y reserva `PENDIENTE` con TTL hasta confirmar.
   - Webhooks Stripe con firma verificada sobre body crudo e idempotencia por `event.id`; sin PII en logs; secretos por entorno con secret scanning.
3. **Stack backend:** .NET 10 LTS + EF Core 10 + Npgsql 10, fijado con `global.json`, `Directory.Build.props` y Central Package Management (HECHO, GON-35: .NET 8/9 vencen el 2026-11-10). Pruebas de integración con PostgreSQL real; prohibido InMemory/SQLite.
4. **Auth:** ASP.NET Identity + JWT propio (A), con JWT ≤ 15 min, refresh rotado y claims `tenant_id`, `role`, `branch_ids` emitidos por el servidor. La decisión final queda en GON-12.
5. **Anti-doble-reserva:** restricción en la BD, no en código. Con CR-1 y duración variable por tipo de atención se propone `EXCLUDE USING gist` por **veterinario** y rango (`btree_gist`), mapeando `23P01`/`23505` a **409** `slot-unavailable`. Medido por Bruce: la verificación-luego-inserción en READ COMMITTED duplicó reservas (3 filas en 2 de 5 corridas). Disponibilidad de `btree_gist` en Neon: HIPÓTESIS a verificar.
6. **Contrato API:** `/api/v1`, OpenAPI, ProblemDetails (RFC 9457), `Idempotency-Key` en reservas y pagos, fechas UTC, superficies separadas (`public`, backoffice, `webhooks/stripe`).
7. **Frontend:** una SPA Vite + TypeScript con rutas públicas (`/c/:clinicSlug/...`) y de backoffice (`/app/*`) con `lazy()`; React Router 7, TanStack Query, react-hook-form + zod; **`primereact@10.9.9`** con tema Lara y overrides de `DESIGN.md`; Vitest + Testing Library + MSW + `vitest-axe`. Versiones exactas con lockfile. FullCalendar 6.1.22 vs 7.1.3 se decide con un spike de accesibilidad y estilos (CR-5).
8. **Hosting:** Render (Docker) + Neon + Cloudflare Pages. Free solo para dev/staging; **producción en instancia de pago mínima** (≈ USD 12/mes, HIPÓTESIS) porque el plan Free duerme el servicio y los recordatorios no corren (ver D3). GitHub Actions con jobs por ruta y PostgreSQL de servicio. Vercel Hobby, Fly.io y Supabase como BD quedan descartados.
9. **Pagos:** se registra la recomendación de Natasha (Stripe Connect por clínica) como **insumo**, sin decidir; el cierre es GON-25.

## 5. Desacuerdos entre informes y su resolución

| ID | Desacuerdo | Resolución (por matriz, no por mayoría) |
|---|---|---|
| **D1** | RLS: Natasha lo recomienda ya (4,25) y Tony/Bruce lo difieren; Natasha reconoce que solo filtros puntúa 4,35. | M1 gana la matriz (4,35 vs 4,25). Se adoptan sus controles como bloqueantes, que era la condición de Natasha si se elegía solo filtros. RLS se difiere a fase 2 porque no hay cliente real (CR-4), el pooler en modo transaction exige `SET LOCAL` (Tony HIP-2) y la simplicidad pesa 20 %. **Margen estrecho:** decisión abierta al humano (sección 8). |
| **D2** | Auth: Bruce recomienda Identity; Supabase Auth solo cabe si la BD es Supabase. | Identity + JWT (4,05). Strange descarta Supabase como BD (pausa a 7 días), lo que deja B sin base. Costo: 5-8 pts de esfuerzo, dividido en la hoja de ruta. |
| **D3** | "Costo $0" vs. recordatorios fiables: el plan Free de Render duerme a los 15 min (cold start ~1 min) y no corre jobs. | $0 vale para dev/staging; producción requiere pago mínimo o un cron externo que despierte el servicio. Pierde puntos el criterio de costo, no se oculta. Decisión de gasto: humano. |
| **D4** | Disponibilidad por sucursal (`CLAUDE.md`, Tony/Bruce) vs. por veterinario (Claudio). | Prevalece el requisito humano (CR-1/CR-2). Se pasa de `EXCLUDE` por sucursal a por veterinario. Queda abierto cómo se limita la capacidad de **boxes** (D-abierta 2). |
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
- La disponibilidad por veterinario con boxes compartidos puede requerir un modelo de recursos más complejo (D-abierta 2).
- La auth propia aumenta la superficie a mantener (reset, lockout, rotación de claves).

**Efectos documentales**
- `CLAUDE.md` §1 y §2.4 deben actualizarse por CR-1 a CR-3 en el PR del scaffold.
- Higiene del repo pendiente (GON-32/38): endurecer `.gitignore` y `.claude/settings.json` (tarea T-01).

## 7. Reversibilidad y rollback

Mientras este ADR esté **Propuesto** no se aplica nada. Descartar = cerrar GON-33 y marcarlo Rechazado. Si se acepta y luego se cambia: M4 se agrega o retira con una migración nueva; un módulo se extrae sin cambiar `/api/v1`; el hosting se mueve con `pg_dump`/`pg_restore` y redeploy de la misma imagen.

## 8. Decisiones abiertas para el humano

1. **RLS (D1):** ¿se acepta diferir RLS a fase 2 (recomendado) o se exige antes del primer cliente? Recomendación de Charles: diferir, con la revisión como tarea explícita (T-20).
2. **Capacidad y boxes (CR-1, D4):** ¿la sucursal tiene un número de boxes que limita las citas simultáneas aunque haya más veterinarios libres? Si sí, se modela `Box`/recurso con su propia restricción.
3. **Cita sin veterinario fijo:** al buscar por sucursal, ¿el sistema asigna un veterinario concreto al reservar (recomendado) o la cita queda "sin asignar" hasta el día? Cambia el esquema.
4. **Duración de citas:** ¿fija o variable por tipo de atención? Se asume variable (D5).
5. **Costo de producción (D3):** ¿se aprueba ≈ USD 12/mes (Render de pago + Neon Launch) o se acepta cron externo sobre el plan Free?
6. **FullCalendar 6 o 7:** se resolverá con el spike T-04; ¿se acepta ese criterio?
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
| Domino · QA | GON-39 | **No participó todavía.** Revisa este PR después de publicarlo. | Sus objeciones bloqueantes se resuelven o se registran antes de la aceptación. |
| Charles · Engineering Manager | GON-33 | Preguntas, criterios, matrices, síntesis, ADR y hoja de ruta | Las puntuaciones son juicio propio; Domino debe verificar su trazabilidad. |
| Claudio (humano) | GON-34, 35, 36 | Cinco cambios de requisito (sección 2) | Prevalecen sobre `CLAUDE.md`. |

No participaron Pepper, Reed, Wanda, Sage, Forge ni Peter. Sage no investigó HIPÓTESIS-2 (pooler) ni `btree_gist` en Neon, aunque Tony lo sugirió: queda como tarea.
