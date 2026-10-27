---
name: owasp-review
description: Revisa código, diseño o un diff contra OWASP Top 10 y OWASP API Security Top 10, con foco en fugas entre tenants y datos clínicos. Úsala para revisiones de seguridad de PRs o de arquitectura.
---

# Revisión OWASP

## Mapa de revisión (Web Top 10 + API Top 10)
| Riesgo | Qué buscar en SafePet |
|---|---|
| Broken Access Control / BOLA (API1) | Acceso a `appointments/{id}`, `pets/{id}` de otro tenant; falta de política de rol |
| Broken Authentication (API2) | JWT mal validado, sin expiración, sin rate limit en login |
| Broken Object Property Level Auth (API3) | Mass assignment: DTO de entrada permite `TenantId`, `Role`, `Amount` |
| Unrestricted Resource Consumption (API4) | Agenda pública sin rate limit ni paginación |
| Broken Function Level Auth (API5) | Endpoints de OWNER accesibles por ADMINISTRATIVO |
| Sensitive Business Flows (API6) | Reservas masivas automatizadas que bloquean la agenda |
| Injection | SQL crudo concatenado, logs con entrada sin sanitizar |
| Cryptographic Failures | Datos clínicos sin TLS, secretos en repo |
| Security Misconfiguration | CORS `*`, Swagger expuesto en prod, errores detallados |
| Vulnerable Components | Dependencias con CVE (ver `dependency-audit`) |
| SSRF / Unsafe API consumption (API10) | Webhooks sin verificación de firma, URLs controladas por usuario |
| Logging Failures | Sin logs de seguridad o con PII |

## Procedimiento
1. `git diff origin/master...HEAD` (o el alcance indicado).
2. Recorre la tabla; para cada hallazgo traza un camino real desde una entrada.
3. Clasifica: Alta (explotable, fuga de datos entre tenants o pagos), Media, Baja.

## Formato de hallazgo
`[Severidad] archivo:línea — riesgo OWASP — escenario de explotación — corrección — HECHO/HIPÓTESIS`.
Sin hallazgos ⇒ dilo explícitamente.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
