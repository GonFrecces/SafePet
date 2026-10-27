---
name: api-contracts
description: Define y revisa contratos de API REST: recursos, verbos, DTOs, errores, paginación, versionado, idempotencia y compatibilidad con el frontend. Úsala al diseñar endpoints o cambiar un contrato existente.
---

# Contratos de API

## Convenciones
- Base: `/api/v1`. Público sin login: `/api/v1/public/...`. Backoffice: `/api/v1/...` con `[Authorize]` + política de rol.
- Recursos en plural, kebab-case: `/branches/{branchId}/availability`, `/appointments/{id}/close`.
- JSON camelCase; fechas ISO-8601 UTC (`2026-10-27T15:00:00Z`); dinero `{ "amount": 15000, "currency": "CLP" }` en unidad mínima.
- Enums como string (`"CONSULTA"`).
- Errores: RFC 9457 `application/problem+json` con `type`, `title`, `status`, `detail`, `errors` (validación).
- Paginación: `?page=1&pageSize=20` → `{ items, page, pageSize, total }`.
- Idempotencia: `Idempotency-Key` en POST de reserva y cobro.
- El tenant **nunca** viaja como parámetro libre en el backoffice; en público se deriva de la sucursal y se valida.

## Procedimiento
1. Describe cada endpoint: método, ruta, rol, request DTO, response DTO, códigos (200/201/400/401/403/404/409/422).
2. Documenta en OpenAPI (Swashbuckle/NSwag) y genera tipos para `frontend/src/api/`.
3. **Cambio compatible** (agregar campo opcional, nuevo endpoint) vs **breaking** (renombrar/eliminar campo, cambiar tipo/semántica). Breaking ⇒ nueva versión o expand→contract.
4. Lista consumidores afectados (pantallas del frontend).

## Checklist de revisión
- [ ] Sin entidades de EF expuestas.
- [ ] 404 en vez de 403 para recursos de otro tenant (no revelar existencia).
- [ ] Validación en servidor de todos los campos.
- [ ] Pruebas de contrato/integración del endpoint.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
