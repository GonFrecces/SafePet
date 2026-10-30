---
name: database-design
description: Diseña y revisa el modelo de datos PostgreSQL/EF Core: entidades, claves, índices, multi-tenancy, integridad y migraciones seguras. Úsala al modelar tablas o revisar cambios de esquema.
---

# Diseño de base de datos

## Reglas SafePet
- Toda tabla de datos de clínica: `tenant_id uuid not null` + índice; `branch_id` cuando aplique; cubierta por global query filter.
- PK `uuid` (v7 si está disponible) o `bigint identity`; consistente en todo el modelo.
- Fechas `timestamptz` (UTC); dinero `numeric(12,2)` en dominio.
- Enums de dominio mapeados a `text` con check constraint o enum de PostgreSQL (decidir uno y documentarlo).
- FKs explícitas con `ON DELETE` pensado (normalmente `RESTRICT` para datos clínicos).
- Índices: por `tenant_id` + columnas de filtro frecuentes; único `(branch_id, starts_at)` para evitar doble reserva.
- Borrado lógico (`deleted_at`) para fichas clínicas; nunca borrado físico de historial médico.
- Auditoría: `created_at`, `created_by`, `updated_at`.

## Procedimiento
1. Modelo conceptual (entidades y relaciones) → lógico (tablas, columnas, tipos) → físico (índices, constraints).
2. Consultas principales y su plan esperado (`EXPLAIN` en local).
3. Cambio de esquema → delegar a `reed-principal-engineer`: migración EF Core con `Down`, expand→migrate→contract si es destructivo, script idempotente.
4. Alternativas multi-tenant si se evalúan: columna + filtro (default), schema por tenant, RLS en PostgreSQL (defensa en profundidad). Comparar complejidad y riesgo.

## Checklist
- [ ] `tenant_id` + índice + filtro. - [ ] Constraints de integridad. - [ ] Índices justificados por consultas. - [ ] Migración reversible. - [ ] Prueba de aislamiento.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
