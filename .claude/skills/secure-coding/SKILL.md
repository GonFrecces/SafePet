---
name: secure-coding
description: Aplica prácticas de código seguro en el backend .NET y el frontend React de SafePet: autenticación, autorización, multi-tenancy, validación, manejo de errores y pagos. Úsala al escribir o revisar código sensible.
---

# Código seguro

## Autenticación
- Contraseñas con hasher de ASP.NET Identity (o proveedor gestionado); nunca hash propio.
- JWT: firma con clave de ≥256 bits desde configuración segura, `exp` corto (≤15 min) + refresh rotativo, validar `iss`, `aud`, `exp`, algoritmo fijo.
- Tokens en cookie `HttpOnly; Secure; SameSite` o memoria; no `localStorage` si puede evitarse.

## Autorización y multi-tenancy
- `[Authorize(Policy=...)]` por defecto (fallback policy autenticada); endpoints públicos marcados explícitamente `[AllowAnonymous]`.
- Tenant desde claims, nunca desde body/query en backoffice. En público, derivado de la sucursal y validado.
- Prohibido `IgnoreQueryFilters()` salvo caso justificado y revisado.
- Recursos ajenos ⇒ 404.

## Entradas
- Validar todo en servidor (tipo, rango, longitud). EF Core parametriza; si hay SQL crudo, `FromSqlInterpolated` nunca concatenación.
- React escapa por defecto; prohibido `dangerouslySetInnerHTML` con datos de usuario.
- Rate limiting (`AddRateLimiter`) en endpoints públicos de reserva; CAPTCHA si hay abuso.

## Pagos (Stripe)
- Montos calculados en servidor desde catálogo; el cliente solo envía ids.
- Webhooks: verificar firma (`EventUtility.ConstructEvent`), idempotencia por `event.id`.
- Solo claves de modo test en desarrollo.

## Errores y configuración
ProblemDetails sin stack trace en producción; HTTPS + HSTS; CORS con orígenes explícitos; headers de seguridad (CSP en el front).

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
