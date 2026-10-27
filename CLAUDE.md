# CLAUDE.md — Reglas del repositorio SafePet

Este archivo define cómo trabajar en este repositorio (personas y agentes de IA, incluido Claude Code).
Es de cumplimiento obligatorio. Si una regla entra en conflicto con una instrucción puntual, se pregunta antes de actuar.

> **Estado del repositorio (hecho observado, 2026-10-26):** el repositorio solo contiene `README.md`.
> No existe aún código de frontend, backend, base de datos, CI ni pruebas.
> Todo lo que se marca como **[Propuesto]** es la estructura objetivo definida a partir del alcance del proyecto
> en Linear ("SafePet - Update 🐈", equipo GON) y debe actualizarse en este archivo cuando se implemente.
> Lo marcado como **[Hecho]** está verificado en el repositorio o en Linear.

---

## 1. Producto

SafePet es una aplicación web responsiva (escritorio y móvil) para agendar horas en clínicas veterinarias y gestionar la atención clínica.

**[Hecho — Linear]** Requerimientos funcionales:

- **Agenda pública sin login:** el cliente agenda una hora disponible según la disponibilidad de la **clínica/sucursal** (no del veterinario). Cualquier veterinario de la clínica puede atender.
- **Notificaciones:** se avisa al usuario minutos antes del inicio de la cita.
- **Multi-tenant:** muchas compañías usan la plataforma; cada dueño ve solo sus sucursales y clientes.
- **Ficha clínica a nivel de clínica:** el historial médico de la mascota pertenece al tenant (clínica), no es global.
- **Tipos de atención:** `CONSULTA`, `TRATAMIENTO`, `CORTE_DE_PELO`, `URGENCIA`, `PROCEDIMIENTO`.
- **Ciclo de cita:** cada veterinario abre y cierra sus citas agendadas.
- **Gestión:** atenciones, veterinarios, historial médico, recetas, tratamientos, pagos de atenciones y ventas con **Stripe**.

**[Hecho — Linear]** Perfiles:

| Rol               | Alcance                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------- |
| `OWNER`           | Único por compañía. Acceso a todas las funcionalidades, limitado a sus sucursales.          |
| `PROFESIONAL`     | Veterinario: atención, seguimiento, tratamientos, recetas, historial médico, corte de pelo. |
| `ADMINISTRATIVO`  | Agenda, disponibilidad, venta de productos, cobro de atenciones, turnos.                    |
| Cliente (anónimo) | Solo agenda horas. Sin cuenta.                                                              |

---

## 2. Arquitectura y responsabilidades

### 2.1 Stack **[Hecho — definido en el proyecto]**

- **Frontend:** React + PrimeReact + React Bits + FullCalendar (React).
- **Backend:** C# / ASP.NET Core, proyecto en capas.
- **Base de datos:** PostgreSQL alojado en Neon o Supabase.
- **Pagos:** Stripe.
- **Hosting backend:** proveedor gratuito (por definir; candidatos: Render, Fly.io).

### 2.2 Estructura de carpetas **[Propuesto]**

```
/
├── CLAUDE.md
├── README.md
├── frontend/                     # SPA React (Vite)
│   ├── src/
│   │   ├── app/                  # router, providers, layout
│   │   ├── features/<dominio>/   # agenda, pacientes, recetas, pagos... (componentes + hooks + api)
│   │   ├── shared/               # componentes UI reutilizables, utilidades
│   │   └── api/                  # cliente HTTP y tipos de contrato
│   └── tests/
├── backend/
│   ├── SafePet.sln
│   ├── src/
│   │   ├── SafePet.Api/            # Presentación: controllers/endpoints, auth, middlewares, DI
│   │   ├── SafePet.Application/    # Casos de uso, DTOs, validaciones, interfaces (puertos)
│   │   ├── SafePet.Domain/         # Entidades, value objects, reglas de negocio, enums
│   │   └── SafePet.Infrastructure/ # EF Core + Npgsql, migraciones, Stripe, notificaciones, email/SMS
│   └── tests/
│       ├── SafePet.Domain.Tests/
│       ├── SafePet.Application.Tests/
│       └── SafePet.Api.IntegrationTests/
├── docs/                         # equipo de agentes, ADRs (docs/adr) y hoja de ruta
├── .claude/agents/               # subagentes de Claude Code
└── .claude/skills/               # skills de los subagentes (SKILL.md)
```

