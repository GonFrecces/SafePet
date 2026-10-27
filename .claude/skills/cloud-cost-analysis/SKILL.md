---
name: cloud-cost-analysis
description: Compara proveedores de hosting y base de datos (free tiers vigentes, límites, costos al escalar, lock-in) con fuentes y fecha de consulta. Úsala al decidir dónde desplegar backend, frontend y PostgreSQL.
---

# Análisis de costos cloud

## Procedimiento
1. **Requisitos**: runtime .NET en contenedor, PostgreSQL, HTTPS, jobs de notificación, webhooks Stripe (requieren endpoint siempre accesible o reintentos), región cercana a usuarios.
2. **Candidatos** (verificar vigencia, cambian seguido):
   - Backend: Render, Fly.io, Azure App Service (F1), Railway, Koyeb, Google Cloud Run.
   - BD: Neon, Supabase.
   - Frontend: Netlify, Vercel, Cloudflare Pages, Render static.
3. **Por candidato, desde la página oficial de precios** (enlace + fecha): horas/mes, RAM/CPU, sleep/cold start, ancho de banda, almacenamiento, conexiones, backups/branching, región, tarjeta requerida, límites que pausan el servicio.
4. **Impacto funcional**: cold start vs recordatorios puntuales; sleep vs webhooks; límite de conexiones vs pool de EF Core.
5. **Costo al escalar**: primer plan pago y qué lo dispara.
6. **Lock-in y salida**: ¿es contenedor estándar? ¿Postgres estándar exportable con `pg_dump`?

## Salida
Tabla `proveedor | límites free | riesgos para SafePet | primer plan pago | fuente (fecha)` + recomendación + plan B.

## Reglas
Dato no verificado = HIPÓTESIS. Nunca crear cuentas ni recursos.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
