---
name: backend-engineering
description: Guía para evaluar e implementar el backend C#/ASP.NET Core en capas de SafePet: capas, casos de uso, validación, autorización, concurrencia, errores y pruebas. Úsala al analizar o escribir código del backend.
---

# Backend engineering (.NET en capas)

## Estructura
`Api → Application → Domain`; `Infrastructure → Application/Domain`. `Domain` sin dependencias. Una prueba de arquitectura (NetArchTest o reflexión) lo verifica.

## Patrones
- **Caso de uso** en `Application`: `public sealed class BookAppointmentHandler` con `HandleAsync(BookAppointmentCommand cmd, CancellationToken ct)`; retorna `Result<T>` o lanza excepciones de dominio mapeadas a ProblemDetails.
- **Controller/endpoint delgado**: valida DTO, llama al caso de uso, mapea respuesta.
- **Validación**: FluentValidation o DataAnnotations en DTO; invariantes en entidades de dominio (constructor/métodos, no setters públicos).
- **Autorización**: políticas por rol (`OwnerOnly`, `Clinical`, `FrontDesk`); tenant resuelto por `ITenantContext` desde claims.
- **Persistencia**: EF Core + Npgsql, global query filter por `TenantId`, `SaveChanges` intercepta y asigna `TenantId` al crear.
- **Concurrencia**: reserva con índice único `(BranchId, StartsAt)` + manejo de `DbUpdateException` → 409; o token de concurrencia `xmin`.
- **Tiempo**: `TimeProvider` inyectado; todo en UTC.
- **Errores**: middleware global → `application/problem+json`; sin stack traces en producción.
- **Logs**: estructurados (Serilog/ILogger), sin datos clínicos ni personales.

## Checklist de evaluación (modo análisis)
Versión .NET y soporte · capas y dependencias reales · lógica en controllers · validación · auth · multi-tenancy · concurrencia · manejo de errores · pruebas existentes. Cada punto: hecho/hipótesis + evidencia.

## Verificación (modo ejecución)
`dotnet build -warnaserror`, `dotnet format --verify-no-changes`, `dotnet test` (suite completa si tocaste Domain/Application).

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
