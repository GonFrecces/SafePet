---
name: ci-cd
description: Diseña y revisa pipelines de CI/CD en GitHub Actions para backend .NET y frontend React: jobs por ruta, PostgreSQL de servicio, caché, checks requeridos y despliegue con aprobación manual. Úsala al crear o modificar workflows.
---

# CI/CD (GitHub Actions)

## CI (`.github/workflows/ci.yml`)
- Disparadores: `pull_request` y `push` a `master`.
- `dorny/paths-filter` (o `paths`) para correr solo el job afectado.
- **backend**: `actions/setup-dotnet` (versión fijada) → `dotnet restore` → `dotnet build --no-restore -warnaserror` → `dotnet format --verify-no-changes` → `dotnet test` con `services: postgres` (imagen versionada, healthcheck).
- **frontend**: `actions/setup-node` con `cache: npm` → `npm ci` → `lint` → `format:check` → `test` → `build`.
- Caché NuGet (`~/.nuget/packages`, key por `*.csproj`/`packages.lock.json`).
- `permissions: contents: read` por defecto; `concurrency` para cancelar ejecuciones obsoletas.
- Acciones fijadas por versión mayor o SHA.

## CD
- Separado del CI; `workflow_dispatch` o `environment` con **reviewers requeridos** (aprobación humana).
- Migraciones: script idempotente revisado; aplicación manual aprobada; backup/branch previo.
- Secretos solo en GitHub Environments/proveedor; nunca en el YAML.

## Checks requeridos
Documentar que la protección de rama de `master` debe exigir `backend` y `frontend` (acción manual del dueño).

## Verificación
PR de prueba con un test roto ⇒ el check falla. Tiempo objetivo < 10 min.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
