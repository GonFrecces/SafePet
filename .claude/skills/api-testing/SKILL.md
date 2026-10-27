---
name: api-testing
description: Diseña y escribe pruebas de integración de la API ASP.NET Core con WebApplicationFactory y PostgreSQL real (Testcontainers), incluidas autorización por rol y aislamiento multi-tenant. Úsala al crear o modificar endpoints.
---

# Pruebas de API

## Stack
xUnit + `Microsoft.AspNetCore.Mvc.Testing` (`WebApplicationFactory<Program>`) + Testcontainers.PostgreSql (o PostgreSQL de servicio en CI) + Respawn para limpiar datos entre pruebas.

## Casos mínimos por endpoint
1. **Feliz**: 200/201 y forma del response (contrato).
2. **Validación**: 400/422 con `errors`.
3. **AuthN**: 401 sin token (excepto rutas públicas).
4. **AuthZ por rol**: 403 para rol no permitido.
5. **Aislamiento multi-tenant**: usuario del tenant A pide recurso del tenant B → 404; listado de A no contiene datos de B; escritura sobre recurso de B falla y no modifica nada.
6. **Concurrencia/idempotencia** donde aplique: dos reservas al mismo horario → una 201 y otra 409; repetir POST con misma `Idempotency-Key` → mismo resultado.
7. **Webhooks**: firma inválida → 400; evento duplicado → procesado una vez.

## Convenciones
- Nombre: `Metodo_Escenario_ResultadoEsperado`.
- Datos con builders (`TenantBuilder`, `AppointmentBuilder`), nunca datos reales.
- Autenticación de prueba con `TestAuthHandler` que emite claims de rol y tenant.
- Sin dependencias de servicios externos reales (Stripe con stub/mock o modo test sin red).

## Ejecución
`dotnet test tests/SafePet.Api.IntegrationTests` ; filtrar: `--filter "FullyQualifiedName~Appointments"`.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
