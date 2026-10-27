---
name: observability
description: Define logs estructurados, health checks, métricas, trazas y alertas para el backend y frontend dentro de free tiers, sin exponer datos sensibles. Úsala al preparar el sistema para producción o diagnosticar.
---

# Observabilidad

## Mínimo viable
- **Health**: `/health/live` (proceso) y `/health/ready` (BD) con `AddHealthChecks().AddNpgSql(...)`.
- **Logs estructurados**: `ILogger` + Serilog/console JSON; campos `traceId`, `tenantId` (id, no nombre), `userId` (id), `route`, `status`, `elapsedMs`.
- **Correlación**: propagar `traceparent`; incluir `traceId` en ProblemDetails.
- **Métricas**: OpenTelemetry (`AddAspNetCoreInstrumentation`, `AddHttpClientInstrumentation`, runtime). Exportador OTLP a un backend con free tier (p. ej. Grafana Cloud) o solo logs al inicio.
- **Frontend**: captura de errores JS (p. ej. Sentry free) sin PII.

## Señales clave SafePet
Reservas creadas/fallidas (409), latencia p95 de `/public/availability`, webhooks Stripe fallidos, recordatorios enviados vs programados, errores 5xx.

## Alertas
5xx > umbral en 5 min; webhook fallido; job de recordatorios sin ejecutar; health ready caído.

## Reglas de privacidad
Nunca registrar: datos clínicos, nombres/teléfonos/correos de tutores, tokens, cuerpos de request de pago. Usar ids.

## Verificación
Probar en local que un error genera log con `traceId` y que `/health/ready` falla con BD detenida.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