### 2.3 Responsabilidades por capa

| Componente               | Responsabilidad                                                                                    | No debe                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `frontend`               | UI responsiva, agenda pública, backoffice por rol, consumo de la API.                              | Contener reglas de negocio críticas, secretos o claves privadas de Stripe. |
| `SafePet.Api`            | HTTP, autenticación/autorización por rol, resolución del tenant, serialización, manejo de errores. | Acceder a `DbContext` directamente ni contener lógica de negocio.          |
| `SafePet.Application`    | Orquestar casos de uso (agendar, abrir/cerrar cita, emitir receta, cobrar).                        | Depender de EF Core, Stripe SDK o ASP.NET.                                 |
| `SafePet.Domain`         | Entidades y reglas puras (estados de cita, tipos de atención, invariantes).                        | Tener dependencias externas.                                               |
| `SafePet.Infrastructure` | Persistencia, migraciones, integración Stripe (incl. webhooks), jobs de notificación.              | Exponer entidades de EF hacia la API.                                      |

Regla de dependencias: `Api → Application → Domain` e `Infrastructure → Application/Domain`. `Domain` no referencia a nadie.

### 2.4 Multi-tenancy **[Propuesto — crítico]**

- Toda entidad perteneciente a una clínica lleva `TenantId` (compañía) y, cuando aplique, `BranchId` (sucursal).
- El aislamiento se aplica con **global query filters** de EF Core sobre `TenantId`; nunca se confía en filtros manuales en cada consulta.
- El tenant se resuelve en `SafePet.Api` a partir del usuario autenticado. En la agenda pública se resuelve desde la clínica/sucursal elegida, nunca desde un parámetro libre que permita leer datos de otro tenant.
- Todo cambio que toque consultas, autorización o el modelo de datos debe incluir una prueba que verifique que un tenant **no** puede leer ni modificar datos de otro.

---

## 3. Comandos

> **[Propuesto]** Los comandos asumen la estructura de la sección 2.2. Cuando se cree el scaffold, verificar y corregir esta sección en el mismo PR.

### Requisitos

- Node.js LTS + npm
- .NET SDK (LTS vigente)
- PostgreSQL local (o Docker) para desarrollo y pruebas de integración
- `dotnet-ef` como herramienta local: `dotnet tool restore`

### Frontend (`frontend/`)

```bash
npm ci                # instalación reproducible
npm run dev           # servidor de desarrollo
npm run build         # build de producción
npm run lint          # ESLint
npm run format:check  # Prettier (sin escribir)
npm test              # pruebas unitarias (Vitest + Testing Library)
npx tsc --noEmit      # chequeo de tipos (si se usa TypeScript)
```

### Backend (`backend/`)

```bash
dotnet restore
dotnet build --no-restore -warnaserror
dotnet run --project src/SafePet.Api
dotnet format --verify-no-changes          # lint / estilo
dotnet test                                # todas las pruebas
dotnet test tests/SafePet.Domain.Tests     # un proyecto puntual
dotnet test --filter "FullyQualifiedName~Agenda"   # pruebas afectadas por filtro
```

### Migraciones (`backend/`)

```bash
dotnet ef migrations add <NombreDescriptivo> -p src/SafePet.Infrastructure -s src/SafePet.Api
dotnet ef migrations script <Desde> <Hasta> --idempotent -p src/SafePet.Infrastructure -s src/SafePet.Api
dotnet ef database update -p src/SafePet.Infrastructure -s src/SafePet.Api   # SOLO contra BD local
```

---

## 4. Convenciones de código

### Generales

