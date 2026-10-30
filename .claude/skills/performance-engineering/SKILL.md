---
name: performance-engineering
description: Mide y mejora el rendimiento de SafePet (BD, API, frontend) con evidencia antes/después y solo en entornos locales.
---

# performance-engineering

## Cuándo usarla
Al cerrar la capa 3 de un MVP con flujos críticos (agenda pública, calendario del backoffice, cobro) o ante una regresión de rendimiento.

## Procedimiento
1. Define el presupuesto: p95 de endpoints públicos, consultas críticas, tamaño de bundle, LCP/INP en móvil.
2. Mide la línea base: `EXPLAIN (ANALYZE, BUFFERS)` en BD local con datos sintéticos multi-tenant; carga ligera local (k6 o similar); Lighthouse en perfil móvil.
3. Identifica el cuello de botella con evidencia (plan de ejecución, perfil, traza).
4. Propón el cambio mínimo; si toca esquema o índices, pide issue a Reed.
5. Mide de nuevo y reporta antes/después en una tabla.

## Límites
Nunca pruebas de carga contra entornos compartidos. Sin optimizaciones sin medición. Sin datos reales.