- Idioma: código (tipos, métodos, variables) en **inglés**; textos de UI, documentación y mensajes de commit en **español**.
- Cambios pequeños, incrementales y reversibles. Un PR = un objetivo.
- No actualizar dependencias de forma masiva. Cada actualización mayor va en su propio PR con análisis de compatibilidad.
- Ramas: `feature/<descripcion>`, `fix/<descripcion>`, `chore/<descripcion>`, o la rama asignada por la sesión de Claude Code.
- Commits: [Conventional Commits](https://www.conventionalcommits.org/) — `feat(agenda): ...`, `fix(pagos): ...`, `chore: ...`, `docs: ...`.

### C# / backend

- Convenciones estándar de .NET: `PascalCase` para tipos y miembros públicos, `camelCase` para locales y parámetros, `_camelCase` para campos privados, interfaces con prefijo `I`.
- `Nullable` habilitado y `TreatWarningsAsErrors` en todos los proyectos.
- `async`/`await` de punta a punta con `CancellationToken`; sufijo `Async` en métodos asíncronos.
- Controllers delgados: validan entrada y delegan en un caso de uso de `Application`.
- DTOs de entrada/salida separados de las entidades de dominio.
- Fechas en **UTC** (`DateTimeOffset`/`timestamptz`); la conversión a zona horaria de la sucursal se hace en el borde.
- Montos de dinero como `decimal` en el dominio; en Stripe, en la unidad mínima de la moneda.
- Enums de dominio para tipos de atención, estados de cita y roles; nunca strings sueltos.

### React / frontend

- Componentes funcionales y hooks. Nombres de componentes en `PascalCase`, hooks con prefijo `use`.
- Organización por feature (`features/agenda`, `features/recetas`, ...).
- Usar componentes de PrimeReact antes de crear componentes propios; React Bits solo para elementos visuales/animación.
- Calendario de agenda con `@fullcalendar/react`.
- Diseño responsivo obligatorio (mobile first); toda pantalla se valida en ancho móvil.
- Las llamadas HTTP pasan por `src/api/`; los componentes no usan `fetch` directamente.
- Solo variables públicas en el frontend (`VITE_*`). Ninguna clave secreta puede llegar al bundle.

---

## 5. Reglas para migraciones de base de datos

1. Toda modificación de esquema se hace con **migraciones de EF Core** versionadas en `SafePet.Infrastructure/Persistence/Migrations`. Prohibido modificar el esquema a mano en Neon/Supabase.
2. **Nunca editar una migración ya fusionada** en la rama principal. Para corregir, crear una nueva migración.
3. Nombres descriptivos: `AddPrescriptionTable`, `AddTenantIdToAppointments`.
4. Cada migración debe tener `Down` funcional y probado localmente (`update` → `update <anterior>` → `update`).
5. **Cambios destructivos** (borrar/renombrar columnas o tablas, cambiar tipos, `NOT NULL` sobre datos existentes) se hacen en fases (_expand → migrate → contract_) y en PRs separados, para mantener compatibilidad entre versiones del backend y del frontend.
6. Toda tabla de datos de clínica incluye `TenantId` con índice y está cubierta por el query filter de tenant.
7. Migraciones con datos (seed/backfill) deben ser idempotentes.
8. El PR que agrega una migración debe incluir el script SQL generado (`migrations script --idempotent`) en la descripción o como artefacto para revisión.
9. Claude Code **solo** puede ejecutar `dotnet ef database update` contra una base de datos local o efímera de pruebas. Aplicar migraciones en staging/producción requiere autorización humana explícita.
10. Antes de aplicar en producción: respaldo/branch de la base (Neon branch o backup de Supabase) y plan de rollback documentado en el PR.

---

## 6. Secretos — prohibido introducirlos

- **Prohibido** commitear secretos: cadenas de conexión con contraseña, claves de Stripe (`sk_*`, `rk_*`, `whsec_*`), JWT signing keys, tokens de API, credenciales de Neon/Supabase, archivos `.env`, `appsettings.*.json` con valores reales, certificados o claves privadas.
- Configuración sensible solo vía variables de entorno, `dotnet user-secrets` en local y el gestor de secretos del proveedor de hosting.
- En el repositorio solo se permiten plantillas con valores ficticios (`.env.example`, `appsettings.Example.json`).
- Las claves publicables (`pk_*` de Stripe) pueden ir en el frontend, pero igual por variable de entorno.
- No pegar secretos en issues, PRs, logs, comentarios ni prompts. No pedirlos ni reproducirlos.
- Si se detecta un secreto commiteado: detener el trabajo, avisar al responsable y **rotar** el secreto. Reescribir historial no basta.

---

## 7. Pruebas — requisito de ejecutar las afectadas

- Todo cambio de código debe ejecutar **las pruebas afectadas** antes de commitear y reportar el resultado (comando y salida resumida) en el PR.
  - Cambio en `frontend/` → `npm run lint` + `npm test` (y `npm run build` si toca configuración o rutas).
  - Cambio en `backend/` → `dotnet build -warnaserror` + `dotnet test` de los proyectos afectados; si cambia `Domain` o `Application`, correr toda la suite del backend.
  - Cambio en migraciones o `DbContext` → pruebas de integración contra PostgreSQL local + verificación de `Down`.
  - Cambio en contrato de API → pruebas del backend **y** del frontend que lo consumen.
- Código nuevo trae pruebas: reglas de dominio con pruebas unitarias; endpoints con pruebas de integración; aislamiento multi-tenant con prueba explícita.
- Prohibido desactivar, omitir (`skip`) o borrar pruebas para que un cambio pase.
- No declarar una tarea o migración terminada sin pruebas verificables. Si no se pudieron ejecutar, decirlo explícitamente.

---

## 8. Merge y despliegue — prohibido sin autorización

- **Prohibido hacer merge** a `master`/`main` sin aprobación humana explícita en el PR.
- **Prohibido desplegar** (frontend, backend o base de datos) a cualquier entorno compartido sin autorización humana explícita.
- Prohibido `git push --force` sobre ramas compartidas y cualquier push directo a `master`/`main`.
- Prohibido aplicar migraciones, ejecutar SQL de escritura o cambiar configuración en las bases de Neon/Supabase de staging/producción.
- Prohibido crear, modificar o borrar recursos en Stripe en modo live. Solo modo test.
- Todo trabajo de Claude Code se entrega en una rama con Pull Request sujeto a revisión humana.
- **[Hecho]** `.claude/settings.json` bloquea para Claude Code: push forzado, push directo a `master`/`main`, merge de PRs y lectura de archivos de secretos (`.env`, `appsettings.Production.json`). `.gitignore` excluye esos archivos. Estas barreras complementan, no reemplazan, la protección de rama en GitHub.

---

## 9. Flujo de trabajo esperado para agentes

1. Leer este archivo y los archivos relevantes antes de proponer cambios.
2. Distinguir **hechos observados** de **hipótesis** en análisis y descripciones de PR.
3. Identificar riesgos (multi-tenancy, compatibilidad API/BD, pagos) antes de modificar código.
4. Implementar en la rama asignada, en pasos pequeños.
5. Ejecutar las pruebas afectadas (sección 7).
6. Abrir PR con: alcance, exclusiones, criterios de aceptación, pruebas ejecutadas y plan de rollback.
7. Esperar revisión humana. No hacer merge ni desplegar.

Subagentes disponibles en `.claude/agents/` (detalle de capacidades, límites, modelos y flujo en [`docs/equipo-agentes.md`](docs/equipo-agentes.md)):

| Agente                    | Rol                                                     | Escribe código                         |
| ------------------------- | ------------------------------------------------------- | -------------------------------------- |
| `engineering-lead`        | Alpha — facilita decisiones, redacta ADR y hoja de ruta | No (solo `docs/adr/`, `docs/roadmap/`) |
| `repo-auditor`            | Evidencia del repositorio actual                        | No                                     |
| `solution-architect`      | Arquitectura y alternativas                             | No                                     |
| `backend-engineer`        | Backend .NET en capas                                   | Sí, con issue aprobado                 |
| `frontend-engineer`       | SPA React                                               | Sí, con issue aprobado                 |
| `devops-cloud`            | CI/CD, hosting, costos, observabilidad                  | Solo CI/contenedores, nunca despliega  |
| `security-engineer`       | Seguridad y multi-tenancy                               | No                                     |
| `qa-reviewer`             | Revisión independiente y pruebas afectadas              | No                                     |
| `modernization-developer` | Implementa decisiones aprobadas                         | Sí, con issue + ADR aceptado           |
| `db-migrations`           | Apoyo: esquema y migraciones EF Core                    | Sí, con issue aprobado                 |

Las decisiones de arquitectura siguen el flujo: issue de decisión → informes independientes (Auditor, Architect, Backend, Frontend, DevOps, Security) → QA Review → Alpha publica ADR **Propuesto** + hoja de ruta + tareas pendientes de aprobación humana.
